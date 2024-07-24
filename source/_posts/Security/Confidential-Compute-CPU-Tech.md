---
layout: post
title: "机密计算: CPU 技术细节"
index_img: /img/post_pics/cc/trusted_compute_img.png
date: 2024-07-24 14:48:35
tags:
    - Cryptography
    - TEE
    - Confidential Compute
    - Security
categories: 
    - Security
---

# 可信链的建立

## 从可信根出发建立信任链

任何一步验签失败，都会使得系统无法启动。

![系统安全启动验签流程](/img/post_pics/cc/trusted_compute_0.png)

## 片外固件结构

加密+签名（安全服务器中完成）：

1. 在 Header 中填入加密参数（随机数），用于对固件 Body 进行加密；**随机数会作为对称加密的初始化向量**
2. 对 Header + Body 计算摘要得到哈希值，然后进行签名操作
3. 使用加密参数加密 Body
4. 拼接 Header + Enc_Body + Signature

![片外固件结构](/img/post_pics/cc/trusted_compute_1.png)

解密+验签（Boot阶段由安全管理芯片完成）：

1. 获得 Header 中的解密参数（随机数）
2. 使用对称密码对 Body 进行解密；
3. 对 Header + Dec_Body 计算摘要得到哈希值
4. 对得到的哈希值进行验签

## 可信度量

![静态度量](/img/post_pics/cc/trusted_compute_2.png)

- 逐级检查代码的完整性，度量的结果放在PCR(Platform Configurate Register)寄存器中；
- 核心度量根 CRTM，就是 ROM 代码；
- 服务器的 BIOS 都是 UEFI 模式；
- 将预期值和 PCR 寄存器的值进行比对，确认代码的完整性。

动态度量：

- TPCM 特性：TPM 只支持静态度量，国内 TPCM 标准可以支持动态度量
- 动态度量：安全处理器对运行中的关键程序和数据进行度量监控，包括：
  - 中断向量表
  - 系统调用表
  - 内核模块
- 度量报告：包含当前计算机的可信状态，同时对度量报告进行签名，无法伪造
- Hy独有：TDM 轻量级动态度量，满足不同用户需求

Windows 运行 certmgr 命令可以查看证书：

![查看证书](/img/post_pics/cc/trusted_compute_3.png)

# Mbedtls

[Support for SM4 (Chinese National Standard)](https://forums.mbed.com/t/support-for-sm4-chinese-national-standard/4119)
[Mbedtls SM4](https://github.com/Mbed-TLS/mbedtls/pull/1165)

```bash
$ git clone git@github.com:Mbed-TLS/mbedtls.git
$ cd mbedtls
$ git checkout main

$ git submodule update --init
# Or
$ rm -rf framework
$ git clone git@github.com:Mbed-TLS/mbedtls-framework.git
$ mv mbedtls-framework framework
$ cd framework
$ git checkout 750634d

# 编译
$ mkdir build && cd build
$ cmake ..
$ cmake --build . -j32
$ sudo make install

# 测试
$ cd tests
$ ./test_suite_aes.cbc
AES-128-CBC Encrypt NIST KAT #1 ................................... PASS
AES-128-CBC Encrypt NIST KAT #2 ................................... PASS
AES-128-CBC Encrypt NIST KAT #3 ................................... PASS
AES-128-CBC Encrypt NIST KAT #4 ................................... PASS
...
AES-256-CBC Decrypt NIST KAT #12 .................................. PASS

----------------------------------------------------------------------------

PASSED (72 / 72 tests (0 skipped))
```

## AES

```bash
crypt_and_hash <mode> <input filename> <output filename> <cipher> <mbedtls_md> <key>

   <mode>: 0 = encrypt, 1 = decrypt

  example: crypt_and_hash 0 file file.aes AES-128-CBC SHA1 hex:E76B2413958B00E193

Available ciphers:
  AES-128-ECB
  AES-192-ECB
  ...
  AES-256-XTS
  AES-128-GCM
  AES-192-GCM
  AES-256-GCM
  ...
  AES-256-KWP

Available message digests:
  SHA512
  SHA384
  SHA256
  SHA224
  SHA1
  RIPEMD160
  MD5
  SHA3-224
  SHA3-256
  SHA3-384
  SHA3-512
```

- 准备一个txt文本，内容为'Hello World'
- 使用 AES-256-GCM 加密，SHA256 验证完整性，密钥为 hex:7a30d862c42e74

```bash
$ echo 'Hello World' >> plain.txt
$ crypt_and_hash 0 plain.txt plain.txt.enc AES-256-GCM SHA256 hex:7a30d862c42e74
$ crypt_and_hash 1 plain.txt.enc plain.txt.dec AES-256-GCM SHA256 hex:7a30d862c42e74
$ cat plain.txt.dec
Hello World
```

## Hash

```bash
$ generic_sum SHA256 plain.txt
d2a84f4b8b650937ec8f73cd8be2c74add5a911ba64df27458ed8229da804a26  plain.txt

$ sha256sum plain.txt
d2a84f4b8b650937ec8f73cd8be2c74add5a911ba64df27458ed8229da804a26  plain.txt
```

## SM4 国密算法

```bash
$ git clone git@github.com:Low-power/mbedtls.git
$ cd mbedtls
$ mkdir build && cd build
```
