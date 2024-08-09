---
layout: post
title: Llama Local Deployment
index_img: /img/ai/llama.jpeg
date: 2024-07-23 16:10:26
sticky: 110
tags:
    - AI
    - GPU
    - LLM
    - LLAMA
categories: 
    - AI
---

Chinese-LLaMA-Alpaca 是基于 Meta 发布的可商用大模型 Llama 开发，llama.cpp 是一个 C++ 语言部署 Llama 的框架。

<!-- more -->

- [llama.cpp](https://github.com/ggerganov/llama.cpp)

# llama 2

- [Chinese-LLaMA-Alpaca-2](https://github.com/ymcui/Chinese-LLaMA-Alpaca-2)

## 部署

- [长上下文版模型 Chinese-Alpaca-2-7B-64K](https://huggingface.co/hfl/chinese-alpaca-2-7b-64k-gguf/tree/main)
- [llama.cpp部署教程](https://github.com/ymcui/Chinese-LLaMA-Alpaca-2/wiki/llamacpp_zh)

注：FP16版本较慢，可以下`q8_0`或`Q6_K`，非常接近F16模型的效果。

```bash
git clone https://github.com/ggerganov/llama.cpp
cd llama.cpp

make GGML_CUDA=1
./chat.sh model/chinese-alpaca-2-7b-64k-ggml-model-f16.gguf '介绍一下北京'
```

- `chat.sh` 内容：

```bash
#!/bin/bash

# temporary script to chat with Chinese Alpaca-2 model
# usage: ./chat.sh alpaca2-ggml-model-path your-first-instruction

SYSTEM='You are a helpful assistant. 你是一个乐于助人的助手。'
FIRST_INSTRUCTION=$2

# 注，新版 llama.cpp 拆分了编译的 main 可执行文件
./llama-cli -m $1 \
--color -i -c 4096 -t 8 --temp 0.5 --top_k 40 --top_p 0.9 --repeat_penalty 1.1 \
--in-prefix-bos -ngl 40 --in-prefix ' [INST] ' --in-suffix ' [/INST]' -p \
"[INST] <<SYS>>
$SYSTEM
<</SYS>>

$FIRST_INSTRUCTION [/INST]"
```

{% note primary %}
GPU版本需要用`-ngl 40`参数指定（必须编译CUDA版本）。
{% endnote %}

![LLAMA](/img/ai/llama_gpu.png)

## server 设置

```bash
./llama-server -m model/chinese-alpaca-2-7b-64k-ggml-model-q8_0.gguf -c 4096 -ngl 40
```

创建一个client脚本：

```bash
# server_curl_example.sh

SYSTEM_PROMPT='You are a helpful assistant. 你是一个乐于助人的助手。'
# SYSTEM_PROMPT='You are a helpful assistant. 你是一个乐于助人的助手。请你提供专业、有逻辑、内 容真实、有价值的详细回复。' # Try this one, if you prefer longer response.
INSTRUCTION=$1
ALL_PROMPT="[INST] <<SYS>>\n$SYSTEM_PROMPT\n<</SYS>>\n\n$INSTRUCTION [/INST]"
CURL_DATA="{\"prompt\": \"$ALL_PROMPT\",\"n_predict\": 128}"

curl --request POST \
    --url http://localhost:8080/completion \
    --header "Content-Type: application/json" \
    --data "$CURL_DATA"

echo
```

```bash
./server_curl_example.sh '请列举5条文明乘车的建议'
```

![LLAMA Server](/img/ai/llama_server.png)

{% fold @运行后返回一个json条目 %}

```json
{
    "content": " 1. 保持秩序：尽量保持车内安静，尊重他人的权益。不要大声喧哗、大声打电话或使用电子设备，以 免干扰他人的休息和学习。\n\n2. 不吸烟：在乘车过程中，不吸烟或使用电子烟，以保持车内空气质量，并尊重他人的健康。\n\n3. 注意个人卫生：尽量避免在车内进食、嚼口香糖、吐痰等，保持车内的卫生和清洁。同时，保持个人卫生，如洗手、洗脸等，以减少细菌和病毒传播的风险。\n\n4. 遵守乘车规则：遵守乘车",
    "id_slot": 0,
    "stop": true,
    "model": "model/chinese-alpaca-2-7b-64k-ggml-model-q8_0.gguf",
    "tokens_predicted": 128,
    "tokens_evaluated": 43,
    "generation_settings": {
        "n_ctx": 4096,
        "n_predict": -1,
        "model": "model/chinese-alpaca-2-7b-64k-ggml-model-q8_0.gguf",
        "seed": 4294967295,
        "temperature": 0.800000011920929,
        "dynatemp_range": 0.0,
        "dynatemp_exponent": 1.0,
        "top_k": 40,
        "top_p": 0.949999988079071,
        "min_p": 0.05000000074505806,
        "tfs_z": 1.0,
        "typical_p": 1.0,
        "repeat_last_n": 64,
        "repeat_penalty": 1.0,
        "presence_penalty": 0.0,
        "frequency_penalty": 0.0,
        "penalty_prompt_tokens": [],
        "use_penalty_prompt_tokens": false,
        "mirostat": 0,
        "mirostat_tau": 5.0,
        "mirostat_eta": 0.10000000149011612,
        "penalize_nl": false,
        "stop": [],
        "n_keep": 0,
        "n_discard": 0,
        "ignore_eos": false,
        "stream": false,
        "logit_bias": [],
        "n_probs": 0,
        "min_keep": 0,
        "grammar": "",
        "samplers": [
            "top_k",
            "tfs_z",
            "typical_p",
            "top_p",
            "min_p",
            "temperature"
        ]
    },
    "prompt": "[INST] <<SYS>>\nYou are a helpful assistant. 你是一个乐于助人的助手。\n<</SYS>>\n\n请列举5条文明乘车的建议 [/INST]",
    "truncated": false,
    "stopped_eos": false,
    "stopped_word": false,
    "stopped_limit": true,
    "stopping_word": "",
    "tokens_cached": 170,
    "timings": {
        "prompt_n": 43,
        "prompt_ms": 244.353,
        "prompt_per_token_ms": 5.6826279069767445,
        "prompt_per_second": 175.97492152746233,
        "predicted_n": 128,
        "predicted_ms": 2295.145,
        "predicted_per_token_ms": 17.9308203125,
        "predicted_per_second": 55.7698968910461
    }
}
```

{% endfold %}

## Jetson Orin Nano

- [Your GPU Compute Capability](https://developer.nvidia.com/cuda-gpus)
- [chinese-alpaca-2-1.3b-gguf](https://huggingface.co/hfl/chinese-alpaca-2-1.3b-gguf) 下载 `q_2k` 版本，其他未测试

因为cuda版本低，编译llama.cpp前需要指定计算能力:

```bash
export CUDA_DOCKER_ARCH=compute_87
make GGML_CUDA=1 -j 4
```

其他配置同上

![Jetson 运行 Llama](/img/ai/llama_jetson.png)

{% note primary %}
第一次执行导入模型会很慢，之后会变快很多。图右边为`jtop`指令显示GPU占用情况。
{% endnote %}

测试长上下文 `7B q_2k`: [chinese-alpaca-2-7b-64k-gguf](https://huggingface.co/hfl/chinese-alpaca-2-7b-64k-gguf/tree/main), 显著提高了显存占用率。

![Jetson 运行 Llama 7B](/img/ai/llama_jetson2.png)

## ollama 部署

- [ollama部署体验Chinese-LLaMA-Alpaca-3大模型项目](https://blog.csdn.net/nlpstarter/article/details/138910697)
- [ollama下载地址](https://ollama.com/download)
- [ollama仓库](https://github.com/ollama/ollama?tab=readme-ov-file)

首次运行`ollama run llama3`，会联网加载模型：

```bash
$ ollama run llama3
pulling manifest
pulling 6a0746a1ec1a... 100% ▕███████████████████████████████████████▏  4.7 GB
pulling 4fa551d4f938... 100% ▕███████████████████████████████████████▏  12 KB
pulling 8ab4849b038c... 100% ▕███████████████████████████████████████▏  254 B
pulling 577073ffcc6c... 100% ▕███████████████████████████████████████▏  110 B
pulling 3f8eb4da87fa... 100% ▕███████████████████████████████████████▏  485 B
verifying sha256 digest
writing manifest
removing any unused layers
success
```

![Ollama](/img/ai/ollama.png)

实测是自动运行了GPU加速的。

体验本地中文LLama2大模型，参考[使用Ollama进行聊天](https://github.com/ymcui/Chinese-LLaMA-Alpaca-3/wiki/ollama_zh)，Modelfile如下：

```bash
FROM D:\work\LLAMA-Chinese-Model\chinese-alpaca-2-7b-64k-ggml-model-q8_0.gguf
TEMPLATE """{{ if .System }}<|start_header_id|>system<|end_header_id|>

{{ .System }}<|eot_id|>{{ end }}{{ if .Prompt }}<|start_header_id|>user<|end_header_id|>

{{ .Prompt }}<|eot_id|>{{ end }}<|start_header_id|>assistant<|end_header_id|>

{{ .Response }}<|eot_id|>"""
SYSTEM """You are a helpful assistant. 你是一个乐于助人的助手。请你提供专业、有逻辑、内 容真实、有价值的详细回复。"""
PARAMETER temperature 0.2
PARAMETER num_keep 24
PARAMETER stop <|start_header_id|>
PARAMETER stop <|end_header_id|>
PARAMETER stop <|eot_id|>
```

![Ollama](/img/ai/ollama_zh.png)

## 模型下载

### 模型选择指引

以下是中文LLaMA-2和Alpaca-2模型的对比以及建议使用场景。**如需聊天交互，请选择Alpaca而不是LLaMA。[^1]**

| 对比项                   |                                   中文LLaMA-2                                    |                                   中文Alpaca-2                                   |
| :----------------------- | :------------------------------------------------------------------------------: | :------------------------------------------------------------------------------: |
| 模型类型                 |                                   **基座模型**                                   |                          **指令/Chat模型（类ChatGPT）**                          |
| 已开源大小               |                                  1.3B、7B、13B                                   |                                  1.3B、7B、13B                                   |
| 训练类型                 |                                 Causal-LM (CLM)                                  |                                     指令精调                                     |
| 训练方式                 |                   7B、13B：LoRA + 全量emb/lm-head; 1.3B：全量                    |                   7B、13B：LoRA + 全量emb/lm-head; 1.3B：全量                    |
| 基于什么模型训练         |       [原版Llama-2](https://github.com/facebookresearch/llama)（非chat版）       |                                   中文LLaMA-2                                    |
| 训练语料                 |                           无标注通用语料（120G纯文本）                           |                            有标注指令数据（500万条）                             |
| 词表大小[^2]   |                                      55,296                                      |                                      55,296                                      |
| 上下文长度[^3] | 标准版：4K（12K-18K）; 长上下文版（PI）：16K（24K-32K）; 长上下文版（YaRN）：64K | 标准版：4K（12K-18K）; 长上下文版（PI）：16K（24K-32K）; 长上下文版（YaRN）：64K |
| 输入模板                 |                                      不需要                                      |                 需要套用特定模板[^4]，类似Llama-2-Chat                 |
| 适用场景                 |                        文本续写：给定上文，让模型生成下文                        |                        指令理解：问答、写作、聊天、交互等                        |
| 不适用场景               |                              指令理解 、多轮聊天等                               |                                文本无限制自由生成                                |
| 偏好对齐                 |                                        无                                        |                               RLHF版本（1.3B、7B）                               |

### 完整模型下载

以下是完整版模型，直接下载即可使用，无需其他合并步骤。推荐网络带宽充足的用户。

| 模型名称              |   类型   |  大小   |                                                                                                                                                           下载地址                                                                                                                                                           |                              GGUF                              |
| :-------------------- | :------: | :-----: | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------: | :------------------------------------------------------------: |
| Chinese-LLaMA-2-13B   | 基座模型 | 24.7 GB |   [[Baidu]](https://pan.baidu.com/s/1T3RqEUSmyg6ZuBwMhwSmoQ?pwd=e9qy) [[Google]](https://drive.google.com/drive/folders/1YNa5qJ0x59OEOI7tNODxea-1YvMPoH05?usp=share_link) ；[[🤗HF]](https://huggingface.co/hfl/chinese-llama-2-13b) ；[[🤖ModelScope]](https://modelscope.cn/models/ChineseAlpacaGroup/chinese-llama-2-13b)   |  [[🤗HF]](https://huggingface.co/hfl/chinese-llama-2-13b-gguf)  |
| Chinese-LLaMA-2-7B    | 基座模型 | 12.9 GB |    [[Baidu]](https://pan.baidu.com/s/1E5NI3nlQpx1j8z3eIzbIlg?pwd=n8k3) [[Google]](https://drive.google.com/drive/folders/18pp4I-mvQxRA7b8vF9gP-2cH_ocnXVKh?usp=share_link) ；[[🤗HF]](https://huggingface.co/hfl/chinese-llama-2-7b) ；[[🤖ModelScope]](https://modelscope.cn/models/ChineseAlpacaGroup/chinese-llama-2-7b)    |  [[🤗HF]](https://huggingface.co/hfl/chinese-llama-2-7b-gguf)   |
| Chinese-LLaMA-2-1.3B  | 基座模型 | 2.4 GB  |  [[Baidu]](https://pan.baidu.com/s/1hEuOCllnJJ5NMEZJf8OkRw?pwd=nwjg) [[Google]](https://drive.google.com/drive/folders/1Sd3PA_gs6JctXtBg5HwmHXh9GX93riMP?usp=share_link) ；[[🤗HF]](https://huggingface.co/hfl/chinese-llama-2-1.3b) ；[[🤖ModelScope]](https://modelscope.cn/models/ChineseAlpacaGroup/chinese-llama-2-1.3b)  | [[🤗HF]](https://huggingface.co/hfl/chinese-llama-2-1.3b-gguf)  |
| Chinese-Alpaca-2-13B  | 指令模型 | 24.7 GB |  [[Baidu]](https://pan.baidu.com/s/1MT_Zlap1OtdYMgoBNTS3dg?pwd=9xja) [[Google]](https://drive.google.com/drive/folders/1MTsKlzR61xmbTR4hBWzQas_MOpUZsogN?usp=share_link) ；[[🤗HF]](https://huggingface.co/hfl/chinese-alpaca-2-13b) ；[[🤖ModelScope]](https://modelscope.cn/models/ChineseAlpacaGroup/chinese-alpaca-2-13b)  | [[🤗HF]](https://huggingface.co/hfl/chinese-alpaca-2-13b-gguf)  |
| Chinese-Alpaca-2-7B   | 指令模型 | 12.9 GB |   [[Baidu]](https://pan.baidu.com/s/1wxx-CdgbMupXVRBcaN4Slw?pwd=kpn9) [[Google]](https://drive.google.com/drive/folders/1JsJDVs7tE2y31PBNleBlDPsB7S0ZrY8d?usp=share_link) ；[[🤗HF]](https://huggingface.co/hfl/chinese-alpaca-2-7b) ；[[🤖ModelScope]](https://modelscope.cn/models/ChineseAlpacaGroup/chinese-alpaca-2-7b)   |  [[🤗HF]](https://huggingface.co/hfl/chinese-alpaca-2-7b-gguf)  |
| Chinese-Alpaca-2-1.3B | 指令模型 | 2.4 GB  | [[Baidu]](https://pan.baidu.com/s/1PD7Ng-ltOIdUGHNorveptA?pwd=ar1p) [[Google]](https://drive.google.com/drive/folders/1h6qOy-Unvqs1_CJ8uPp0eKC61Gbbn8n7?usp=share_link) ；[[🤗HF]](https://huggingface.co/hfl/chinese-alpaca-2-1.3b) ；[[🤖ModelScope]](https://modelscope.cn/models/ChineseAlpacaGroup/chinese-alpaca-2-1.3b) | [[🤗HF]](https://huggingface.co/hfl/chinese-alpaca-2-1.3b-gguf) |

### 长上下文版模型

以下是长上下文版模型，**推荐以长文本为主的下游任务使用**，否则建议使用上述标准版。

| 模型名称                  |   类型   |  大小   |                                                                                                                                                              下载地址                                                                                                                                                               |                               GGUF                                |
| :------------------------ | :------: | :-----: | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------: | :---------------------------------------------------------------: |
| Chinese-LLaMA-2-7B-64K 🆕  | 基座模型 | 12.9 GB |   [[Baidu]](https://pan.baidu.com/s/1ShDQ2FG2QUJrvfnxCn4hwQ?pwd=xe5k) [[Google]][https://drive.google.com/drive/folders/17l9xJx55L2YNpqt7NiLVQzOZ6fV4rzJ-?usp=share_link]([🤗HF)](https://huggingface.co/hfl/chinese-llama-2-7b-64k) [[🤖ModelScope]](https://modelscope.cn/models/ChineseAlpacaGroup/chinese-llama-2-7b-64k)   |  [[🤗HF]](https://huggingface.co/hfl/chinese-llama-2-7b-64k-gguf)  |
| Chinese-Alpaca-2-7B-64K 🆕 | 指令模型 | 12.9 GB |  [[Baidu]](https://pan.baidu.com/s/1KBAr9PCGvX2oQkYfCuLEjw?pwd=sgp6) [[Google]][https://drive.google.com/drive/folders/13G_d5xcDnhtaMOaulj1BFiZbVoVwJ-Cu?usp=share_link]([🤗HF)](https://huggingface.co/hfl/chinese-alpaca-2-7b-64k) [[🤖ModelScope]](https://modelscope.cn/models/ChineseAlpacaGroup/chinese-alpaca-2-7b-64k)  | [[🤗HF]](https://huggingface.co/hfl/chinese-alpaca-2-7b-64k-gguf)  |
| Chinese-LLaMA-2-13B-16K   | 基座模型 | 24.7 GB |  [[Baidu]](https://pan.baidu.com/s/1XWrh3Ru9x4UI4-XmocVT2w?pwd=f7ik) [[Google]][https://drive.google.com/drive/folders/1nii6lF0DgB1u81CnsE4cCK2jD5oq_OW-?usp=share_link]([🤗HF)](https://huggingface.co/hfl/chinese-llama-2-13b-16k) [[🤖ModelScope]](https://modelscope.cn/models/ChineseAlpacaGroup/chinese-llama-2-13b-16k)  | [[🤗HF]](https://huggingface.co/hfl/chinese-llama-2-13b-16k-gguf)  |
| Chinese-LLaMA-2-7B-16K    | 基座模型 | 12.9 GB |   [[Baidu]](https://pan.baidu.com/s/1ZH7T7KU_up61ugarSIXw2g?pwd=pquq) [[Google]][https://drive.google.com/drive/folders/1Zc6jI5bl3myQbQsY79dWJJ8mP_fyf3iF?usp=share_link]([🤗HF)](https://huggingface.co/hfl/chinese-llama-2-7b-16k) [[🤖ModelScope]](https://modelscope.cn/models/ChineseAlpacaGroup/chinese-llama-2-7b-16k)   |  [[🤗HF]](https://huggingface.co/hfl/chinese-llama-2-7b-16k-gguf)  |
| Chinese-Alpaca-2-13B-16K  | 指令模型 | 24.7 GB | [[Baidu]](https://pan.baidu.com/s/1gIzRM1eg-Xx1xV-3nXW27A?pwd=qi7c) [[Google]][https://drive.google.com/drive/folders/1mOkYQCvEqtGoZ9DaIpYFweSkSia2Q0vl?usp=share_link]([🤗HF)](https://huggingface.co/hfl/chinese-alpaca-2-13b-16k) [[🤖ModelScope]](https://modelscope.cn/models/ChineseAlpacaGroup/chinese-alpaca-2-13b-16k) | [[🤗HF]](https://huggingface.co/hfl/chinese-alpaca-2-13b-16k-gguf) |
| Chinese-Alpaca-2-7B-16K   | 指令模型 | 12.9 GB |  [[Baidu]](https://pan.baidu.com/s/1Qk3U1LyvMb1RSr5AbiatPw?pwd=bfis) [[Google]][https://drive.google.com/drive/folders/1KBRSd2xAhiVQmamfA5wpm5ovYFRKuMdr?usp=share_link]([🤗HF)](https://huggingface.co/hfl/chinese-alpaca-2-7b-16k) [[🤖ModelScope]](https://modelscope.cn/models/ChineseAlpacaGroup/chinese-alpaca-2-7b-16k)  | [[🤗HF]](https://huggingface.co/hfl/chinese-alpaca-2-7b-16k-gguf)  |

# llama 3

- [Chinese-LLaMA-Alpaca-3](https://github.com/ymcui/Chinese-LLaMA-Alpaca-3)
- [llama-3-chinese-8b-instruct-v3-gguf](https://huggingface.co/hfl/llama-3-chinese-8b-instruct-v3-gguf/tree/main)

## 模型下载

| 模型名称                                          |                                                                                                                                       完整版                                                                                                                                        |                                                                                                                                               LoRA版                                                                                                                                               |                                                                                           GGUF版                                                                                            |
| :------------------------------------------------ | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------: | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------: | :-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------: |
| **Llama-3-Chinese-8B-Instruct-v3**(指令模型) | [[🤗HF]](https://huggingface.co/hfl/llama-3-chinese-8b-instruct-v3) [[🤖ModelScope]][https://modelscope.cn/models/ChineseAlpacaGroup/llama-3-chinese-8b-instruct-v3]([🟣wisemodel)](https://wisemodel.cn/models/ChineseAlpacaGroup/llama-3-chinese-8b-instruct-v3) |                                                                                                                                                N/A                                                                                                                                                 | [[🤗HF]](https://huggingface.co/hfl/llama-3-chinese-8b-instruct-v3-gguf) [[🤖ModelScope]](https://modelscope.cn/models/ChineseAlpacaGroup/llama-3-chinese-8b-instruct-v3-gguf) |
| **Llama-3-Chinese-8B-Instruct-v2**(指令模型) | [[🤗HF]](https://huggingface.co/hfl/llama-3-chinese-8b-instruct-v2) [[🤖ModelScope]][https://modelscope.cn/models/ChineseAlpacaGroup/llama-3-chinese-8b-instruct-v2]([🟣wisemodel)](https://wisemodel.cn/models/ChineseAlpacaGroup/llama-3-chinese-8b-instruct-v2) | [[🤗HF]](https://huggingface.co/hfl/llama-3-chinese-8b-instruct-v2-lora) [[🤖ModelScope]][https://modelscope.cn/models/ChineseAlpacaGroup/llama-3-chinese-8b-instruct-v2-lora]([🟣wisemodel)](https://wisemodel.cn/models/ChineseAlpacaGroup/llama-3-chinese-8b-instruct-v2-lora) | [[🤗HF]](https://huggingface.co/hfl/llama-3-chinese-8b-instruct-v2-gguf) [[🤖ModelScope]](https://modelscope.cn/models/ChineseAlpacaGroup/llama-3-chinese-8b-instruct-v2-gguf) |
| **Llama-3-Chinese-8B-Instruct**(指令模型)    |     [[🤗HF]](https://huggingface.co/hfl/llama-3-chinese-8b-instruct) [[🤖ModelScope]][https://modelscope.cn/models/ChineseAlpacaGroup/llama-3-chinese-8b-instruct]([🟣wisemodel)](https://wisemodel.cn/models/ChineseAlpacaGroup/llama-3-chinese-8b-instruct)      |     [[🤗HF]](https://huggingface.co/hfl/llama-3-chinese-8b-instruct-lora) [[🤖ModelScope]][https://modelscope.cn/models/ChineseAlpacaGroup/llama-3-chinese-8b-instruct-lora]([🟣wisemodel)](https://wisemodel.cn/models/ChineseAlpacaGroup/llama-3-chinese-8b-instruct-lora)      |    [[🤗HF]](https://huggingface.co/hfl/llama-3-chinese-8b-instruct-gguf) [[🤖ModelScope]](https://modelscope.cn/models/ChineseAlpacaGroup/llama-3-chinese-8b-instruct-gguf)    |
| **Llama-3-Chinese-8B**(基座模型)             |                   [[🤗HF]](https://huggingface.co/hfl/llama-3-chinese-8b) [[🤖ModelScope]][https://modelscope.cn/models/ChineseAlpacaGroup/llama-3-chinese-8b]([🟣wisemodel)](https://wisemodel.cn/models/ChineseAlpacaGroup/llama-3-chinese-8b)                   |                   [[🤗HF]](https://huggingface.co/hfl/llama-3-chinese-8b-lora) [[🤖ModelScope]][https://modelscope.cn/models/ChineseAlpacaGroup/llama-3-chinese-8b-lora]([🟣wisemodel)](https://wisemodel.cn/models/ChineseAlpacaGroup/llama-3-chinese-8b-lora)                   |             [[🤗HF]](https://huggingface.co/hfl/llama-3-chinese-8b-gguf) [[🤖ModelScope]](https://modelscope.cn/models/ChineseAlpacaGroup/llama-3-chinese-8b-gguf)             |

## 部署

- [llama.cpp 部署](https://github.com/ymcui/Chinese-LLaMA-Alpaca-3/wiki/llamacpp_zh)
- `chat_llama3.sh`:

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

- 运行：

```bash
./chat_llama3.sh model/llama-3-chinese-8b-instruct-v3-ggml-model-q8_0.gguf 介绍一下北京
```

## ollama 部署

- `Modelfile_llama3`:

```bash
FROM D:\work\LLAMA-Chinese-Model\llama-3-chinese-8b-instruct-v3-ggml-model-q8_0.gguf
TEMPLATE """{{ if .System }}<|start_header_id|>system<|end_header_id|>

{{ .System }}<|eot_id|>{{ end }}{{ if .Prompt }}<|start_header_id|>user<|end_header_id|>

{{ .Prompt }}<|eot_id|>{{ end }}<|start_header_id|>assistant<|end_header_id|>

{{ .Response }}<|eot_id|>"""
SYSTEM """You are a helpful assistant. 你是一个乐于助人的助手。"""
PARAMETER temperature 0.2
PARAMETER num_keep 24
PARAMETER stop <|start_header_id|>
PARAMETER stop <|end_header_id|>
PARAMETER stop <|eot_id|>
```

- 运行

```bash
ollama create llama3-zh-inst -f Modelfile_llama3
ollama run llama3-zh-inst
```

# 指令微调

- 模型下载：[llama-3-chinese-8b-instruct-v3](https://huggingface.co/hfl/llama-3-chinese-8b-instruct-v3/tree/main)
- 数据：[ruozhiba_gpt4](https://huggingface.co/datasets/hfl/ruozhiba_gpt4/tree/main)
- 训练参考：[指令精调脚本](https://github.com/ymcui/Chinese-LLaMA-Alpaca-3/wiki/sft_scripts_zh)

训练示意图：

![Llama train](/img/ai/llama_train_inst.png)

报错：显存不足（TODO: 待解决）

- [CUDA报错:Out of Memory](https://blog.csdn.net/Coldlebron/article/details/127575370)

# 参考

[^1]: 不建议单独使用1.3B模型，而是通过投机采样搭配更大的模型（7B、13B）使用。
[^2]: 本项目一代模型和二代模型的词表不同，请勿混用。二代LLaMA和Alpaca的词表相同。
[^3]: 括号内表示基于NTK上下文扩展支持的最大长度。
[^4]: Alpaca-2采用了Llama-2-chat系列模板（格式相同，提示语不同），而不是一代Alpaca的模板，请勿混用。
