---
title: i++和++i的区别
date: 2020-04-30 13:42:16
tags: 
    - C/C++
categories: 
    - Interview
---

在循环中不体现差异。

<!-- more -->

* `i++`，先执行其他操作，再i自加；  
* `i--`，先执行其他操作，再i自减；  
* `++i`，先i自加，再执行其他操作；  
* `--i`，先i自减，再执行其他操作；

测试代码：
```c
#include <stdio.h>
int main() {
    //直接计算++i和i++并无区别
    int num = 0;
    printf("num++ = %d,num = %d \n", num++, num);  //这种情况num++ = 0,num =1；
    num = 0;
    num++;
    printf("after num++, num= %d\n", num);  //这种情况num++ = 0；
    num = 0;
    ++num;
    printf("after ++num, num = %d\n", num);

    int i = 0, j = 0, k = 4, m = 4;
    int a[] = {0, 0, 0, 0, 0};
    a[i++] = 15;
    // a[0] = 15, i = 1
    // a[i] = 15; i++;
    printf("a[%d] = %d\t", i, a[i]);
    printf("a[%d] = %d\n", i - 1, a[i - 1]);

    int b[] = {0, 0, 0, 0, 0};
    b[++j] = 10;
    // b[1] = 10, j = 1
    // j++; b[j] = 10;
    printf("b[%d] = %d\t", i, b[i]);
    printf("b[%d] = %d\n", i - 1, b[i - 1]);

    int c[] = {0, 0, 0, 0, 0};
    c[k--] = 10;
    // c[4] = 10, k = 3
    // c[k] = 10, k--;
    printf("c[%d] = %d\t", k, c[k]);
    printf("c[%d] = %d\n", k + 1, c[k + 1]);

    int d[] = {0, 0, 0, 0, 0};
    d[--m] = 15;
    // d[3] = 15, m = 3
    // m--; d[m] = 15
    printf("d[%d] = %d\t", m, d[m]);
    printf("d[%d] = %d\n", m + 1, d[m + 1]);

    //对于for循环：写法上没有区别
    for (int i = 0; i < 5; i++) {
        printf("循环计数i = %d\n", i);
    }
    for (int i = 0; i < 5; ++i) {
        printf("循环计数i = %d\n", i);
    }
    //赋值
    int q = 0, p = 5;
    q = p++;
    // q = 5, p = 6;
    printf("q = %d\tp = %d\n", q, p);
    q = 0;
    p = 5;
    q = ++p;
    // q = 6, p = 6;
    printf("q = %d\tp = %d\n", q, p);

    return 0;
}
```
