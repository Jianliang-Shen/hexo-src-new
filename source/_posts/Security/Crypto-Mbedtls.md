---
title: 密码学介绍和Mbedtls相关实现
date: 2024-01-20 13:49:37
tags:
    - 密码
    - TEE
    - 算法
categories: 
    - 安全
---

包含密码学和Mbedtls的实现，以及在K210上的移植事例。

<!-- more -->

## 目录

- [目录](#目录)
- [1 密码](#1-密码)
- [2 历史上的密码](#2-历史上的密码)
- [3 对称密码](#3-对称密码)
- [4 分组密码模式](#4-分组密码模式)
  - [对称加密算法](#对称加密算法)
  - [分组密码模式](#分组密码模式)
    - [ECB Electric Codebook](#ecb-electric-codebook)
    - [CBC Cipher Block Chaining](#cbc-cipher-block-chaining)
    - [CTR](#ctr)
  - [AES](#aes)
    - [AES计算过程](#aes计算过程)
    - [AES加密工具](#aes加密工具)
      - [aescrypt2](#aescrypt2)
      - [crypt\_and\_hash](#crypt_and_hash)
  - [接口与代码示例](#接口与代码示例)
    - [算法](#算法)
    - [宏定义](#宏定义)
    - [示例代码](#示例代码)
      - [main.c](#mainc)
      - [mbedtls\_config.h](#mbedtls_configh)
      - [CMakeLists.txt](#cmakeliststxt)
      - [cipher相关结构体](#cipher相关结构体)
- [5 公钥密码](#5-公钥密码)
- [6 混合密码系统](#6-混合密码系统)
- [7 单向散列函数](#7-单向散列函数)
  - [原理](#原理)
  - [性质](#性质)
  - [应用](#应用)
  - [MD算法](#md算法)
  - [SHA算法](#sha算法)
    - [SHA256](#sha256)
  - [generic\_sum](#generic_sum)
  - [头文件和函数接口](#头文件和函数接口)
  - [依赖](#依赖)
  - [散列示例代码](#散列示例代码)
- [8 消息认证码](#8-消息认证码)
  - [消息认证码](#消息认证码)
  - [实现形式](#实现形式)
  - [HMAC示例](#hmac示例)
    - [依赖的宏](#依赖的宏)
    - [函数接口](#函数接口)
    - [代码示例](#代码示例)
- [9 数字签名](#9-数字签名)
- [10 证书](#10-证书)
- [11 密钥](#11-密钥)
- [12 随机数](#12-随机数)
  - [随机数应用](#随机数应用)
  - [构成](#构成)
  - [随机数算法](#随机数算法)
  - [工具](#工具)
  - [CTR\_DRBG](#ctr_drbg)
- [PGP](#pgp)
- [SSL/TLS](#ssltls)
- [补充](#补充)
  - [大数运算](#大数运算)
    - [库文件](#库文件)
    - [依赖宏](#依赖宏)
    - [接口](#接口)
    - [大数运算示例代码](#大数运算示例代码)
    - [算法分析](#算法分析)
      - [数位统计](#数位统计)
      - [读取字符串](#读取字符串)
      - [输出字符串](#输出字符串)
      - [数值比较](#数值比较)
      - [加减计算](#加减计算)
      - [乘法运算](#乘法运算)
      - [大数除法](#大数除法)
      - [取模运算](#取模运算)
      - [指数运算](#指数运算)
      - [求取最大公约数](#求取最大公约数)
      - [模逆运算](#模逆运算)
  - [平台安全架构](#平台安全架构)
    - [Key management](#key-management)
    - [Message digests](#message-digests)
    - [MAC](#mac)
    - [Asymmetric cryptography](#asymmetric-cryptography)
      - [非对称加解密](#非对称加解密)
      - [签名和验证](#签名和验证)
    - [Symmetric cryptography](#symmetric-cryptography)
    - [AEAD](#aead)
    - [psa示例代码](#psa示例代码)
      - [签名认证](#签名认证)
        - [导入ECC非对称密钥](#导入ecc非对称密钥)
        - [计算签名](#计算签名)
        - [验证签名](#验证签名)
      - [AEAD加密解密](#aead加密解密)
        - [导入对称密钥](#导入对称密钥)
        - [消息加密](#消息加密)
        - [消息解密](#消息解密)
  - [移植K210 RISC-V](#移植k210-risc-v)
    - [移植过程简述](#移植过程简述)
    - [测试例程](#测试例程)

## 1 密码

- 加密encrypt、解密decrypt、明文plaintext、密文ciphertext、密码破译cryptanalysis、密钥key
- 对称密码symmetric cryptography
- 公钥密码public-key cryptography
- 混合密码系统hybrid cryptography

## 2 历史上的密码

- 凯撒密码--平移密码
- 简单替换密码--频率分析破译
- Enigma

## 3 对称密码

- 编码操作：转为二进制序列
- 异或XOR：
  - 加密：明文A 异或 密钥B = 密文C
  - 解密：密文C 异或 密钥B = 明文A
- 一次性密码本：绝对无法破译但效率低的密码，无法解决密钥配送问题
- DES：Data Encryption Standard
  - 分组密码
  - Feistel网络，轮
  - 选择明文攻击CPA
    - 差分分析：改变一部分明文并分析密文如何随之改变
    - 线性分析：将明文和密文的一些对应比特位进行XOR并计算其结果为零的概率
  - 三重DES，包括DES-EDE2和DES-EDE3
- AES密码：advanced encryption standard
  - Rijndael
  - 竞争实现标准化

## 4 分组密码模式

| 模式 | 名称             | 优点                                                         | 缺点                                                         | 备注                                 |
| ---- | ---------------- | :------------------------------------------------------------ | :------------------------------------------------------------ | :------------------------------------ |
| ECB  | 电子密码本模式   | 简单<br />快速<br />支持并行运算                             | 明文中的重复排列会反映在密文中<br />通过删除、替换密文分组可以对明文进行<br />对包含某些比特错误的密文进行解密时，对应的分组会出错<br />不能抵御重放攻击 | 不应使用                             |
| CBC  | 密文分组链接模式 | 明文的重复排列不会反映在密文中<br />支持并行计算（仅解密）<br />能够解密任意密文分组 | 对包含某些错误比特的密文进行解密时，第—个分组的全部比特以及后一个分组的相应比特会出错，<br />加密不支持并行计算 | CRYPTREC推荐<br />《实用密码学》推荐 |
| CFB  | 密文反馈模式     | 不需要填充<br />支持并行运算（仅解密）<br />能够解密任意密文分组 | 加密不支持并行计算<br />对包含某些错误比特的密文进行解密时，第一个分组的全部比特以及后一个分组的相应比特会出错<br />不能抵御重放攻击 | CRYPTREC推荐                         |
| OFB  | 输出反馈模式     | 不需要填充<br />可事先进行加密解密的准备<br />加密解密使用相同的结构<br />对包含某些错误比特的密文进行解密时，只有明文中相应的比特会出错 | 不支持并行运算<br />主动攻击者反转密文分组中的某些比特时，明文分组中相应比特也会反转 | CRYPTREC推荐                         |
| CTR  | 计数器模式       | 不需要填充<br />可事先进行加密解密的准备<br />加密解密使用相同的结构<br />对包含某些错误比特的密文进行解密时，只有明文中相应的比特会出错<br />支持加解密并行运算 | 主动攻击者反转密文分组中的某些比特时，明文分组中相应比特也会反转 | CRYPTREC推荐<br />《实用密码学》推荐 |

Alice和Bob持有相同的共享秘钥，通过秘钥加解密数据。对称加密算法包括AES、DES和3DES等多种算法。对称加密算法需要对数据进行分组。

### 对称加密算法

### 分组密码模式

#### ECB Electric Codebook

电子密码本模式，明文与密文一一对应，有明显缺陷。
![](/img/post_pics/mbedtls/2.png)

#### CBC Cipher Block Chaining

密码分组链接模式，每一组明文在加密前都与前面的密文分组进行**异或**操作。与第一个分组进行异或的“密文分组”称为初始化向量IV。初始化向量一般由伪随机数生成器派生，IV不可泄露。CBC模式无法抵御选择密文攻击，当密文遭到破坏时，后面的内容无法解密。

![](/img/post_pics/mbedtls/3.png)

#### CTR

计数器模式，将累加的计数器与秘钥生成秘钥流，再进行异或。计数器值与分组长度相同，最后一个分组长度不满则截取有效部分。
![](/img/post_pics/mbedtls/4.png)

CTR的优势：

- 可以并行运算
- 可以随机访问，部分密文破坏不影响整体
- 简单，解密等同于加密，无需两套算法
  
### AES

AES是一个对称分组加密算法，分组大小为128bit，密钥长度为128（轮数10）、192（轮数12）、256（轮数14）位。
![](/img/post_pics/mbedtls/5.png)

AES加密过程，
![](/img/post_pics/mbedtls/6.png)

#### AES计算过程

- 字节替换--唯一非线性操作
- 行移位
- 列混合
- 轮秘钥加法--异或
- 轮秘钥生成
  
#### AES加密工具

##### aescrypt2

主要调用`mbedtls_aes_init`，`mbedtls_aes_setkey_enc`，`mbedtls_aes_setkey_dec`等接口，使用方法：

```bash
### mode = 1解密， mode = 0加密
aescrypt2 <mode> <input file> <output file> <hex:key>
```

##### crypt_and_hash

支持加密算法和单项散列函数，将结果输出至文件。代码参考`programs/aes/crypt_and_hash.c`

```bash
crypt_and_hash <mode> <input file> <output file> <cipher-alg> <hash-alg> <key>
crypt_and_hash 0 file file.aes AES-128-CBC SHA256 hex:xxxxxxxxxxxxx
```

### 接口与代码示例

#### 算法

| 算法 | 说明                            |
| :--- | :------------------------------ |
| AES  | 支持ECB、CBC、CTR、CFB和GCM模式 |
| DES  | 支持ECB、CBC模式                |
除此之外，还有ARCFOUR(RC4)、Blowfish、Camellia、XTEA等算法。

#### 宏定义

| 宏定义                           | 描述                                    |
| :------------------------------- | :-------------------------------------- |
| MBEDTLS_AES_C                    | 开启AES算法                             |
| MBEDTLS_CIPHER_MODE_CBC          | 开启CBC模式                             |
| MBEDTLS_CIPHER_MODE_CTR          | 开启CTR模式                             |
| MBEDTLS_AES_ROM_TABLES           | 使用预定义S盒（字节替换使用，节省空间） |
| MBEDTLS_CIPHER_C                 | 开启cipher接口                          |
| MBEDTLS_CIPHER_MODE_WITH_PADDING | 开启填充                                |
| MBEDTLS_CIPHER_PADDING——PKCS7    | 开启PKCS7填充                           |

#### 示例代码

接口描述：

| 接口                            | 描述                                 |
| :------------------------------ | :----------------------------------- |
| mbedtls_cipher_init             | 初始化cipher结构体                   |
| mbedtls_cipher_info_from_string | 通过算法名称获取cipher信息结构体指针 |
| mbedtls_cipher_setup            | 设置cipher结构体                     |
| mbedtls_cipher_get_name         | 获取算法名称                         |
| mbedtls_cipher_get_block_size   | 获取算法分组长度                     |
| mbedtls_cipher_setkey           | 设置解密密钥接口                     |
| mbedtls_cipher_set_iv           | 设置初始化向量接口                   |
| mbedtls_cipher_update           | cipher更新接口                       |
| mbedtls_cipher_finish           | cipher完成接口                       |
| mbedtls_cipher_free             | 释放cipher结构体                     |

代码运行中需要提供明文、密钥和初始化向量。

##### main.c

```c
#include <string.h>
#include <stdio.h>
#include <stdint.h>

#include "mbedtls/cipher.h"
#include "mbedtls/platform.h"

/*
    ### padding with pkcs7 AES_128_CBC Encrypt
    ptx = "CBC has been the most commonly used mode of operation."
    key = 06a9214036b8a15b512e03d534120006
    iv  = 3dafba429d9eb430b422da802c9fac41
    ctx = 4DDF9012D7B3898745A1ED9860EB0FA2
          FD2BBD80D27190D72A2F240C8F372A27
          63746296DDC2BFCE7C252B6CD7DD4BA8
          577E096DBD8024C8B4C5A1160CA2D3F9
*/
char *ptx = "CBC has been the most commonly used mode of operation.";
uint8_t key[16] =
{
    0x06, 0xa9, 0x21, 0x40, 0x36, 0xb8, 0xa1, 0x5b,
    0x51, 0x2e, 0x03, 0xd5, 0x34, 0x12, 0x00, 0x06
};

uint8_t iv[16] =
{
    0x3d, 0xaf, 0xba, 0x42, 0x9d, 0x9e, 0xb4, 0x30,
    0xb4, 0x22, 0xda, 0x80, 0x2c, 0x9f, 0xac, 0x41
};

static void dump_buf(char *info, uint8_t *buf, uint32_t len)
{
    mbedtls_printf("%s", info);
    for (int i = 0; i < len; i++) {
        mbedtls_printf("%s%02X%s", i % 16 == 0 ? "\n\t":" ", 
                        buf[i], i == len - 1 ? "\n":"");
    }
    mbedtls_printf("\n");
}

void cipher(int type)
{
    size_t len;
    int olen = 0;
    uint8_t buf[64];

    mbedtls_cipher_context_t ctx;
    const mbedtls_cipher_info_t *info;

    mbedtls_cipher_init(&ctx);
    info = mbedtls_cipher_info_from_type(type);

    mbedtls_cipher_setup(&ctx, info);
    mbedtls_printf("\n  cipher info setup, name: %s, block size: %d\n", 
                        mbedtls_cipher_get_name(&ctx), 
                        mbedtls_cipher_get_block_size(&ctx));

    mbedtls_cipher_setkey(&ctx, key, sizeof(key)*8, MBEDTLS_ENCRYPT);
    mbedtls_cipher_set_iv(&ctx, iv, sizeof(iv));
    mbedtls_cipher_update(&ctx, ptx, strlen(ptx), buf, &len);
    olen += len;

    mbedtls_cipher_finish(&ctx, buf + len, &len);
    olen += len;

    dump_buf("\n  cipher aes encrypt:", buf, olen);

    mbedtls_cipher_free(&ctx);
}

int main(void)
{
    cipher(MBEDTLS_CIPHER_AES_128_CBC);
    cipher(MBEDTLS_CIPHER_AES_128_CTR);

    return 0;
}

```

与单向散列函数计算过程相似，这里使用setkey和setiv代替start，而update可以多次调用形成数据流，在目标板内存受限时减小资源的压力，finish函数则可以结束运算。

##### mbedtls_config.h

```c
#ifndef MBEDTLS_CONFIG_H
#define MBEDTLS_CONFIG_H

/* System support */
#define MBEDTLS_PLATFORM_C
#define MBEDTLS_PLATFORM_MEMORY
#define MBEDTLS_MEMORY_BUFFER_ALLOC_C
#define MBEDTLS_PLATFORM_NO_STD_FUNCTIONS
#define MBEDTLS_PLATFORM_EXIT_ALT
#define MBEDTLS_NO_PLATFORM_ENTROPY
#define MBEDTLS_NO_DEFAULT_ENTROPY_SOURCES
#define MBEDTLS_PLATFORM_PRINTF_ALT

/* mbed TLS modules */
#define MBEDTLS_AES_C
#define MBEDTLS_CIPHER_C
#define MBEDTLS_CIPHER_MODE_CBC
#define MBEDTLS_CIPHER_MODE_CTR
#define MBEDTLS_CIPHER_MODE_WITH_PADDING
#define MBEDTLS_CIPHER_PADDING_PKCS7

#define MBEDTLS_AES_ROM_TABLES

#include "mbedtls/check_config.h"

#endif /* MBEDTLS_CONFIG_H */

```

##### CMakeLists.txt

```bash
cmake_minimum_required(VERSION 3.8)

project("aes")

include_directories(./ $ENV{MBEDTLS_BASE}/include)
aux_source_directory($ENV{MBEDTLS_BASE}/library MBEDTLS_SOURCES)


set(SOURCES 
 ${CMAKE_CURRENT_LIST_DIR}/main.c
    ${CMAKE_CURRENT_LIST_DIR}/mbedtls_config.h 
 ${MBEDTLS_SOURCES})

add_executable(aes ${SOURCES})
```

运行结果：

```bash
mkdir -p build && cd build && cmake .. && make -j32
./aes               

  cipher info setup, name: AES-128-CBC, block size: 16

  cipher aes encrypt:
        4D DF 90 12 D7 B3 89 87 45 A1 ED 98 60 EB 0F A2
        FD 2B BD 80 D2 71 90 D7 2A 2F 24 0C 8F 37 2A 27
        63 74 62 96 DD C2 BF CE 7C 25 2B 6C D7 DD 4B A8
        57 7E 09 6D BD 80 24 C8 B4 C5 A1 16 0C A2 D3 F9


  cipher info setup, name: AES-128-CTR, block size: 16

  cipher aes encrypt:
        C4 1A 1D B1 56 C0 9B 59 E8 25 D9 5B 72 FD 97 BE
        F7 06 BA C1 B8 4F F5 4E 72 88 2D 17 0B DB 53 0A
        9B 0A FD 86 41 65 73 06 6B C1 F0 52 18 FC 1D 57
        9D F4 81 F7 08 CB
```

##### cipher相关结构体

```c
typedef struct mbedtls_cipher_context_t
{
    /** Information about the associated cipher. */
    const mbedtls_cipher_info_t *cipher_info;

    /** Key length to use. */
    int key_bitlen;

    /** Operation that the key of the context has been
     * initialized for.
     */
    mbedtls_operation_t operation;

#if defined(MBEDTLS_CIPHER_MODE_WITH_PADDING)
    /** Padding functions to use, if relevant for
     * the specific cipher mode.
     */
    void (*add_padding)( unsigned char *output, size_t olen, size_t data_len );
    int (*get_padding)( unsigned char *input, size_t ilen, size_t *data_len );
#endif

    /** Buffer for input that has not been processed yet. */
    unsigned char unprocessed_data[MBEDTLS_MAX_BLOCK_LENGTH];

    /** Number of Bytes that have not been processed yet. */
    size_t unprocessed_len;

    /** Current IV or NONCE_COUNTER for CTR-mode, data unit (or sector) number
     * for XTS-mode. */
    unsigned char iv[MBEDTLS_MAX_IV_LENGTH];

    /** IV size in Bytes, for ciphers with variable-length IVs. */
    size_t iv_size;

    /** The cipher-specific context. */
    void *cipher_ctx;

#if defined(MBEDTLS_CMAC_C)
    /** CMAC-specific context. */
    mbedtls_cmac_context_t *cmac_ctx;
#endif

#if defined(MBEDTLS_USE_PSA_CRYPTO)
    /** Indicates whether the cipher operations should be performed
     *  by Mbed TLS' own crypto library or an external implementation
     *  of the PSA Crypto API.
     *  This is unset if the cipher context was established through
     *  mbedtls_cipher_setup(), and set if it was established through
     *  mbedtls_cipher_setup_psa().
     */
    unsigned char psa_enabled;
#endif /* MBEDTLS_USE_PSA_CRYPTO */

} mbedtls_cipher_context_t;
```

```c
typedef struct mbedtls_cipher_info_t
{
    /** Full cipher identifier. For example,
     * MBEDTLS_CIPHER_AES_256_CBC.
     */
    mbedtls_cipher_type_t type;

    /** The cipher mode. For example, MBEDTLS_MODE_CBC. */
    mbedtls_cipher_mode_t mode;

    /** The cipher key length, in bits. This is the
     * default length for variable sized ciphers.
     * Includes parity bits for ciphers like DES.
     */
    unsigned int key_bitlen;

    /** Name of the cipher. */
    const char * name;

    /** IV or nonce size, in Bytes.
     * For ciphers that accept variable IV sizes,
     * this is the recommended size.
     */
    unsigned int iv_size;

    /** Bitflag comprised of MBEDTLS_CIPHER_VARIABLE_IV_LEN and
     *  MBEDTLS_CIPHER_VARIABLE_KEY_LEN indicating whether the
     *  cipher supports variable IV or variable key sizes, respectively.
     */
    int flags;

    /** The block size, in Bytes. */
    unsigned int block_size;

    /** Struct for base cipher information and functions. */
    const mbedtls_cipher_base_t *base;

} mbedtls_cipher_info_t;
```

## 5 公钥密码

- 密钥配送问题解决方法：
  - 事先共享
  - 密钥分配中心KDC
  - 密钥交换
  - 公钥密码
- 接收者将公钥发送给发送者，发送者使用公钥加密后将密文传递给接收者，接收者使用对应的私钥进行解密
- 公钥密码存在的问题，公钥认证问题，中间人攻击：窃听者给发送者错误的公钥并获取明文，再将明文篡改后使用正确的公钥加密返回给接收者从而实现攻击
- RSA算法

$$
密文=明文^E mod N,(E,N)是公钥\\
明文=密文^D mod N,(D,N)是私钥
$$

- 算法：生成密钥对，准备两个很大的质数p和q

$$
N=p*q\\
L=lcm(p-1,q-1),lcm表示求最小公倍数\\
1<E<L,gcd(E,L) = 1,gcd表示求最大公约数（使用辗转相除法）\\
1<D<L,E*DmodL=1
$$

- RSA攻击方法：一旦发现了对大整数进行质因数分解的高效算法， RSA 就能够被破译
  - 中间人攻击
  - 选择密文攻击--RSA-OAEP
- 其他公钥密码
  - EIGamal
  - Rabin
  - ECC，椭圆曲线加密
  
## 6 混合密码系统

- 背景：
  - 公钥密码不足：计算效率低，但可以解决密钥配送问题
  - 对称密码不足：密钥配送麻烦，但是速度快
- 混合密码系统
  - 概念：使用公钥密码保护对称密码密钥
  - 加密过程：明文使用对称密码密钥加密，对称密码密钥使用公钥加密，两者发送给接收者
  - 解密过程：对称密码密钥使用私钥解密，密文使用对称密码密钥解密

## 7 单向散列函数

- 文件、密文是否被篡改：完整性、一致性，integrity
- 单向散列函数有一个输入和输出，输入称为消息，输出称为散列值
  - 根据任意长度的消息计算出固定长度的散列值，无论消息的长度如何，散列值的大小应当不变
  - 能够快速计算出散列值
  - 消息不同散列值也不同，理论上消息变化一个比特，散列值应当有明显变化以确定消息是否被篡改
  - 两个不同的消息产生同一个散列值的情况称为**碰撞**(collision )，难以发现碰撞的性质称为抗碰撞性(collision resistance)
  - 单向散列函数必须确保要找到和该条消息具有相同散列值的另外一条消息是非常困难的。这一性质称为**弱抗碰撞性**。单向散列函数都必须具备弱抗碰撞性。
  - **强抗碰撞性**。所谓强抗碰撞性，是指要找到散列值相同的两条不同的消息是非常困难的这一性质。在这里，散列值可以是任意值。
  - 单向散列函数必须具备单向性(one-way )。单向性指的是无法通过散列值反算出消息的性质。
- 单向散列函数又称为哈希函数
- 实际应用
  - 检测软件是否被篡改
  - 基于口令的加密
  - 消息认证码
  - 数字签名
  - 伪随机数生成器
  - 一次性口令
- 常见的单向散列函数
  - MD4、MD5，强对抗性已被攻破
  - SHA-1，不安全
  - SHA-256、SHA-384、SHA-521（SHA-2）
  - RIPEMD-160
  - SHA-3：Keccak算法
- 对单向散列函数的攻击
  - 暴力破解，针对弱抗碰撞性
  - 先准备两个散列值相同的不同消息，针对强抗碰撞性
- 单向散列函数无法解决的问题
  - 无法验证伪装，只能识别是否篡改
  - 使用认证技术可以识破伪装，认证技术包括消息验证码和数字签名

### 原理

满足密码学算法安全属性的特殊散列函数，保证数据的完整性，抵御篡改攻击，原数据称作**消息**，计算后的输出称为**摘要**，在通信中，Bob接受Alice发来的消息及摘要，通过相同的算法计算摘要比对以验证消息是否被篡改。

### 性质

输入长度可变，输出长度固定，计算过程应当高效率，具备单向性。

- 抗弱碰撞性，对于给定$a$，能找到$b$使得$h(a)=h(b)$不可行
- 抗强碰撞性，给定一个输出$z$，能够找到$h(a)=z$不可行
  
### 应用

消息完整性检测、伪随机数生成器、数字签名、消息认证码

### MD算法

- MD4：散列碰撞已被攻破
- MD5：抗碰撞性已被攻破
  
### SHA算法

SHA0/SHA1不再使用

- SHA2：包括SHA256，SHA384，SHA512
- SHA3：更为安全
  
#### SHA256

处理步骤：

- 预处理
  - 消息填充
  - 消息分割
  - 设置初始摘要值 $H^{(0)}$
  - 准备常量$K^{256}_{i}$
- 哈希计算
  - 消息调度
  - 初始化工作寄存器
  - 更新工作寄存器
  - 计算消息摘要

### generic_sum

计算散列值

```bash
generic_sum SHA256 <file name>
```

### 头文件和函数接口

```c
#include "mbedtls/md.h"
```

| 接口                      | 描述                                       |
| :------------------------ | :----------------------------------------- |
| mbedtls_md_init           | 初始化md结构体                             |
| mbedtls_md_info_from_type | 根据算法类型得到md信息结构体指针           |
| mbedtls_md_setup          | 初始化md结构体                             |
| mbedtls_md_get_name       | 获得单向散列算法名称                       |
| mbedtls_md_get_size       | 获取单向散列算法输出消息摘要长度           |
| mbedtls_md_starts         | md启动接口                                 |
| mbedtls_md_update         | md更新接口，处理输入数据，包括预处理和计算 |
| mbedtls_md_finish         | md输出消息摘要结果                         |
| mbedtls_md_free           | 释放md结构体                               |

### 依赖

```c
#define MBEDTLS_MD_C
#define MBEDTLS_SHA256_C
```

### 散列示例代码

```c
#include <stdio.h>
#include <string.h>
#include <stdint.h>

#include "mbedtls/md.h"
#include "mbedtls/platform.h"

static void dump_buf(char *info, uint8_t *buf, uint32_t len)
{
    mbedtls_printf("%s", info);
    for (int i = 0; i < len; i++) {
        mbedtls_printf("%s%02X%s", i % 16 == 0 ? "\n\t":" ", 
                        buf[i], i == len - 1 ? "\n":"");
    }
    mbedtls_printf("\n");
}

int main(void)
{
    uint8_t digest[32];
    char *msg = "abc";

    mbedtls_md_context_t ctx;
    const mbedtls_md_info_t *info;

    mbedtls_md_init(&ctx);
    info = mbedtls_md_info_from_type(MBEDTLS_MD_SHA256);

    mbedtls_md_setup(&ctx, info, 0);
    mbedtls_printf("\n  md info setup, name: %s, digest size: %d\n", 
                   mbedtls_md_get_name(info), mbedtls_md_get_size(info));

    mbedtls_md_starts(&ctx);
    mbedtls_md_update(&ctx, msg, strlen(msg));
    mbedtls_md_finish(&ctx, digest);

    dump_buf("\n  md sha-256 digest:", digest, sizeof(digest));

    mbedtls_md_free(&ctx); 

    return 0;
}
```

config头文件

```c
#ifndef MBEDTLS_CONFIG_H
#define MBEDTLS_CONFIG_H

/* System support */
#define MBEDTLS_PLATFORM_C
#define MBEDTLS_PLATFORM_MEMORY
#define MBEDTLS_MEMORY_BUFFER_ALLOC_C
#define MBEDTLS_PLATFORM_NO_STD_FUNCTIONS
#define MBEDTLS_PLATFORM_EXIT_ALT
#define MBEDTLS_NO_PLATFORM_ENTROPY
#define MBEDTLS_NO_DEFAULT_ENTROPY_SOURCES
#define MBEDTLS_PLATFORM_PRINTF_ALT

/* mbed TLS modules */
#define MBEDTLS_MD_C
#define MBEDTLS_SHA256_C

#include "mbedtls/check_config.h"

#endif /* MBEDTLS_CONFIG_H */
```

CMakeLists.txt

```c
cmake_minimum_required(VERSION 3.8)

project("hash")

include_directories(./ $ENV{MBEDTLS_BASE}/include)
aux_source_directory($ENV{MBEDTLS_BASE}/library MBEDTLS_SOURCES)


set(SOURCES 
    ${CMAKE_CURRENT_LIST_DIR}/main.c
    ${CMAKE_CURRENT_LIST_DIR}/mbedtls_config.h 
    ${MBEDTLS_SOURCES})

add_executable(hash ${SOURCES})
```

运行结果：

```bash
mkdir -p build && cd build && cmake .. && make -j32
./hash    

  md info setup, name: SHA256, digest size: 32

  md sha-256 digest:
        BA 78 16 BF 8F 01 CF EA 41 41 40 DE 5D AE 22 23
        B0 03 61 A3 96 17 7A 9C B4 10 FF 61 F2 00 15 AD
```

实际应用中，`mbedtls_md_update`可以调用多次，不断填充数据。

## 8 消息认证码

- 概述：可以判断消息是否被篡改，并且发送者是否被伪装
- 消息认证码，Message Authentication Code，MAC，是一种确认完整性并进行认证的技术
  - 消息认证码的输入包括任意长度的**消息**和一个发送者与接收者之间**共享的密钥**，它可以输出固定长度的数据，这个数据称为MAC 值。
  - 要计算MAC 必须持有共享密钥，没有共享密钥的人就无法计算MAC 值，消息认证码正是利用这一性质来完成认证的。消息认证码是一种与密钥相关联的单向散列函数。
- 使用过程：发送者将密钥和消息计算得到消息认证码MAC，将消息和认证码发送给接收者，接收者根据消息和密钥计算MAC并与发送者提供的MAC进行比对，如果不一致则认证失败
- 消息认证码需要解决共享密钥的配送问题
- 常用消息认证码技术
  - SWIFT
  - IPsec
  - SSL/TLS
- 实现手段：
  - SHA-2、HMAC等单向散列函数
  - 分组密码AES
- 认证加密AEAD，结合对称密码和消息认证码，同时满足CIA即，机密性、完整性和认证
- Encrypt-then-MAC，先加密再计算密文的MAC，除此之外还有Encrypt-and-MAC (将明文用对称密码加密，并对明文计算MAC 值）和MAC-then-Encrypt (先计算明文的MAC 值，然后将明文和MAC 值同时用对称密码加密）。
- HMAC，使用单向散列函数构造消息认证码的方法
- 对消息认证码的攻击
  - 重放攻击：攻击者将窃取的MAC值和密文进行重新发送
    - 抵御方法：序号、时间戳、接收者提前发送nonce随机数并需要发送者将随机数包含在明文中
  - 密钥推测攻击
    - 根据单向散列值推测出对称密钥
- 消息认证码无法解决的问题
  - 第三方证明，因为第三方不知道密钥所以无法验证，需要签名技术
  - 防止否认，发送者可以否认发送，接收者无法证明是否发送

MAC，Message Authentication Code，帮助接收者判断消息是否被第三方篡改，确保消息的完整性和真实性。常使用单向散列函数实现，称为HMAC-SHA1和HMAC-SHA256。也可以使用分组加密构建，称为CMAC、GCM和CCM。MAC和Hash的不同在于输入多一个密钥。

### 消息认证码

![](/img/post_pics/mbedtls/7.png)

### 实现形式

- HMAC：HMAC是基于单向散列函数的算法

- CBC-MAC和CMAC
- 认证加密CCM，输入包括明文、一次性整数Nonce、相关数据A和密钥K，输出密文C和认证码T，使用的算法是CBC-MAC
  
![](/img/post_pics/mbedtls/8.png)

- 认证加密GCM：输入包括明文、初始化向量IV、相关数据A和密钥K，输出密文C和认证码T，使用的算法是CBC-MAC
  
![](/img/post_pics/mbedtls/9.png)

### HMAC示例

#### 依赖的宏

MBEDTLS_MD_C
MBEDTLS_SHA256_C

#### 函数接口

| **接口**               | **描述**     |
| :--------------------- | :----------- |
| mbedtls_md_hmac_starts | 启动HMAC计算 |
| mbedtls_md_hmac_update | 更新输入数据 |
| mbedtls_md_hmac_finish | 完成计算     |

#### 代码示例

运行平台K210，移植参考：[mbedtls学习--移植库至K210平台](https://blog.csdn.net/JackSparrow_sjl/article/details/118583913)

```c

#include <stdint.h>
#include <stdio.h>
#include <string.h>
#include "pin_config.h"

#include "mbed_platform.h"
#include "md.h"

void hardware_init(void)
{
    // fpioa映射
    fpioa_set_function(PIN_UART_USB_RX, FUNC_UART_USB_RX);
    fpioa_set_function(PIN_UART_USB_TX, FUNC_UART_USB_TX);
}
//************************* Mbedtls code start *****************************

static void dump_buf(char *info, uint8_t *buf, uint32_t len)
{
    mbedtls_printf("%s", info);
    for(int i = 0; i < len; i++)
    {
        mbedtls_printf("%s%02X%s", i % 16 == 0 ? "\n     " : " ",
                       buf[i], i == len - 1 ? "\n" : "");
    }
}

//************************* Mbedtls code end *******************************

int main(void)
{
    hardware_init();
    // 初始化串口3，设置波特率为115200
    uart_init(UART_USB_NUM);
    uart_configure(UART_USB_NUM, 115200, UART_BITWIDTH_8BIT, UART_STOP_1, UART_PARITY_NONE);

    /* 开机发送hello yahboom! */
    char *log = {"Run mbedtls on K210\n"};
    uart_send_data(UART_USB_NUM, log, strlen(log));

    //******************************* mbedtls code start *******************

    uint8_t mac[32];
    char *secret = "Jefe";
    char *msg = "what do ya want for nothing?";

    mbedtls_md_context_t ctx;
    const mbedtls_md_info_t *info;

    mbedtls_md_init(&ctx);
    info = mbedtls_md_info_from_type(MBEDTLS_MD_SHA256);

    mbedtls_md_setup(&ctx, info, 1);
    mbedtls_printf("\n  md info setup, name: %s, digest size: %d\n",
                   mbedtls_md_get_name(info), mbedtls_md_get_size(info));

    mbedtls_md_hmac_starts(&ctx, secret, strlen(secret));
    mbedtls_md_hmac_update(&ctx, msg, strlen(msg));
    mbedtls_md_hmac_finish(&ctx, mac);

    dump_buf("\n  md hmac-sha-256 mac:", mac, sizeof(mac));

    mbedtls_md_free(&ctx);

    //******************************* mbedtls code end **********************

    while(1)
    {
    }
    return 0;
}

```

运行结果：
![](/img/post_pics/mbedtls/10.png)

## 9 数字签名

数字签名digital signature，是一种将相当于现实世界中的盖章、签字的功能在计算机世界中进行实现的技术。使用数字签名可以识别篡和伪装，还可以防止否认。

- 主要行为：
  - 发送者Alice生成消息签名，根据消息内容计算数字签名的值
  - 接收者Bob或者第三方Victor验证消息签，检查该消息的签名是否真的属于A lice ,
- Alice 使用“签名密钥”来生成消息的签名，而Bob 和Victor 则使用“验证密钥”来验证消息的签名。数字签名对签名密钥和验证密钥进行了区分，使用验证密钥是无法生成签名的。这一点非常重要。此外，签名密钥只能由签名的人持有，而验证密钥则是任何需要验证签名的人都可以持有。数字签名的实现原理与公钥密码原理相反。

|          | 私钥                           | 公钥                           |
| -------- | ------------------------------ | ------------------------------ |
| 公钥密码 | 接收者解密时使用               | 发送者加密使用                 |
| 数字签名 | 签名者生成签名时使用（发送者） | 验证者验证签名时使用（接收者） |
| 谁持有   | 接收者/签名者个人持有          | 任何人持有                     |

- 用公钥加密所得到的密文，只能用与该公钥配对的私钥才能解密；同样地，用私钥加密所得到的密文，也只能用与该私钥配对的公钥才能解密。也就是说，如果用某个公钥成功解密了密文，那么就能够证明这段密文是用与该公钥配对的私钥进行加密所得到的。
- Alice可以用私钥解密任何人用相对应的公钥加密的密文，相反，任何人都可以用公钥验证Alice用对应的私钥生成的签名。
- 数字签名的方法：
  - 直接对消息签名
  - 对消息的单向散列值签名
    - Alice 用单向散列函数计算消息的散列值。
    - Alice 用自己的私钥对散列值进行加密。
    - Alice 将消息（未加密）和签名发送给Bob 。
    - Bob 用Alice 的公钥对收到的签名进行解密。
    - Bob 将签名解密后得到的散列值与Al ice 直接发送的消息的散列值进行对比。
- 数字签名不是为了保证Alice消息的机密性，而是确保其他人能认证该消息来自于Alice。无论签名被复制多少份，都能确定消息的来源。如果有人篡改签名和消息，至少会造成签名验证失败。
- 应用实例：
  - 安全信息公告
  - 软件下载
  - 对公钥进行签名，确认公钥的合法性
- 数字签名实现：RSA算法，ECDSA算法基于椭圆曲线密码
- 对数字签名的攻击
  - 中间人攻击，需要对公钥进行签名
  - 利用数字签名攻击公钥密码，将密文让私钥持有者签名从而获得明文
  - 潜在伪造
- 数字签名无法解决的问题：公钥认证问题

## 10 证书

证书Public-Key Certificate, PKC，为公钥加上数字签名。认证机构就是能够认定“公钥确实属千此入”并能够生成数字签名的个人或者组织。认证机构中有国际性组织和政府所设立的组织，也有通过提供认证服务来盈利的一般企业，此外个人也可以成立认证机构。Bob在认证机构Trent注册自己的公钥，Alice使用经认证的公钥加密消息给Bob。对证书的攻击：

- 在公钥注册前攻击
- 注册相似人名
- 窃取认证机构的私钥
- 伪装成认证机构攻击

## 11 密钥

- 密钥长度决定密钥空间的大小
- 密钥与明文是等价的
- 密钥分为用于机密性的密钥和用于认证的密钥，会话密钥（一次性）和主密钥，加密内容的密钥和加密密钥的密钥
- 密钥生成：
  - 用随机数生成密钥，密码学用途的伪随机数生成器必须是专门针对密码学用途而设计的，必须具备不可预测性
  - 用口令（password）生成密钥
- 配送密钥：事先共享，密钥分配中心，公钥密码，Diffie-Hellman密钥交换
- 更新密钥：（参考《信息安全工程》Anderson著）用当前密钥的散列值作为下一个密钥。
- 密钥保存：将密钥加密后保存KEK
- Diffie-Hellman 密钥交换：基于有限域的离散对数难以计算所实现
  - Alic给Bob一个大质数P和生成元G
  - Alice和Bob各在1~P-2的区间内生成一个随机数并保密
  - Alic发送$G^A mod P$给Bob
  - Bob发送$G^B mod P$给Alice
  - Alice计算$(G^B mod P)^A mod P$作为密钥
  - Bob计算$(G^A mod P)^B mod P$作为密钥，与Alice相同
  - 生成元的计算方法：
    - 设13为P，则任取G在0-P-1内，A在1~P-1的范围内，生成元$G^A mod P$应当与A一一对应的关系
    - 例如2的1次方到12次方取13的模与1-12一一对应因此2是13的生成元
- 基于口令的密码Password Based Encryption
  - 生成KEK，用口令和伪随机生成的盐计算单向散列函数值生成KEK
  - 生成会话密钥并加密，用KEK加密，加密后保存盐和加密后的会话密钥，并丢弃KEK（可根据口令和盐重新生成）
  - 加密消息
- 盐：盐是由伪随机数生成器生成的随机数，在生成密钥(KEK ) 时会和口令一起被输入单向散列函数。
  - 盐用来抵御字典攻击
- 拉伸：使用多次单向散列函数计算KEK，增加攻击者的成本

## 12 随机数

- 应用场景：
  - 生成密钥和密钥对
  - 在CBC、CFB、OFB中生成初始化向量
  - 生成nonce
  - 生成盐
- 随机数的分类
  - 随机性--弱伪随机数
    - 不存在统计学偏差
  - 不可预测性--强伪随机数
    - 在攻击者知道之前数据的情况下依旧无法预测下一个随机数
    - 不可预测性是通过使用其他的密码技术来实现
  - 不可重现性--真随机数
    - 计算机所有能产生的随机数会存在周期，就可以重现
    - 不可重现的随机数需要从不可重现的物理现象中获取信息
- 密码学中的随机数需要具备不可预测性，只具备随机性的随机数是不够的，”杂乱无章也可以被看穿“
- 伪随机数生成器：从外部输入种子seed进入内部状态初始化
  - 线性同余法--不具备不可预测性
    - 将当前的伪随机数值乘以A 再加上C, 然后将除以M 得到的余数作为下一个伪随机数。
    - 可以根据内部状态推测下一个伪随机数，且具备周期性
    - C语言的rand函数也是基于线性同余法
  - 单向散列函数法
    - 用种子初始化内部状态计数器
    - 根据计数器值计算散列值作为随机数
    - 计数值++
    - 继续输出随机数
    - 由于攻击者无法根据散列值推算出计数器的状态，因此无法预测下一个随机数，**单向散列函数的单向性是支撑伪随机数生成器不可预测性的基础。**
  - 密码法
    - 将单向散列函数法中的函数计算过程改为密钥加密过程，**密码的机密性是支撑伪随机数生成器不可预测性的基础。**
  - ANSI X9.17：PGP密码软件的基础
- 对伪随机数生成器的攻击：
  - 对种子进行攻击
  - 对随机数池进行攻

### 随机数应用

- 生成盐，用于基于口令的密码
- 生成密钥，用于加密和认证
- 生成一次性整数Nonce，防止重放攻击
- 生成初始化向量IV

### 构成

- 种子，真随机数生成器的种子来源于物理现象
- 内部状态，种子用来初始化内部状态
  
### 随机数算法

| 算法      | 原理             |
| :-------- | :--------------- |
| Hash_DRBG | 使用单向散列函数 |
| HMAC_DRBG | 使用消息认证码   |
| CTR_DRBG  | 使用分组密码算法 |

### 工具

```bash
gen_entropy file  # 产生熵
gen_random_ctr_drbg file # 生成随机数
gen_random_havege file # 使用硬件时钟源生成伪随机数，依赖MBEDTLS_HAVEGE_C
```

### CTR_DRBG

![fig1](/img/post_pics/mbedtls/1.png)
攻击时，由于无法破解明文获取内部状态，得到计数值，因此无法预测下一个随机数。

## PGP

Pretty Good Privacy密码软件

## SSL/TLS

Secure Socket Layer和Transport Layer Security，用于安全通信的密码通信方法。

## 补充

### 大数运算

大数计算，顾名思义，指超出64位的数的乘法运算、指数运算和模逆运算，其中模逆运算，特指求逆元，所谓乘法逆元，例如：
$$
2*9 mod 17 = 1
$$
则9是2关于模17的逆元（余数为1的被除数）或者2 * 9 与 1 关于模17同余即：
$$
9 = 2^{-1} mod 17
$$
  
#### 库文件

mbedtls/library/bignum.c

#### 依赖宏

MBEDTLS_BIGNUM_C
MBEDTLS_PLATFORM_C

#### 接口

| 接口                     | 描述                   |
| :----------------------- | :--------------------- |
| mbedtls_mpi_init         | 初始化大数结构体       |
| mbedtls_mpi_read_string  | 读取字符串到大数结构体 |
| mbedtls_mpi_write_string | 大数结构体输出到字符串 |
| mbedtls_mpi_mul_mpi      | 大数乘法               |
| mbedtls_mpi_exp_mod      | 大数指数               |
| mbedtls_mpi_inv_mod      | 大数模逆               |
| mbedtls_mpi_free         | 释放大数结构体         |

#### 大数运算示例代码

代码结构

```bash
.
├── build
├── CMakeLists.txt
├── main.c
└── mbedtls_config.h    #对使用的库的裁剪，包含需要的库
```

main.c

```c
#include <string.h>
#include <stdio.h>

#include "mbedtls/bignum.h"
#include "mbedtls/platform.h"

static void dump_buf(char *buf, size_t len) 
{
    for (int i = 0; i < len; i++) {
        mbedtls_printf("%c%s", buf[i], 
                        (i + 1) % 32 ? "" : "\n\t"); 
    }
    mbedtls_printf("\n");
}

int main(void)
{
    size_t olen;
    char buf[256];
    mbedtls_mpi A, E, N, X;

    mbedtls_mpi_init(&A); 
    mbedtls_mpi_init(&E); 
    mbedtls_mpi_init(&N); 
    mbedtls_mpi_init(&X);

    mbedtls_mpi_read_string(&A, 16,     //以16进制读取字符串，允许2-16进制
        "EFE021C2645FD1DC586E69184AF4A31E" \
        "D5F53E93B5F123FA41680867BA110131" \
        "944FE7952E2517337780CB0DB80E61AA" \
        "E7C8DDC6C5C6AADEB34EB38A2F40D5E6" );

    mbedtls_mpi_read_string(&E, 16,
        "B2E7EFD37075B9F03FF989C7C5051C20" \
        "34D2A323810251127E7BF8625A4F49A5" \
        "F3E27F4DA8BD59C47D6DAABA4C8127BD" \
        "5B5C25763222FEFCCFC38B832366C29E" );

    mbedtls_mpi_read_string(&N, 16,
        "0066A198186C18C10B2F5ED9B522752A" \
        "9830B69916E535C8F047518A889A43A5" \
        "94B6BED27A168D31D4A52F88925AA8F5" );

    mbedtls_mpi_mul_mpi(&X, &A, &N);
    mbedtls_mpi_write_string(&X, 16, buf, 256, &olen);
    mbedtls_printf("\n  X = A * N = \n\t");
    dump_buf(buf, olen);

    mbedtls_mpi_exp_mod(&X, &A, &E, &N, NULL);
    mbedtls_mpi_write_string(&X, 16, buf, 256, &olen);
    mbedtls_printf("\n  X = A^E mode N = \n\t");
    dump_buf(buf, olen);

    mbedtls_mpi_inv_mod( &X, &A, &N);
    mbedtls_mpi_write_string(&X, 16, buf, 256, &olen);
    mbedtls_printf("\n  X = A^-1 mod N = \n\t");
    dump_buf(buf, olen);

    mbedtls_mpi_free(&A); 
    mbedtls_mpi_free(&E);
    mbedtls_mpi_free(&N); 
    mbedtls_mpi_free(&X);

    return 0;   
}
```

CmakeList.txt

```bash
cmake_minimum_required(VERSION 3.8)

project("bignum")

include_directories(./ $ENV{MBEDTLS_BASE}/include)
aux_source_directory($ENV{MBEDTLS_BASE}/library MBEDTLS_SOURCES)

set(SOURCES 
 ${CMAKE_CURRENT_LIST_DIR}/main.c
    ${CMAKE_CURRENT_LIST_DIR}/mbedtls_config.h 
 ${MBEDTLS_SOURCES})

add_executable(bignum ${SOURCES})
```

mbedtls_config.h

```c
/* System support */
#define MBEDTLS_PLATFORM_C
#define MBEDTLS_PLATFORM_MEMORY
#define MBEDTLS_MEMORY_BUFFER_ALLOC_C
#define MBEDTLS_PLATFORM_NO_STD_FUNCTIONS
#define MBEDTLS_PLATFORM_EXIT_ALT
#define MBEDTLS_NO_PLATFORM_ENTROPY
#define MBEDTLS_NO_DEFAULT_ENTROPY_SOURCES
#define MBEDTLS_PLATFORM_PRINTF_ALT

/* mbed TLS modules */
#define MBEDTLS_BIGNUM_C

#include "mbedtls/check_config.h"

#endif /* MBEDTLS_CONFIG_H */

```

运行测试

```bash
mkdir -p build && cd build && cmake .. && make -j32
./bignum          

  X = A * N = 
        602AB7ECA597A3D6B56FF9829A5E8B85
        9E857EA95A03512E2BAE7391688D264A
        A5663B0341DB9CCFD2C4C5F421FEC814
        8001B72E848A38CAE1C65F78E56ABDEF
        E12D3C039B8A02D6BE593F0BBBDA56F1
        ECF677152EF804370C1A305CAF3B5BF1
        30879B56C61DE584A0F53A2447A51E

  X = A^E mode N = 
        36E139AEA55215609D2816998ED020BB
        BD96C37890F65171D948E9BC7CBAA4D9
        325D24D6A3C12710F10A09FA08AB87

  X = A^-1 mod N = 
        3A0AAEDD7E784FC07D8F9EC6E3BFD5C3
        DBA76456363A10869622EAC2DD84ECC5
        B8A74DAC4D09E03B5E0BE779F2DF61
```

#### 算法分析

```c
typedef struct mbedtls_mpi
{
    int s;                 /*!<  Sign: -1 if the mpi is negative, 1 otherwise */
    size_t n;              /*!<  total # of limbs  */
    mbedtls_mpi_uint *p;           /*!<  pointer to limbs  */
}
mbedtls_mpi;
```

X->s指大数的正负，X->指数位总数，X->p[n]指向每一位，大小为32位或64位数。例如十六进制数-1 000000000000000F的结果为，最低位为65535，最高位为1，这里每16个字符是一位。

```c
A->s = -1 , A->n = 2
A->[0] = 65535
A->[1] = 1
```

##### 数位统计

```c
size_t mbedtls_mpi_bitlen( const mbedtls_mpi *X )
```

该函数计算大数的bit位的总个数，忽略前导0，譬如十六进制`-1000000000000FFFF`的运算结果为65，实现过程如下：i从最高位`n-1`往后数，遇到`X->p[i] != 0`截止，例如`-1000000000000FFFF`的n=2, i=1开始，此时再计算当前位（最左位）的bit数，依靠

```c
static size_t mbedtls_clz( const mbedtls_mpi_uint x )
```

函数来计算，这个函数将数值转为二进制并去除前导0计算剩余的有效bit位的总和，返回j=1，则大数的数位综合为`( i * biL ) + j`，biL在这里值为64。

##### 读取字符串

```c
int mbedtls_mpi_read_string( mbedtls_mpi *X, int radix, const char *s )
```

radix指进制，支持2-16进制以内的读取。以十六进制为例：

```c
    mbedtls_mpi_init( &T );  //初始化一个大数
    slen = strlen( s );         //获取字符串长度
    if( radix == 16 )
    {
        if( slen > MPI_SIZE_T_MAX >> 2 )     //判断是否超出范围
            return( MBEDTLS_ERR_MPI_BAD_INPUT_DATA );

        n = BITS_TO_LIMBS( slen << 2 );

        MBEDTLS_MPI_CHK( mbedtls_mpi_grow( X, n ) );  //扩展n位
        MBEDTLS_MPI_CHK( mbedtls_mpi_lset( X, 0 ) );  //初始化每一位为0

        for( i = slen, j = 0; i > 0; i--, j++ )
        {
            if( i == 1 && s[i - 1] == '-' )     //判断字符串首位
            {
                X->s = -1;
                break;
            }

            MBEDTLS_MPI_CHK( mpi_get_digit( &d, radix, s[i - 1] ) );
            X->p[j / ( 2 * ciL )] |= d << ( ( j % ( 2 * ciL ) ) << 2 );
        }
    }
```

`i`初始为slen，也就是字符串最右边（数值上的低位），`j`初始为大数的最低位，mpi_get_digit将`i-1`位的char类型转为digit类型，这里ciL=8，所以`j / ( 2 * ciL )`意味着每16个字符为一组记为大数的1位。`d << ( ( j % ( 2 * ciL ) ) << 2 )`则表示这组16个字符每一位的数值，拆解来开，譬如字符串00001234，

```bash
i=7，d=4，j=0，左移0位
i=6，d=3，j=1，左移4位，乘以16
i=5，d=2，j=2，左移8位，乘以256
i=4，d=1，j=3，左移12位，乘以4096
```

以此类推求和。这里采用或运算而非加法。

##### 输出字符串

```c
int mbedtls_mpi_write_string( const mbedtls_mpi *X, int radix,
                              char *buf, size_t buflen, size_t *olen )
```

过程与读取相反，作者写的非常精简：

```c
    if( X->s == -1 )
    {
        *p++ = '-';
        buflen--;
    }
    if( radix == 16 )
    {
        int c;
        size_t i, j, k;

        for( i = X->n, k = 0; i > 0; i-- )
        {
            for( j = ciL; j > 0; j-- )
            {
                c = ( X->p[i - 1] >> ( ( j - 1 ) << 3) ) & 0xFF;

                if( c == 0 && k == 0 && ( i + j ) != 2 )
                    continue;

                *(p++) = "0123456789ABCDEF" [c / 16];
                *(p++) = "0123456789ABCDEF" [c % 16];
                k = 1;
            }
        }
    }
```

除此之外还有操作文件的接口：

```c
int mbedtls_mpi_read_file( mbedtls_mpi *X, int radix, FILE *fin )
int mbedtls_mpi_write_file( const char *p, const mbedtls_mpi *X, int radix, FILE *fout )
```

##### 数值比较

mbedtls提供了两个大数比较的接口，分别是原值和绝对值：

```c
int mbedtls_mpi_cmp_abs( const mbedtls_mpi *X, const mbedtls_mpi *Y )
int mbedtls_mpi_cmp_mpi( const mbedtls_mpi *X, const mbedtls_mpi *Y )
```

由于大数结构体存放了数位n，因此首先比较两者n的大小，对绝对值的情况，n越大，值越大；如果n相同，则从高位向后循环。

##### 加减计算

大数加减提供四个主要函数：

```c
/*
 * Unsigned addition: X = |A| + |B|  (HAC 14.7)
 */
int mbedtls_mpi_add_abs( mbedtls_mpi *X, const mbedtls_mpi *A, const mbedtls_mpi *B )
/*
 * Unsigned subtraction: X = |A| - |B|  (HAC 14.9, 14.10)
 */
int mbedtls_mpi_sub_abs( mbedtls_mpi *X, const mbedtls_mpi *A, const mbedtls_mpi *B )
/*
 * Signed addition: X = A + B
 */
int mbedtls_mpi_add_mpi( mbedtls_mpi *X, const mbedtls_mpi *A, const mbedtls_mpi *B )
/*
 * Signed subtraction: X = A - B
 */
int mbedtls_mpi_sub_mpi( mbedtls_mpi *X, const mbedtls_mpi *A, const mbedtls_mpi *B )
```

首先是绝对值相加，然后是绝对值减法，有符号数加减法则可以是前两者的组合，例如正数加负数其实就是减法，负数加负数（或者负数减正数）则是绝对值加法取反。例如X=A+B的计算，
首先判断A和B的符号位乘积，如果为负数，
则比较A和B的绝对值，

- 绝对值A>B，则为绝对值A-B，符号位与A一致
- 绝对值A<B，则为绝对值B-A，符号位与A相反

如果为正数，则为绝对值A+B，符号位与A一致

##### 乘法运算

乘法提供两个接口

```c
/*
 * Baseline multiplication: X = A * B  (HAC 14.12)
 */
int mbedtls_mpi_mul_mpi( mbedtls_mpi *X, const mbedtls_mpi *A, const mbedtls_mpi *B )
/*
 * Baseline multiplication: X = A * b
 */
int mbedtls_mpi_mul_int( mbedtls_mpi *X, const mbedtls_mpi *A, mbedtls_mpi_uint b )
```

##### 大数除法

```c
/*
 * Division by mbedtls_mpi: A = Q * B + R  (HAC 14.20)
 */
int mbedtls_mpi_div_mpi( mbedtls_mpi *Q, mbedtls_mpi *R, const mbedtls_mpi *A,
                         const mbedtls_mpi *B )
```

这里Q=A/B，R=A mod B

##### 取模运算

其实就是上面的R=0

```c
/*
 * Modulo: R = A mod B
 */
int mbedtls_mpi_mod_mpi( mbedtls_mpi *R, const mbedtls_mpi *A, const mbedtls_mpi *B )
```

##### 指数运算

因为结果可能非常大，所以对结果取模N，即
$$
X = A^{E} mod N
$$

```c
/*
 * Sliding-window exponentiation: X = A^E mod N  (HAC 14.85)
 */
int mbedtls_mpi_exp_mod( mbedtls_mpi *X, const mbedtls_mpi *A,
                         const mbedtls_mpi *E, const mbedtls_mpi *N,
                         mbedtls_mpi *_RR )
```

##### 求取最大公约数

```c
/*
 * Greatest common divisor: G = gcd(A, B)  (HAC 14.54)
 */
int mbedtls_mpi_gcd( mbedtls_mpi *G, const mbedtls_mpi *A, const mbedtls_mpi *B )
```

##### 模逆运算

这里是求乘法逆元，即找到一个数X使得A和X的积关于模N与1同余，或者说
$$
A*X mod N = 1, X=A^{-1} mod N
$$

```c
/*
 * Modular inverse: X = A^-1 mod N  (HAC 14.61 / 14.64)
 */
int mbedtls_mpi_inv_mod( mbedtls_mpi *X, const mbedtls_mpi *A, const mbedtls_mpi *N )
```

### 平台安全架构

PSA，全称Platform Secure Architecture，平台安全架构，是ARM公司在物联网安全领域联合众多芯片厂商推出的一个重要的软件实现架构，主要内容包括：

 1. 威胁模型
 2. 安全分析
 3. 硬件和固件架构规范
 4. 开源固件参考实现
 5. 独立评估和认证计划--PSA Certified

TF-M是PSA的一种实现方式，主要基于ARM Cortex-M系列的芯片，除此之外还有TF-A等。[Mbedtls](https://tls.mbed.org/)也是ARM维护的基于ARM平台的开源嵌入式加密库，现已是[trust-frimware](https://www.trustedfirmware.org/)的一部分，除了基本的各类加解密算法及安全工具箱的实现，为方便嵌入式开发人员快速编写符合PSA规范的代码，mbedtls提供了各类安全工具的调用接口。源代码见mbdetls/library/psa_crypto.c文件。

#### Key management

PSA支持对称和非对称密钥，实现基础是Mbedtls提供的各类密码箱工具。在导入key的时候需要注意key的type（类型）、usage（用途）和alg（算法）等属性，决定了密文或者签名的长度和解密的数据是否正确。Key在使用时，程序会检查key的policy的合法性，因此key的管理非常重要。主要的key管理函数如下：

- 设置和检查key的用途
  
```c
static void psa_set_key_usage_flags(psa_key_attributes_t *attributes,
                                    psa_key_usage_t usage_flags);
static psa_key_usage_t psa_get_key_usage_flags(
    const psa_key_attributes_t *attributes);
```

- 设置和检查key的算法

```c
static void psa_set_key_algorithm(psa_key_attributes_t *attributes,
                                  psa_algorithm_t alg);
static psa_algorithm_t psa_get_key_algorithm(
    const psa_key_attributes_t *attributes);
```

- 设置和检查key的类型

```c
static void psa_set_key_type(psa_key_attributes_t *attributes,
                             psa_key_type_t type);
static psa_key_type_t psa_get_key_type(const psa_key_attributes_t *attributes);
```

- 获取key的属性

```c
psa_status_t psa_get_key_attributes(mbedtls_svc_key_id_t key,
                                    psa_key_attributes_t *attributes);
```

- 删除key

```c
psa_status_t psa_destroy_key(mbedtls_svc_key_id_t key);
```

- 导入导出key，注意`psa_export_public_key`函数只能是非对称密钥类型。

```c
psa_status_t psa_import_key(const psa_key_attributes_t *attributes,
                            const uint8_t *data,
                            size_t data_length,
                            mbedtls_svc_key_id_t *key);
psa_status_t psa_export_key(mbedtls_svc_key_id_t key,
                            uint8_t *data,
                            size_t data_size,
                            size_t *data_length);
psa_status_t psa_export_public_key(mbedtls_svc_key_id_t key,
                                   uint8_t *data,
                                   size_t data_size,
                                   size_t *data_length);
```

导入key后，常用来使用的参数是`mbedtls_svc_key_id_t *key`，即`psa_key_id_t *key`，在TF-M中又被命名为`psa_key_handle_t *key`，这是一个重要的参数，而`psa_key_attributes_t *attributes`则是指key的属性，内容如下：

```c
typedef struct psa_client_key_attributes_s psa_key_attributes_t;

/* This is the client view of the `key_attributes` structure. Only
 * fields which need to be set by the PSA crypto client are present.
 * The PSA crypto service will maintain a different version of the
 * data structure internally. */
struct psa_client_key_attributes_s
{
    uint16_t type;
    uint16_t bits;
    uint32_t lifetime;
    psa_key_id_t id;
    uint32_t usage;
    uint32_t alg;
};
```

#### Message digests

消息摘要，主要计算消息的单向散列函数值，计算hash的函数和Mbedtls类似，`psa_hash_operation_t *operation`需要先设置使用的算法，算法必须保证`PSA_ALG_IS_HASH(alg)`为真。

```c
psa_status_t psa_hash_setup(psa_hash_operation_t *operation,
                            psa_algorithm_t alg);
psa_status_t psa_hash_update(psa_hash_operation_t *operation,
                             const uint8_t *input,
                             size_t input_length);
psa_status_t psa_hash_finish(psa_hash_operation_t *operation,
                             uint8_t *hash,
                             size_t hash_size,
                             size_t *hash_length);
```

PSA提供了一个更简单的Hash计算接口，适合轻量化的计算，使用时需要注意确保hash的长度`hash_size`应当大于算法输出的长度（根据`PSA_HASH_LENGTH(alg)`计算）。

```c
psa_status_t psa_hash_compute(psa_algorithm_t alg,
                              const uint8_t *input,
                              size_t input_length,
                              uint8_t *hash,
                              size_t hash_size,
                              size_t *hash_length);
```

#### MAC

消息认证码（Message authentication codes），用于认证消息，计算的输入为消息和消息发送者持有的密钥（对称密钥），计算和验证消息认证码，计算MAC的函数与Mbedtls类似，可以多次调用update函数计算长消息的MAC，`psa_mac_operation_t *operation`包含计算消息验证码用的算法和key的信息。
此处key的usage属性必须为`PSA_KEY_USAGE_SIGN_MESSAGE | PSA_KEY_USAGE_VERIFY_MESSAGE`，算法必须保证`PSA_ALG_IS_MAC(alg)`为真。

```c
psa_status_t psa_mac_sign_setup(psa_mac_operation_t *operation,
                                mbedtls_svc_key_id_t key,
                                psa_algorithm_t alg);
psa_status_t psa_mac_verify_setup(psa_mac_operation_t *operation,
                                  mbedtls_svc_key_id_t key,
                                  psa_algorithm_t alg);
psa_status_t psa_mac_update(psa_mac_operation_t *operation,
                            const uint8_t *input,
                            size_t input_length);
psa_status_t psa_mac_sign_finish(psa_mac_operation_t *operation,
                                 uint8_t *mac,
                                 size_t mac_size,
                                 size_t *mac_length);
psa_status_t psa_mac_verify_finish(psa_mac_operation_t *operation,
                                   const uint8_t *mac,
                                   size_t mac_length);
```

同样PSA提供了简短消息情况下的接口：

```c
psa_status_t psa_mac_compute(mbedtls_svc_key_id_t key,
                             psa_algorithm_t alg,
                             const uint8_t *input,
                             size_t input_length,
                             uint8_t *mac,
                             size_t mac_size,
                             size_t *mac_length);
psa_status_t psa_mac_verify(mbedtls_svc_key_id_t key,
                            psa_algorithm_t alg,
                            const uint8_t *input,
                            size_t input_length,
                            const uint8_t *mac,
                            size_t mac_length);
```

#### Asymmetric cryptography

非对称加解密和签名认证的接口。

##### 非对称加解密

使用公钥加密，私钥解密，支持RSA、ECC等算法，key的usage为`PSA_KEY_USAGE_ENCRYPT | PSA_KEY_USAGE_DECRYPT`。参数salt（盐）在算法不需要的时候填`NULL`。

```c
psa_status_t psa_asymmetric_encrypt(mbedtls_svc_key_id_t key,
                                    psa_algorithm_t alg,
                                    const uint8_t *input,
                                    size_t input_length,
                                    const uint8_t *salt,
                                    size_t salt_length,
                                    uint8_t *output,
                                    size_t output_size,
                                    size_t *output_length);
psa_status_t psa_asymmetric_decrypt(mbedtls_svc_key_id_t key,
                                    psa_algorithm_t alg,
                                    const uint8_t *input,
                                    size_t input_length,
                                    const uint8_t *salt,
                                    size_t salt_length,
                                    uint8_t *output,
                                    size_t output_size,
                                    size_t *output_length);
```

##### 签名和验证

通常采用对消息的单向散列值使用私钥进行签名，而消息接收者验证签名的步骤如下：

- 对消息进行单向散列值计算
- 用公钥对签名进行解密
- 比对解密后的散列值和计算的散列值时否相同

上述过程是`psa_verify_hash`函数的执行过程。因此接口定义如下：

```c
psa_status_t psa_sign_hash(mbedtls_svc_key_id_t key,
                           psa_algorithm_t alg,
                           const uint8_t *hash,
                           size_t hash_length,
                           uint8_t *signature,
                           size_t signature_size,
                           size_t *signature_length);
psa_status_t psa_verify_hash(mbedtls_svc_key_id_t key,
                             psa_algorithm_t alg,
                             const uint8_t *hash,
                             size_t hash_length,
                             const uint8_t *signature,
                             size_t signature_length);
```

使用时key的usage为`PSA_KEY_USAGE_SIGN_HASH | PSA_KEY_USAGE_VERIFY_HASH`，key的type必须是非对称密钥类型，算法需要和计算单向散列值的函数（例如`psa_hash_compute`）使用的算法保持一致。

#### Symmetric cryptography

对称加解密的接口。key的usage为`PSA_KEY_USAGE_ENCRYPT | PSA_KEY_USAGE_DECRYPT`,算法必须满足`PSA_ALG_IS_CIPHER(alg)`为真。

```c
psa_status_t psa_cipher_encrypt(mbedtls_svc_key_id_t key,
                                psa_algorithm_t alg,
                                const uint8_t *input,
                                size_t input_length,
                                uint8_t *output,
                                size_t output_size,
                                size_t *output_length);
psa_status_t psa_cipher_decrypt(mbedtls_svc_key_id_t key,
                                psa_algorithm_t alg,
                                const uint8_t *input,
                                size_t input_length,
                                uint8_t *output,
                                size_t output_size,
                                size_t *output_length);
```

上面的接口使用的是随机初始化向量IV，通过下列一组函数可以使用特定的初始化向量：

```c
psa_cipher_encrypt_setup
psa_cipher_decrypt_setup
psa_cipher_generate_iv
psa_cipher_set_iv
psa_cipher_update
psa_cipher_finish
```

#### AEAD

Authenticated Encryption with Associated Data ([AEAD](https://blog.csdn.net/qq_35324057/article/details/106035773?utm_medium=distribute.pc_relevant.none-task-blog-baidujs_title-0&spm=1001.2101.3001.4242))，对称密码和消息认证码，是一种同时具备保密性，完整性和可认证性的加密形式。key的usage为`PSA_KEY_USAGE_DECRYPT | PSA_KEY_USAGE_ENCRYPT`，算法需要保证`PSA_ALG_IS_AEAD(alg)`为真。nonce可以是随机数或初始化向量，additional_data用来认证，不放在消息中加密。

```c
psa_status_t psa_aead_encrypt(mbedtls_svc_key_id_t key,
                              psa_algorithm_t alg,
                              const uint8_t *nonce,
                              size_t nonce_length,
                              const uint8_t *additional_data,
                              size_t additional_data_length,
                              const uint8_t *plaintext,
                              size_t plaintext_length,
                              uint8_t *ciphertext,
                              size_t ciphertext_size,
                              size_t *ciphertext_length);
psa_status_t psa_aead_decrypt(mbedtls_svc_key_id_t key,
                              psa_algorithm_t alg,
                              const uint8_t *nonce,
                              size_t nonce_length,
                              const uint8_t *additional_data,
                              size_t additional_data_length,
                              const uint8_t *ciphertext,
                              size_t ciphertext_length,
                              uint8_t *plaintext,
                              size_t plaintext_size,
                              size_t *plaintext_length);
```

这里支持的算法包括：

```c
#define PSA_ALG_CCM                             ((psa_algorithm_t)0x05500100)
#define PSA_ALG_GCM                             ((psa_algorithm_t)0x05500200)
#define PSA_ALG_CHACHA20_POLY1305               ((psa_algorithm_t)0x05100500)
```

#### psa示例代码

##### 签名认证

###### 导入ECC非对称密钥

```c
#define ECC_P256_PUBLIC_KEY_SIZE PSA_KEY_EXPORT_ECC_PUBLIC_KEY_MAX_SIZE(256)
static uint8_t attestation_public_key[ECC_P256_PUBLIC_KEY_SIZE]; /* 65bytes */
static size_t attestation_public_key_len = 0;
static psa_key_handle_t public_key_handle = 0;

static void load_key_pair()
{
    psa_key_attributes_t key_attributes = psa_key_attributes_init();
    psa_key_handle_t key_handle = 0;
    psa_ecc_family_t psa_curve;
    struct ecc_key_t attest_key = {0};
    uint8_t key_buf[3 * PSA_BITS_TO_BYTES(256)]; /* priv + x_coord + y_coord */
    enum tfm_plat_err_t plat_res;
    psa_key_usage_t usage = PSA_KEY_USAGE_SIGN_HASH | PSA_KEY_USAGE_VERIFY_HASH;

    tfm_plat_get_initial_attest_key(key_buf, sizeof(key_buf),
                                    &attest_key, &psa_curve);

    /* Setup the key policy for private key */
    psa_set_key_usage_flags(&key_attributes, usage);
    psa_set_key_algorithm(&key_attributes, PSA_ALG_ECDSA(PSA_ALG_SHA_256));
    psa_set_key_type(&key_attributes, PSA_KEY_TYPE_ECC_KEY_PAIR(psa_curve));

    /* Register private key to Crypto service */
    int32_t crypto_result = psa_import_key(&key_attributes,
                                           attest_key.priv_key,
                                           attest_key.priv_key_size,
                                           &key_handle);

    crypto_result = psa_export_public_key(key_handle, attestation_public_key,
                                          ECC_P256_PUBLIC_KEY_SIZE,
                                          &attestation_public_key_len);

    public_key_handle = key_handle;
}
```

###### 计算签名

```c
static int spm_server_sign_certificate(psa_msg_t *msg)
{
    uint8_t server_msg[16] = {0};
    uint8_t server_msg_hash[32] = {0};
    size_t hash_length = 0;

    uint8_t signature[64] = {0};
    size_t signature_length = 0;
    int i = 0;

    /* read data */
    uint8_t msg_length = psa_read(msg->handle, 0,
                                  server_msg, sizeof(server_msg));

    /* calculate hash */
    int32_t crypto_result = psa_hash_compute(PSA_ALG_SHA_256,
                                             server_msg,
                                             msg_length,
                                             server_msg_hash,
                                             sizeof(server_msg_hash),
                                             &hash_length);

    /* calculate signature */
    crypto_result = psa_asymmetric_sign(public_key_handle,
                                        PSA_ALG_ECDSA(PSA_ALG_SHA_256),
                                        server_msg_hash,
                                        hash_length,
                                        signature,
                                        sizeof(signature),
                                        &signature_length);

    psa_write(msg->handle, 0, signature, signature_length);
    psa_write(msg->handle, 1, &crypto_result, sizeof(crypto_result));

    return crypto_result;
}
```

###### 验证签名

```c
static int spm_client_verify_certificate(psa_msg_t *msg)
{
    uint8_t server_msg[16] = {0};
    uint8_t server_msg_hash[32] = {0};
    size_t hash_length = 0;
    int32_t verify_ret = 0;
    uint8_t signature[64] = {0};

    /* read data */
    uint8_t msg_length = psa_read(msg->handle, 0,
                                  server_msg, sizeof(server_msg));
    size_t signature_length = psa_read(msg->handle, 1,
                                       signature, sizeof(signature));

    /* calculate hash */
    int32_t crypto_result = psa_hash_compute(PSA_ALG_SHA_256,
                                             server_msg,
                                             msg_length,
                                             server_msg_hash,
                                             sizeof(server_msg_hash),
                                             &hash_length);

    /* verify signature */
    verify_ret = psa_asymmetric_verify(public_key_handle,
                                       PSA_ALG_ECDSA(PSA_ALG_SHA_256),
                                       server_msg_hash,
                                       hash_length,
                                       signature,
                                       signature_length);

    psa_write(msg->handle, 0, &verify_ret, sizeof(verify_ret));

    return 0;
}
```

##### AEAD加密解密

###### 导入对称密钥

```c
#define MAX_KEY_NUM 2
static psa_key_handle_t symmetric_key_handle[MAX_KEY_NUM] = {0};

static int32_t load_symmetric_key(int8_t key_index)
{
    if (key_index > MAX_KEY_NUM || key_index < 0)
    {
        return -1;
    }
    const uint8_t key_buf[] = "This is AES key";
    psa_key_attributes_t key_attributes = psa_key_attributes_init();
    psa_key_handle_t key_handle = 0;
    psa_key_usage_t usage = PSA_KEY_USAGE_ENCRYPT | PSA_KEY_USAGE_DECRYPT;

    psa_set_key_usage_flags(&key_attributes, usage);
    psa_set_key_type(&key_attributes, PSA_KEY_TYPE_AES);
    psa_set_key_algorithm(&key_attributes, PSA_ALG_CCM);

    int32_t crypto_result = psa_import_key(&key_attributes, key_buf,
                                           sizeof(key_buf), &key_handle);

    symmetric_key_handle[key_index] = key_handle;
}
```

###### 消息加密

```c
const size_t nonce_length = 12;
const uint8_t nonce[] = "01234567890";
const uint8_t associated_data[] = "This is associated data";

static int spm_client_encrypt_data(psa_msg_t *msg)
{
    int8_t key_index = -1;
    uint8_t plaintext[16] = {0};
    size_t plaintext_len = 0;
    uint8_t ciphertext[32] = {0};
    size_t ciphertext_len = 0;

    /* read data */
    int32_t result = psa_read(msg->handle, 0, &key_index, sizeof(key_index));
    plaintext_len = psa_read(msg->handle, 1, plaintext, sizeof(plaintext));

    /* encrypt data by key */
    result = psa_aead_encrypt(symmetric_key_handle[key_index],
                              PSA_ALG_CCM,
                              nonce,
                              nonce_length,
                              associated_data,
                              sizeof(associated_data),
                              plaintext,
                              plaintext_len,
                              ciphertext,
                              sizeof(ciphertext),
                              &ciphertext_len);

    psa_write(msg->handle, 0, ciphertext, ciphertext_len);
    psa_write(msg->handle, 1, &result, sizeof(result));

    return result;
}
```

###### 消息解密

```c
const size_t nonce_length = 12;
const uint8_t nonce[] = "01234567890";
const uint8_t associated_data[] = "This is associated data";

static int spm_client_decrypt_data(psa_msg_t *msg)
{
    int8_t key_index = -1;
    uint8_t ciphertext[32] = {0};
    size_t ciphertext_len = 0;
    uint8_t decrypttext[16] = {0};
    size_t decrypttext_len = 0;

    /* read data */
    int32_t result = psa_read(msg->handle, 0, &key_index, sizeof(key_index));
    ciphertext_len = psa_read(msg->handle, 1, ciphertext, sizeof(ciphertext));

    /* decrypt data by key */
    result = psa_aead_decrypt(symmetric_key_handle[key_index],
                              PSA_ALG_CCM,
                              nonce,
                              nonce_length,
                              associated_data,
                              sizeof(associated_data),
                              ciphertext,
                              ciphertext_len,
                              decrypttext,
                              sizeof(decrypttext),
                              &decrypttext_len);

    psa_write(msg->handle, 0, decrypttext, decrypttext_len);
    psa_write(msg->handle, 1, &result, sizeof(result));

    return result;
}
```

### 移植K210 RISC-V

亚博智能开发板，使用的编译工具和sdk为riscv64-unknown-elf-gcc和kendryte-standalone-sdk-develop，开发环境为windows。

![](/img/post_pics/index_img/mbed.png)

#### 移植过程简述

- 在sdk/lib下添加mbedtls的文件夹，删除无关的文件只保留.c和.h文件：
  - include
  - library

- 在sdk/lib/mbedtls下添加CMakeLists.txt

```bash
FILE(GLOB_RECURSE MBEDTLS_SRC
        "${CMAKE_CURRENT_LIST_DIR}/include/mbedtls/*.c"
        "${CMAKE_CURRENT_LIST_DIR}/include/mbedtls/*.h"
        "${CMAKE_CURRENT_LIST_DIR}/include/psa/*.c"
        "${CMAKE_CURRENT_LIST_DIR}/include/psa/*.h"
        "${CMAKE_CURRENT_LIST_DIR}/library/*.c"
        "${CMAKE_CURRENT_LIST_DIR}/library/*.h"
)

ADD_LIBRARY(mbedtls
        ${MBEDTLS_SRC}
)
TARGET_COMPILE_OPTIONS(mbedtls PRIVATE -O2)
```

- 在lib/CMakeLists.txt中添加

```bash
ADD_SUBDIRECTORY(mbedtls)
ADD_LIBRARY(kendryte
        ${LIB_SRC}
        ${MBEDTLS_SRC}
)

TARGET_LINK_LIBRARIES(kendryte
        PUBLIC
                nncase
                mbedtls
)
```

- 全局搜索替换
  - `#include "mbedtls/`替换为`#include "`
  - `#include <mbedtls/`替换为`#include <`
  - `#include "psa/`替换为`#include "`
  - `#include <psa/`替换为`#include <`

- 重命名mbedtls中的
  - `aes.h`重命名`mbed_aes.h`
  - `platform.h`重命名`mbed_platform.h`
  - `sha256.h`重命名`mbed_sha256.h`
并将mbedtls中依赖这些头文件的代码替换成新名称。
- 设置mbedtls/config.h中的几个宏，这是平台的限制：

```c
#define MBEDTLS_NO_PLATFORM_ENTROPY
// #define MBEDTLS_NET_C
// #define MBEDTLS_TIMING_C
```

- 由于库的冲突，删除x509_crl.c和x509_crt.c文件
  
#### 测试例程

使用IOT-Security仓库的AES代码，在sdk/src下添加test_mbedtls文件夹，下面新建main.c和pin_config.h文件，代码如下：
main.c

```c
#include <stdio.h>
#include <string.h>
#include "pin_config.h"

#include <stdint.h>
#include "cipher.h"
#include "mbed_platform.h"

/*
    ### padding with pkcs7 AES_128_CBC Encrypt
    ptx = "CBC has been the most commonly used mode of operation."
    key = 06a9214036b8a15b512e03d534120006
    iv  = 3dafba429d9eb430b422da802c9fac41
    ctx = 4DDF9012D7B3898745A1ED9860EB0FA2
          FD2BBD80D27190D72A2F240C8F372A27
          63746296DDC2BFCE7C252B6CD7DD4BA8
          577E096DBD8024C8B4C5A1160CA2D3F9
*/
char *ptx = "CBC has been the most commonly used mode of operation.";
uint8_t key[16] =
    {
        0x06, 0xa9, 0x21, 0x40, 0x36, 0xb8, 0xa1, 0x5b,
        0x51, 0x2e, 0x03, 0xd5, 0x34, 0x12, 0x00, 0x06};

uint8_t iv[16] =
    {
        0x3d, 0xaf, 0xba, 0x42, 0x9d, 0x9e, 0xb4, 0x30,
        0xb4, 0x22, 0xda, 0x80, 0x2c, 0x9f, 0xac, 0x41};

static void dump_buf(char *info, uint8_t *buf, uint32_t len)
{
    mbedtls_printf("%s", info);
    for(int i = 0; i < len; i++)
    {
        mbedtls_printf("%s%02X%s", i % 16 == 0 ? "\n\t" : " ",
                       buf[i], i == len - 1 ? "\n" : "");
    }
    mbedtls_printf("\n");
}

void cipher(int type)
{
    size_t len;
    int olen = 0;
    uint8_t buf[64];

    mbedtls_cipher_context_t ctx;
    const mbedtls_cipher_info_t *info;

    mbedtls_cipher_init(&ctx);
    info = mbedtls_cipher_info_from_type(type);

    mbedtls_cipher_setup(&ctx, info);
    mbedtls_printf("\ncipher info setup, name: %s, block size: %d\n",
                   mbedtls_cipher_get_name(&ctx),
                   mbedtls_cipher_get_block_size(&ctx));

    mbedtls_cipher_setkey(&ctx, key, sizeof(key) * 8, MBEDTLS_ENCRYPT);
    mbedtls_cipher_set_iv(&ctx, iv, sizeof(iv));
    mbedtls_cipher_update(&ctx, ptx, strlen(ptx), buf, &len);
    olen += len;

    mbedtls_cipher_finish(&ctx, buf + len, &len);
    olen += len;

    dump_buf("\ncipher aes encrypt:", buf, olen);

    mbedtls_cipher_free(&ctx);
}

void hardware_init(void)
{
    // fpioa映射
    fpioa_set_function(PIN_UART_USB_RX, FUNC_UART_USB_RX);
    fpioa_set_function(PIN_UART_USB_TX, FUNC_UART_USB_TX);
}

int main(void)
{
    hardware_init();
    // 初始化串口3，设置波特率为115200
    uart_init(UART_USB_NUM);
    uart_configure(UART_USB_NUM, 115200, UART_BITWIDTH_8BIT, UART_STOP_1, UART_PARITY_NONE);

    char *info = {"This is AES test on K210 RISC-V!\n"};
    uart_send_data(UART_USB_NUM, info, strlen(info));

    cipher(MBEDTLS_CIPHER_AES_128_CBC);
    cipher(MBEDTLS_CIPHER_AES_128_CTR);

    while(1)
    {
    }
    return 0;
}

```

pin_config.h

```c
#ifndef _PIN_CONFIG_H_
#define _PIN_CONFIG_H_
/*****************************HEAR-FILE************************************/
#include "fpioa.h"
#include "uart.h"

/*****************************HARDWARE-PIN*********************************/
// 硬件IO口，与原理图对应
#define PIN_UART_USB_RX       (4)
#define PIN_UART_USB_TX       (5)

/*****************************SOFTWARE-GPIO********************************/
// 软件GPIO口，与程序对应
#define UART_USB_NUM           UART_DEVICE_3

/*****************************FUNC-GPIO************************************/
// GPIO口的功能，绑定到硬件IO口
#define FUNC_UART_USB_RX       (FUNC_UART1_RX + UART_USB_NUM * 2)
#define FUNC_UART_USB_TX       (FUNC_UART1_TX + UART_USB_NUM * 2)

#endif /* _PIN_CONFIG_H_ */

```

在sdk下新建build文件夹并进入：

```bash
cmake .. -DPROJ=test_mbedtls -G "MinGW Makefiles"
```

将编译好的文件烧入开发板中
![](/img/post_pics/mbedtls/11.png)

上电运行：
![](/img/post_pics/mbedtls/12.png)
demo运行成功。
