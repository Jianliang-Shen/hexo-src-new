---
layout: post
title: Stable Diffusion 安裝配置
date: 2024-07-23 15:03:51
index_img: /img/stable/index.png
tags:
    - AI
    - GPU
    - LLM
categories: 
    - AI
---

[Stable Diffusion](https://github.com/Stability-AI/stablediffusion)
[Stable Diffusion WebUI](https://github.com/AUTOMATIC1111/stable-diffusion-webui)

<!-- more -->

## Installation on Windows 10/11

1. Download sd.webui.zip from [v1.0.0-pre](https://github.com/AUTOMATIC1111/stable-diffusion-webui/releases/tag/v1.0.0-pre) and extract its contents.
2. Run update.bat.
3. Run run.bat.

For more details see [Install-and-Run-on-NVidia-GPUs](https://github.com/AUTOMATIC1111/stable-diffusion-webui/wiki/Install-and-Run-on-NVidia-GPUs)

## Model Download

要下载的是：`v1-5-pruned-emaonly.safetensors`，国内可能下载不下来，可以到[huggingface](https://huggingface.co/models?sort=trending&search=pruned-emaonly)下载。

其他模型：[medium model](https://huggingface.co/stabilityai/stable-diffusion-3-medium/tree/main)

## 插件

将[简体中文翻译扩展](https://github.com/dtlnor/stable-diffusion-webui-localization-zh_CN)一个个安装后，得到下面的界面：

![Lobe主题的界面](/img/stable/lobe.png)

>注：有时候无法从github拉取插件，或者搜索安装，考虑手动安装。

已安装插件一览：

![插件一览](/img/stable/lobe2.png)

## issues

xformer问题：[No module 'xformers'. Proceeding without it. #5303](https://github.com/AUTOMATIC1111/stable-diffusion-webui/discussions/5303)

修改`webui-user.bat`：

```bat
set COMMANDLINE_ARGS= --xformers --opt-sdp-no-mem-attention --enable-insecure-extension-access
```
