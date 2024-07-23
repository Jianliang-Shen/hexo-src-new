---
layout: post
title: Llama 本地化部署
index_img: /img/post_pics/ai/index_llama.png
date: 2024-07-23 16:10:26
tags:
    - AI
    - GPU
    - LLM
    - LLAMA
categories: 
    - GPU
---

Chinese-LLaMA-Alpaca-2是基于Meta发布的可商用大模型Llama-2开发，是中文LLaMA&Alpaca大模型的第二期项目，开源了中文LLaMA-2基座模型和Alpaca-2指令精调大模型。

<!-- more -->

这些模型在原版Llama-2的基础上扩充并优化了中文词表，使用了大规模中文数据进行增量预训练，进一步提升了中文基础语义和指令理解能力，相比一代相关模型获得了显著性能提升。相关模型支持FlashAttention-2训练。标准版模型支持4K上下文长度，长上下文版模型支持16K、64k上下文长度。RLHF系列模型为标准版模型基础上进行人类偏好对齐精调，相比标准版模型在正确价值观体现方面获得了显著性能提升。

[Chinese-LLaMA-Alpaca-2](https://github.com/ymcui/Chinese-LLaMA-Alpaca-2)
[Chinese-LLaMA-Alpaca](https://github.com/ymcui/Chinese-LLaMA-Alpaca)
[LLM inference in C/C++](https://github.com/ggerganov/llama.cpp)

# 目录

- [目录](#目录)
- [Llama 2](#llama-2)
  - [配置过程](#配置过程)
  - [模型选择指引](#模型选择指引)
  - [完整模型下载](#完整模型下载)

# Llama 2

## 配置过程

[chinese-alpaca-2-1.3b-gguf](https://huggingface.co/hfl/chinese-alpaca-2-1.3b-gguf/tree/main)
[llama.cpp部署教程](https://github.com/ymcui/Chinese-LLaMA-Alpaca-2/wiki/llamacpp_zh)

```bash
git clone https://github.com/ggerganov/llama.cpp
cd llama.cpp

make GGML_CUDA=1
./chat.sh model/ggml-model-f16.gguf '背一下静夜思'
```

`chat.sh` 内容：

```bash
#!/bin/bash

# temporary script to chat with Chinese Alpaca-2 model
# usage: ./chat.sh alpaca2-ggml-model-path your-first-instruction

SYSTEM='You are a helpful assistant. 你是一个乐于助人的助手。'
FIRST_INSTRUCTION=$2

# 注，新版 llama.cpp 拆分了编译的 main 可执行文件
./llama-cli -m $1 \
--color -i -c 4096 -t 8 --temp 0.5 --top_k 40 --top_p 0.9 --repeat_penalty 1.1 \
--in-prefix-bos --in-prefix ' [INST] ' --in-suffix ' [/INST]' -p \
"[INST] <<SYS>>
$SYSTEM
<</SYS>>

$FIRST_INSTRUCTION [/INST]"
```

![LLAMA](/img/post_pics/ai/llama.png)

{% fold @执行log %}

```txt
./chat.sh model/ggml-model-f16.gguf '背一下静夜思'
Log start
main: build = 3446 (938943cd)
main: built with cc (Ubuntu 9.4.0-1ubuntu1~20.04.3) 9.4.0 for x86_64-linux-gnu
main: seed  = 1721743303
llama_model_loader: loaded meta data with 21 key-value pairs and 39 tensors from model/ggml-model-f16.gguf (version GGUF V3 (latest))
llama_model_loader: Dumping metadata keys/values. Note: KV overrides do not apply in this output.
llama_model_loader: - kv   0:                       general.architecture str              = llama
llama_model_loader: - kv   1:                               general.name str              = LLaMA v2
llama_model_loader: - kv   2:                       llama.context_length u32              = 4096
llama_model_loader: - kv   3:                     llama.embedding_length u32              = 4096
llama_model_loader: - kv   4:                          llama.block_count u32              = 4
llama_model_loader: - kv   5:                  llama.feed_forward_length u32              = 11008
llama_model_loader: - kv   6:                 llama.rope.dimension_count u32              = 128
llama_model_loader: - kv   7:                 llama.attention.head_count u32              = 32
llama_model_loader: - kv   8:              llama.attention.head_count_kv u32              = 32
llama_model_loader: - kv   9:     llama.attention.layer_norm_rms_epsilon f32              = 0.000010
llama_model_loader: - kv  10:                       llama.rope.freq_base f32              = 10000.000000
llama_model_loader: - kv  11:                          general.file_type u32              = 1
llama_model_loader: - kv  12:                       tokenizer.ggml.model str              = llama
llama_model_loader: - kv  13:                      tokenizer.ggml.tokens arr[str,55296]   = ["<unk>", "<s>", "</s>", "<0x00>", "<...
llama_model_loader: - kv  14:                      tokenizer.ggml.scores arr[f32,55296]   = [0.000000, 0.000000, 0.000000, 0.0000...
llama_model_loader: - kv  15:                  tokenizer.ggml.token_type arr[i32,55296]   = [2, 3, 3, 6, 6, 6, 6, 6, 6, 6, 6, 6, ...
llama_model_loader: - kv  16:                tokenizer.ggml.bos_token_id u32              = 1
llama_model_loader: - kv  17:                tokenizer.ggml.eos_token_id u32              = 2
llama_model_loader: - kv  18:            tokenizer.ggml.padding_token_id u32              = 0
llama_model_loader: - kv  19:               tokenizer.ggml.add_bos_token bool             = true
llama_model_loader: - kv  20:               tokenizer.ggml.add_eos_token bool             = false
llama_model_loader: - type  f32:    9 tensors
llama_model_loader: - type  f16:   30 tensors
llm_load_vocab: special tokens cache size = 3
llm_load_vocab: token to piece cache size = 0.2971 MB
llm_load_print_meta: format           = GGUF V3 (latest)
llm_load_print_meta: arch             = llama
llm_load_print_meta: vocab type       = SPM
llm_load_print_meta: n_vocab          = 55296
llm_load_print_meta: n_merges         = 0
llm_load_print_meta: vocab_only       = 0
llm_load_print_meta: n_ctx_train      = 4096
llm_load_print_meta: n_embd           = 4096
llm_load_print_meta: n_layer          = 4
llm_load_print_meta: n_head           = 32
llm_load_print_meta: n_head_kv        = 32
llm_load_print_meta: n_rot            = 128
llm_load_print_meta: n_swa            = 0
llm_load_print_meta: n_embd_head_k    = 128
llm_load_print_meta: n_embd_head_v    = 128
llm_load_print_meta: n_gqa            = 1
llm_load_print_meta: n_embd_k_gqa     = 4096
llm_load_print_meta: n_embd_v_gqa     = 4096
llm_load_print_meta: f_norm_eps       = 0.0e+00
llm_load_print_meta: f_norm_rms_eps   = 1.0e-05
llm_load_print_meta: f_clamp_kqv      = 0.0e+00
llm_load_print_meta: f_max_alibi_bias = 0.0e+00
llm_load_print_meta: f_logit_scale    = 0.0e+00
llm_load_print_meta: n_ff             = 11008
llm_load_print_meta: n_expert         = 0
llm_load_print_meta: n_expert_used    = 0
llm_load_print_meta: causal attn      = 1
llm_load_print_meta: pooling type     = 0
llm_load_print_meta: rope type        = 0
llm_load_print_meta: rope scaling     = linear
llm_load_print_meta: freq_base_train  = 10000.0
llm_load_print_meta: freq_scale_train = 1
llm_load_print_meta: n_ctx_orig_yarn  = 4096
llm_load_print_meta: rope_finetuned   = unknown
llm_load_print_meta: ssm_d_conv       = 0
llm_load_print_meta: ssm_d_inner      = 0
llm_load_print_meta: ssm_d_state      = 0
llm_load_print_meta: ssm_dt_rank      = 0
llm_load_print_meta: model type       = ?B
llm_load_print_meta: model ftype      = F16
llm_load_print_meta: model params     = 1.26 B
llm_load_print_meta: model size       = 2.35 GiB (16.00 BPW)
llm_load_print_meta: general.name     = LLaMA v2
llm_load_print_meta: BOS token        = 1 '<s>'
llm_load_print_meta: EOS token        = 2 '</s>'
llm_load_print_meta: UNK token        = 0 '<unk>'
llm_load_print_meta: PAD token        = 0 '<unk>'
llm_load_print_meta: LF token         = 13 '<0x0A>'
llm_load_print_meta: max token length = 48
ggml_cuda_init: GGML_CUDA_FORCE_MMQ:    no
ggml_cuda_init: GGML_CUDA_FORCE_CUBLAS: no
ggml_cuda_init: found 1 CUDA devices:
  Device 0: NVIDIA GeForce RTX 4070 SUPER, compute capability 8.9, VMM: yes
llm_load_tensors: ggml ctx size =    0.02 MiB
llm_load_tensors: offloading 0 repeating layers to GPU
llm_load_tensors: offloaded 0/5 layers to GPU
llm_load_tensors:        CPU buffer size =  2408.14 MiB
..............................
llama_new_context_with_model: n_ctx      = 4096
llama_new_context_with_model: n_batch    = 2048
llama_new_context_with_model: n_ubatch   = 512
llama_new_context_with_model: flash_attn = 0
llama_new_context_with_model: freq_base  = 10000.0
llama_new_context_with_model: freq_scale = 1
llama_kv_cache_init:  CUDA_Host KV buffer size =   256.00 MiB
llama_new_context_with_model: KV self size  =  256.00 MiB, K (f16):  128.00 MiB, V (f16):  128.00 MiB
llama_new_context_with_model:  CUDA_Host  output buffer size =     0.21 MiB
llama_new_context_with_model:      CUDA0 compute buffer size =   642.00 MiB
llama_new_context_with_model:  CUDA_Host compute buffer size =    24.01 MiB
llama_new_context_with_model: graph nodes  = 134
llama_new_context_with_model: graph splits = 48

system_info: n_threads = 8 / 20 | AVX = 1 | AVX_VNNI = 0 | AVX2 = 1 | AVX512 = 0 | AVX512_VBMI = 0 | AVX512_VNNI = 0 | AVX512_BF16 = 0 | FMA = 1 | NEON = 0 | SVE = 0 | ARM_FMA = 0 | F16C = 1 | FP16_VA = 0 | WASM_SIMD = 0 | BLAS = 1 | SSE3 = 1 | SSSE3 = 1 | VSX = 0 | MATMUL_INT8 = 0 | LLAMAFILE = 1 |
main: interactive mode on.
Input prefix with BOS
Input prefix: ' [INST] '
Input suffix: ' [/INST]'
sampling:
        repeat_last_n = 64, repeat_penalty = 1.100, frequency_penalty = 0.000, presence_penalty = 0.000
        top_k = 40, tfs_z = 1.000, top_p = 0.900, min_p = 0.050, typical_p = 1.000, temp = 0.500
        mirostat = 0, mirostat_lr = 0.100, mirostat_ent = 5.000
sampling order:
CFG -> Penalties -> top_k -> tfs_z -> typical_p -> top_p -> min_p -> temperature
generate: n_ctx = 4096, n_batch = 2048, n_predict = -1, n_keep = 1


== Running in interactive mode. ==
 - Press Ctrl+C to interject at any time.
 - Press Return to return control to the AI.
 - To return control without starting a new line, end your input with '/'.
 - If you want to submit another line, end your input with '\'.

 [INST] <<SYS>>
You are a helpful assistant. 你是一个乐于助人的助手。
<</SYS>>

背一下静夜思 [/INST] 床前明月光，疑是地上霜。举头望明月，低头思故乡。
 [INST]
```

{% endfold %}

## 模型选择指引

以下是中文LLaMA-2和Alpaca-2模型的对比以及建议使用场景。**如需聊天交互，请选择Alpaca而不是LLaMA。**

| 对比项                   |                                   中文LLaMA-2                                    |                                   中文Alpaca-2                                   |
| :----------------------- | :------------------------------------------------------------------------------: | :------------------------------------------------------------------------------: |
| 模型类型                 |                                   **基座模型**                                   |                          **指令/Chat模型（类ChatGPT）**                          |
| 已开源大小               |                                  1.3B、7B、13B                                   |                                  1.3B、7B、13B                                   |
| 训练类型                 |                                 Causal-LM (CLM)                                  |                                     指令精调                                     |
| 训练方式                 |                   7B、13B：LoRA + 全量emb/lm-head; 1.3B：全量                    |                   7B、13B：LoRA + 全量emb/lm-head; 1.3B：全量                    |
| 基于什么模型训练         |       [原版Llama-2](https://github.com/facebookresearch/llama)（非chat版）       |                                   中文LLaMA-2                                    |
| 训练语料                 |                           无标注通用语料（120G纯文本）                           |                            有标注指令数据（500万条）                             |
| 词表大小<sup>[1]</sup>   |                                      55,296                                      |                                      55,296                                      |
| 上下文长度<sup>[2]</sup> | 标准版：4K（12K-18K）; 长上下文版（PI）：16K（24K-32K）; 长上下文版（YaRN）：64K | 标准版：4K（12K-18K）; 长上下文版（PI）：16K（24K-32K）; 长上下文版（YaRN）：64K |
| 输入模板                 |                                      不需要                                      |                 需要套用特定模板<sup>[3]</sup>，类似Llama-2-Chat                 |
| 适用场景                 |                        文本续写：给定上文，让模型生成下文                        |                        指令理解：问答、写作、聊天、交互等                        |
| 不适用场景               |                              指令理解 、多轮聊天等                               |                                文本无限制自由生成                                |
| 偏好对齐                 |                                        无                                        |                               RLHF版本（1.3B、7B）                               |

> [!NOTE]
> [1] *本项目一代模型和二代模型的词表不同，请勿混用。二代LLaMA和Alpaca的词表相同。*
> [2] *括号内表示基于NTK上下文扩展支持的最大长度。*
> [3] *Alpaca-2采用了Llama-2-chat系列模板（格式相同，提示语不同），而不是一代Alpaca的模板，请勿混用。*
> [4] *不建议单独使用1.3B模型，而是通过投机采样搭配更大的模型（7B、13B）使用。*

## 完整模型下载

以下是完整版模型，直接下载即可使用，无需其他合并步骤。推荐网络带宽充足的用户。

| 模型名称              |   类型   |  大小   |                                                                                                                                                           下载地址                                                                                                                                                           |                              GGUF                              |
| :-------------------- | :------: | :-----: | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------: | :------------------------------------------------------------: |
| Chinese-LLaMA-2-13B   | 基座模型 | 24.7 GB |   [[Baidu]](https://pan.baidu.com/s/1T3RqEUSmyg6ZuBwMhwSmoQ?pwd=e9qy) [[Google]](https://drive.google.com/drive/folders/1YNa5qJ0x59OEOI7tNODxea-1YvMPoH05?usp=share_link) ；[[🤗HF]](https://huggingface.co/hfl/chinese-llama-2-13b) ；[[🤖ModelScope]](https://modelscope.cn/models/ChineseAlpacaGroup/chinese-llama-2-13b)   |  [[🤗HF]](https://huggingface.co/hfl/chinese-llama-2-13b-gguf)  |
| Chinese-LLaMA-2-7B    | 基座模型 | 12.9 GB |    [[Baidu]](https://pan.baidu.com/s/1E5NI3nlQpx1j8z3eIzbIlg?pwd=n8k3) [[Google]](https://drive.google.com/drive/folders/18pp4I-mvQxRA7b8vF9gP-2cH_ocnXVKh?usp=share_link) ；[[🤗HF]](https://huggingface.co/hfl/chinese-llama-2-7b) ；[[🤖ModelScope]](https://modelscope.cn/models/ChineseAlpacaGroup/chinese-llama-2-7b)    |  [[🤗HF]](https://huggingface.co/hfl/chinese-llama-2-7b-gguf)   |
| Chinese-LLaMA-2-1.3B  | 基座模型 | 2.4 GB  |  [[Baidu]](https://pan.baidu.com/s/1hEuOCllnJJ5NMEZJf8OkRw?pwd=nwjg) [[Google]](https://drive.google.com/drive/folders/1Sd3PA_gs6JctXtBg5HwmHXh9GX93riMP?usp=share_link) ；[[🤗HF]](https://huggingface.co/hfl/chinese-llama-2-1.3b) ；[[🤖ModelScope]](https://modelscope.cn/models/ChineseAlpacaGroup/chinese-llama-2-1.3b)  | [[🤗HF]](https://huggingface.co/hfl/chinese-llama-2-1.3b-gguf)  |
| Chinese-Alpaca-2-13B  | 指令模型 | 24.7 GB |  [[Baidu]](https://pan.baidu.com/s/1MT_Zlap1OtdYMgoBNTS3dg?pwd=9xja) [[Google]](https://drive.google.com/drive/folders/1MTsKlzR61xmbTR4hBWzQas_MOpUZsogN?usp=share_link) ；[[🤗HF]](https://huggingface.co/hfl/chinese-alpaca-2-13b) ；[[🤖ModelScope]](https://modelscope.cn/models/ChineseAlpacaGroup/chinese-alpaca-2-13b)  | [[🤗HF]](https://huggingface.co/hfl/chinese-alpaca-2-13b-gguf)  |
| Chinese-Alpaca-2-7B   | 指令模型 | 12.9 GB |   [[Baidu]](https://pan.baidu.com/s/1wxx-CdgbMupXVRBcaN4Slw?pwd=kpn9) [[Google]](https://drive.google.com/drive/folders/1JsJDVs7tE2y31PBNleBlDPsB7S0ZrY8d?usp=share_link) ；[[🤗HF]](https://huggingface.co/hfl/chinese-alpaca-2-7b) ；[[🤖ModelScope]](https://modelscope.cn/models/ChineseAlpacaGroup/chinese-alpaca-2-7b)   |  [[🤗HF]](https://huggingface.co/hfl/chinese-alpaca-2-7b-gguf)  |
| Chinese-Alpaca-2-1.3B | 指令模型 | 2.4 GB  | [[Baidu]](https://pan.baidu.com/s/1PD7Ng-ltOIdUGHNorveptA?pwd=ar1p) [[Google]](https://drive.google.com/drive/folders/1h6qOy-Unvqs1_CJ8uPp0eKC61Gbbn8n7?usp=share_link) ；[[🤗HF]](https://huggingface.co/hfl/chinese-alpaca-2-1.3b) ；[[🤖ModelScope]](https://modelscope.cn/models/ChineseAlpacaGroup/chinese-alpaca-2-1.3b) | [[🤗HF]](https://huggingface.co/hfl/chinese-alpaca-2-1.3b-gguf) |
