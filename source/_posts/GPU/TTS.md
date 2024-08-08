---
layout: post
title: ChatTTS 语音合成试用
index_img: /img/ai/tts/index.png
date: 2024-08-07 23:30:47
tags:
    - GPU
    - AI
    - TTS
categories: 
    - GPU
---

[ChatTTS](https://github.com/2noise/ChatTTS) 是一个用于对话场景的文本转语音的强大工具

<!-- more -->

- [ChatTTS 主页](https://chattts.com/zh)
- [ChatTTS增强版V3【已开源】，长文本修复，中英混读，导入音色，批量SRT、TXT](https://blog.csdn.net/weixin_43935971/article/details/139877978)
  - [ChatTTS-Enhanced](https://github.com/CCmahua/ChatTTS-Enhanced)
  - [ChatTTS-Forge](https://github.com/lenML/ChatTTS-Forge)
  - [离线整合包 ChatTTS_colab](https://github.com/6drf21e/ChatTTS_colab)

```bash
git clone git@github.com:2noise/ChatTTS.git
cd ChatTTS

# 使用 https://www.jianliang-shen.cn/2024/06/24/GPU/Computing-Startup/#%E5%AE%89%E8%A3%85-pytorch-cuda-%E7%89%88%E6%9C%AC 已经创建好的 conda-gpu 环境
pip install -r requirements.txt -i https://pypi.tuna.tsinghua.edu.cn/simple

# 以下是为适配 GPU，可以快速生成
# 注意：安装编译十分缓慢
pip install git+https://github.com/NVIDIA/TransformerEngine.git@stable -i https://pypi.tuna.tsinghua.edu.cn/simple
pip install flash-attn --no-build-isolation -i https://pypi.tuna.tsinghua.edu.cn/simple

# 需要手动一一下载模型文件: https://huggingface.co/2Noise/ChatTTS
# 启动一个 WebUI
python examples/web/webui.py
```

运行示意图：

{% gi 2 1-1 %}
    ![UI](/img/ai/tts/1.png)
    ![Backend](/img/ai/tts/2.png)
{% endgi %}

示例结果：

<!-- 参考 https://dplayer.diygod.dev/zh/ -->
{%
    dplayer
    "url=/img/ai/tts/audio.mp3" //设置视频目录，这里我放在了网站根目录下面，也就是public目录下面
    "pic=/img/ai/tts/index.png" //设置封面图，同样是放在根目录下面
    "loop=yes"                  //循环播放
    "theme=#FADFA3"             //主题
    "autoplay=true"             //自动播放
    "screenshot=true"           //允许截屏
    "hotkey=true"               //允许hotKey，比如点击空格暂停视频等操作
    "preload=auto"              //预加载：auto
    "volume=0.9"                //初始音量
    "playbackSpeed=1"           //播放速度1倍速，可以选择1.5,2等
    "lang=zh-cn"                //语言
    "mutex=true"                //播放互斥，就比如其他视频播放就会导致这个视频自动暂停
%}

# Demo

一个基础的测试：

```py
import ChatTTS
import torch
import torchaudio

chat = ChatTTS.Chat()
chat.load(compile=False) # Set to True for better performance

texts = ["你好，欢迎来到上海", "你好，我是你的私人助理"]

wavs = chat.infer(texts)

for i in range(len(wavs)):
    torchaudio.save(f"basic_output{i}.wav", torch.from_numpy(wavs[i]).unsqueeze(0), 24000)
```

在 WSL 中，我们需要播放这个.wav，使用 VLC 来简单打开并播放它。参考 [在适用于 Linux 的 Windows 子系统上运行 Linux GUI 应用](https://learn.microsoft.com/zh-cn/windows/wsl/tutorials/gui-apps)

```bash
# Ref: https://learn.microsoft.com/zh-cn/windows/wsl/tutorials/gui-apps
sudo apt install vlc -y
cvlc basic_output0.wav # will not create a window.
```
