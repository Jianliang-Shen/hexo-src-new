---
layout: post
title: AMD CC
date: 2024-08-01 20:29:31
tags: 
    - TEE
    - Confidential Compute
    - Security
categories: 
    - Security
---

转载：[AMD机密计算解决方案分析](https://blog.csdn.net/huang987246510/article/details/135707747)

[水印去除](https://www.mindonmap.com/zh/watermark-remover-online/#)
## 数据结构

AMD SEV机密计算场景下，Hypervisor的职能稍有改变，可以概括为以下三点：

- 管理主机计算资源管理
- 代理可信组件间通信
- 准备机密计算环境

对于第一点，QEMU/KVM复用已有的逻辑，无需修改，对于二、三点，QEMU/KVM主要基于SEV API实现。具体实现时，KVM需要封装SEV API并对QEMU暴露IOCTL命令字，QEMU按照SEV API定义的流程实现对机密虚机的生命周期管理逻辑。

## KVM

分析KVM对QEMU提供的IOCTL命令字涉及的数据结构

![](/img/cc/amd/amd-cc-1.png)
