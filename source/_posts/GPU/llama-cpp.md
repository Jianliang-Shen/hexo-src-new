---
layout: post
title: Code Structure of Llama.cpp
date: 2024-08-04 11:45:51
index_img: /img/ai/llama-cpp.png
tags:
    - AI
    - GPU
    - LLM
    - LLAMA
categories: 
    - GPU
---

This blog is about the code structure analysis of `llama.cpp` project.

<!-- more -->

## Compilation

This project supports `Makefile` and `CMakeLists.txt`, while I don't understand `Makefile` well, I'll use the latter one to help understand the compilation of `Make`.

The most important example given by the LLM deployment framework is `llama-cli`. **The usage of `llama-cli` can be found in <https://github.com/ggerganov/llama.cpp/tree/master/examples/main>**.

It's CMake configuration shows in `examples/main/CMakeLists.txt`:

```cmake
set(TARGET llama-cli)
add_executable(${TARGET} main.cpp)
install(TARGETS ${TARGET} RUNTIME)
target_link_libraries(${TARGET} PRIVATE common llama ${CMAKE_THREAD_LIBS_INIT})
target_compile_features(${TARGET} PRIVATE cxx_std_11)
```

The source file to compile `llama-cli` is `examples/main/main.cpp`. It shows the execution file depends on `common` and `llama` libraries.

We could easily found the `llama` library in `src/CMakeLists.txt`, it is built by the most important backend source file `src/llama.cpp`, and the public header file `include/llama.h`. I may analyze the how the backend works later, while in this blog, I mainly focus the logic of `llama-cli` execution, and then I want to modify it. The library `common` is in the folder `common`, from the CMake code we can know it contains basic functions of the project.

## main.cpp

Let's dive into the `main.cpp` to find how it woks.

In the [chat_llama3.sh](https://www.jianliang-shen.cn/2024/07/23/GPU/Local-LLAMA-Deployment/)

```bash
FIRST_INSTRUCTION=$2
SYSTEM_PROMPT="You are a helpful assistant."

./llama-cli -m $1 --color -i \
-c 0 -t 6 --temp 0.2 --repeat_penalty 1.1 -ngl 40 \
-r '<|eot_id|>' \
--in-prefix '<|start_header_id|>user<|end_header_id|>\n\n' \
--in-suffix '<|eot_id|><|start_header_id|>assistant<|end_header_id|>\n\n' \
-p "<|start_header_id|>system<|end_header_id|>\n\n$SYSTEM_PROMPT\n\n<|eot_id|><|start_header_id|>user<|end_header_id|>\n\n$FIRST_INSTRUCTION\n\n<|eot_id|><|start_header_id|>assistant<|end_header_id|>\n\n"
```

TODO: Debug the llama.cpp.

## Install ChatTTs

I choose [ChatTTS](https://github.com/2noise/ChatTTS) to create the dialogue.

- [ChatTTS Homepage](https://chattts.com/zh)

```bash
git clone git@github.com:2noise/ChatTTS.git
cd ChatTTS

# I have already create the Conda environment
pip install -r requirements.txt -i https://pypi.tuna.tsinghua.edu.cn/simple
python examples/web/webui.py # Need download models manually: https://huggingface.co/2Noise/ChatTTS
```

A basic test:

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

In WSL, we need to play the .wav. Use VLC to easily open and play it:

```bash
# Ref: https://learn.microsoft.com/zh-cn/windows/wsl/tutorials/gui-apps
sudo apt install vlc -y
cvlc basic_output0.wav # will not create a window.
```

TODO: Learn advanced usage of this tool. Another way is alibaba cloud TTS.

- [ChatTTS进阶篇（适用于win/linux/wsl）](https://blog.csdn.net/imok1234567/article/details/140192207)

## Reference

- [Llama.cpp 上手实战指南](https://blog.yanghong.dev/llama-cpp-practice/)
- [深入解析Llama.cpp：揭秘高性能计算的秘密武器](https://cloud.baidu.com/article/3261084)
- [笔记：Llama.cpp 代码浅析（一）：并行机制与KVCache](https://zhuanlan.zhihu.com/p/670515231)
- [深入解读llama.cpp：本地CPU上的量化模型部署](https://developer.baidu.com/article/details/3185708)
- [llama.cpp](https://blog.csdn.net/qq_29788741/article/details/132352856)
- [基于llama.cpp的量化部署实战](https://www.53ai.com/news/qianyanjishu/976.html)
- [lama.cpp 源码解析](https://gitcode.csdn.net/662f72ce16ca5020cb5b6ef3.html)
- [LLaMa.cpp深度解析](https://hub.baai.ac.cn/view/28608)
- [llama.cpp 源码解析-- CUDA版本流程与逐算子详解](https://www.bilibili.com/video/BV1Ez4y1w7fc/?vd_source=5b91070ddac10c80113df5d29b7e2899)
- [研究完llama.cpp，我发现手机跑大模型竟这么简单](https://cloud.tencent.com/developer/article/2325942)
