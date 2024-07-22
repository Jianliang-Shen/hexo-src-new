---
title: C语言回调函数、计时方式及在Linux进度条程序
date: 2020-04-25 13:51:18
tags: 
    - C/C++
categories: 
    - 编程语言
---
简单介绍一些C语言的常用技巧。
<!-- more -->

- [回调函数的用法](#回调函数的用法)
- [计时方式](#计时方式)
- [写一个进度条程序](#写一个进度条程序)
## 回调函数的用法

定义一个普通函数和包括这个回调函数的函数
```c
int mission(long _num_);
int bar(int N, long num, int (*mission_call_back)(long));{
    ...
    mission_call_back(num);
    ...
}
```
在bar函数中，定义了一个回调函数，传递的是函数的指针，名称为misson_call_back，回调函数传递的值数据类型为long，在bar函数参数中应当包括回调函数用到的参数。
在调用时应当如下使用，直接传递函数名称不带参数：
```c
bar(N, 10000000L, mission);
```
## 计时方式
使用clock_t结构体和clock()函数，及CLOCKS_PER_SEC进行计算，精度为ms：
```c
    clock_t start, finish;
    double duration;

    start = clock();
    //需要计时的任务
    finish = clock();

    duration = (double)(finish - start) / CLOCKS_PER_SEC;
    printf("总共%0.2f秒\n", duration);
```
## 写一个进度条程序
```c
#include <stdio.h>
#include <time.h>
#include <stdlib.h>
#include <string.h>
int mission(long _num_)
{
    //long i = 100000000L;
    while (_num_--)
    {
    }
    return 0;
}
int bar(int N, long num, int (*mission_call_back)(long))
{
    int i = 1;
    char bar[102];
    memcpy(bar, "\0", 102);
    const char *flag = "-\\|/";

    if (N >= 100)
    {
        bar[0] = '=';
        while (i <= N)
        {

            printf("[%-100s] [%3d%%] [%c]\r", bar, i * 100 / N, flag[i % 4]);
            fflush(stdout);
            if (i * 100 / N - 1 > 0)
                bar[i * 100 / N - 1] = '=';
            if (i * 100 / N > 0)
                bar[i * 100 / N] = '>';
            i++;
            bar[i * 100 / N + 1] = '\0';
            mission_call_back(num);
        }
    }
    else if (N < 100)
    {
        i = 0;
        while (i <= N)
        {
            printf("[%-100s] [%3d%%] [%c]\r", bar, i * 100 / N, flag[i % 4]);
            fflush(stdout);
            bar[0] = '=';
            if (i == N)
            {
                break;
            }
            int block = 100 / N;
            for (int j = 1; j <= block; j++)
            {
                bar[i * block + j] = '=';
            }
            bar[(i + 1) * block + 1] = '>';
            bar[(i + 1) * block + 2] = '\0';
            i++;
            if (i == N)
            {
                for (int j = 1; j <= 98 - i * block; j++)
                {
                    bar[i * block + j] = '=';
                }
                bar[99] = '>';
                bar[100] = '\0';
            }

            mission_call_back(num);
        }
    }

    printf("\n");
}

int main(int arvc,char *argv[])
{
    printf("start run \"%s\"\n", __FILE__);

    clock_t start, finish;
    double duration;
    int N;
    N = atoi(argv[1]);
    //scanf("%d", &N);
    start = clock();
    bar(N, 10000000L, mission);
    finish = clock();
    duration = (double)(finish - start) / CLOCKS_PER_SEC;
    printf("总共%0.2f秒\n", duration);
    return 0;
}
```
运行结果：
```bash
➜  ~ ./pro_bar 130
start run "pro_bar.c"
[===================================================================================================>] [100%] [|]
总共1.70秒
```
总体思路，定义100基数作为基准，如果任务重复次数低于100，则每次增加的格子"="为100/N，反之为N/100次加1格。