---
layout: post
title: GPU Computing Startup
date: 2024-06-24 23:46:14
index_img: /img/ai/AGI.jpeg
sticky: 100
tags:
    - Algorithm
    - CUDA
    - Pytorch
    - AI
    - GPU
categories: 
    - AI
---

GPU, Compute and AI.

<!-- more -->

## 目录

- [目录](#目录)
- [资料汇总](#资料汇总)
- [论文](#论文)
- [pytorch 知识点整理](#pytorch-知识点整理)
- [安装 pytorch cpu 版本](#安装-pytorch-cpu-版本)
- [安装 pytorch cuda 版本](#安装-pytorch-cuda-版本)
- [pytorch book vscode 环境配置](#pytorch-book-vscode-环境配置)
- [示例](#示例)
  - [线性回归](#线性回归)
  - [fashion mnist 手写数字分类](#fashion-mnist-手写数字分类)
  - [cifar 10](#cifar-10)

## 资料汇总

| 链接  | 说明 |
| ---- | --- |
| [**《深度学习框架PyTorch：入门与实战》代码**](https://github.com/chenyuntc/pytorch-book)| 这个适合开始阶段，参考[pytorch 知识点整理](#pytorch-知识点整理)。此书配套1.6版本的pytorch，参考[安装 pytorch cpu 版本](#安装-pytorch-cpu-版本)和[pytorch book vscode 环境配置](#pytorch-book-vscode-环境配置) |
| [Pytorch 官方文档](https://pytorch.org/tutorials/beginner/basics/intro.html)  | 最新的是2.3版本，和下面教程搭配着看 |
| [Pytorch 官方文档教程仓库](https://github.com/pytorch/tutorials) | PyTorch tutorials。有点占地方，里面和文档内容对应，有直接可运行的脚本  |
| [Pytorch 安装指引](https://pytorch.org/get-started/locally/)| 安装CPU/CUDA版本 |
| [Pytorch 官方文档翻译版](https://github.com/apachecn/pytorch-doc-zh)| Pytorch 中文文档，缺点，广告太多 |
| [Pytorch Examples](https://github.com/pytorch/examples)| 围绕 pytorch 的视觉、文本、强化学习等方面的一组示例。 |
| [Pytorch 源码仓库](https://github.com/pytorch/pytorch)| 具有强大 GPU 加速的 Python 张量和动态神经网络。 |

| Cuda | Jetson 嵌入式AI | GPU driver |
| :--: | :--: | :--: |
| [CUDA 官方文档](https://docs.nvidia.com/cuda/)<br/>[CUDA Runtime API](https://docs.nvidia.com/cuda/cuda-runtime-api/index.html)<br/>[**Samples for CUDA Developers**](https://github.com/NVIDIA/cuda-samples) | [**Jetson Orin**](https://www.nvidia.com/en-us/autonomous-machines/embedded-systems/jetson-orin/)<br/>[**NVIDIA Jetson AI Lab**](https://www.jetson-ai-lab.com/tutorial-intro.html)<br/>[**NVIDIA Jetson-projects**](https://developer.nvidia.com/embedded/community/jetson-projects)<br/>[**Yahboom官方教程 提取码lguu**](https://www.yahboom.com/study/Jetson-Orin-NANO)<br/>[Jetson Orin Nano Developer Kit Now Available](https://developer.nvidia.com/blog/develop-ai-powered-robots-smart-vision-systems-and-more-with-nvidia-jetson-orin-nano-developer-kit) | [**NVIDIA GPU Kernel Modules**](https://github.com/NVIDIA/open-gpu-kernel-modules)<br/>[AMD KMD Driver in Linux](https://github.com/torvalds/linux/tree/master/drivers/gpu/drm/amd/amdgpu) |

{% fold @Github 资源汇总 %}

| 仓库                                     |说明                 |
| :---------------------------------------: | ------------------ |
| **1. AI/AGI/AIoT** |  |
| [**HuggingFace/Transformers<br/>★★★★★**](https://github.com/huggingface/transformers) | 著名论文[Attention Is All You Need](https://arxiv.org/abs/1706.03762) 提出的 Transformers  提供数千个预训练模型，用于执行不同模态（例如文本、视觉和音频）的任务。这些模型可应用于：<br/>1. 📝 文本，用于 100 多种语言的文本分类、信息提取、问答、摘要、翻译和文本生成等任务。<br/>2. 🖼️ 图像，用于图像分类、对象检测和分割等任务。<br/>3. 🗣️ 音频，用于语音识别和音频分类等任务。<br/>Transformers 模型还可以执行多种模态组合的任务，例如表格问答、光学字符识别、从扫描文档中提取信息、视频分类和视觉问答。 |
| [**Karpathy/llm.c<br/>★★★★★**](https://github.com/karpathy/llm.c)| 简单、纯 C/CUDA 的 LLM，无需 245MB 的 PyTorch 或 107MB 的 cPython。当前重点是预训练，特别是重现 GPT-2 和 GPT-3 迷你剧，以及 train_gpt2.py 中的并行 PyTorch 参考实现。测试见：[llm.c](http://jianliang-shen.cn/2024/04/28/llm.c/) |
| [**Google/Vision Transformer<br/>★★★★★**](https://github.com/google-research/vision_transformer) |在这个存储库中，我们发布了论文中的模型<br/>一张图片胜过 16x16 个单词：用于大规模图像识别的 Transformers<br/>MLP-Mixer：用于视觉的全 MLP 架构<br/>如何训练你的 ViT？视觉 Transformers 中的数据、增强和正则化<br/>当视觉 Transformers 在没有预训练或强大的数据增强的情况下胜过 ResNets 时<br/>LiT：使用锁定图像文本调整的零样本传输<br/>替代间隙最小化改进了清晰度感知训练<br/>这些模型在 ImageNet 和 ImageNet-21k 数据集上进行了预训练。我们在 JAX/Flax 中提供了用于微调已发布模型的代码。<br/>|
| [**Ultralytics/Yolov5<br/>★★★★★**](https://github.com/ultralytics/yolov5)| YOLOv5🚀是世界上最受欢迎的视觉 AI，代表了 Ultralytics 对未来视觉 AI 方法的开源研究，融合了数千小时研发过程中获得的经验教训和最佳实践。|
| [**Dusty-nv/Jetson Inference<br/>★★★★★**](https://github.com/dusty-nv/jetson-inference)| 该项目使用 TensorRT 在 C++ 或 Python 的 GPU 上运行优化网络，并使用 PyTorch 训练模型。支持的 DNN 视觉基元包括用于图像分类的 imageNet、用于对象检测的 detectNet、用于语义分割的 segNet、用于姿势估计的 poseNet 和用于动作识别的 actionNet。提供了从实时摄像头源进行流式传输、使用 WebRTC 制作 Web 应用程序以及对 ROS/ROS2 的支持的示例。 |
| [**Stable Diffusion WebUI<br/>★★★★★**](https://github.com/AUTOMATIC1111/stable-diffusion-webui)| 使用 Gradio 库实现的Stable Diffusion的 Web 界面。 [简体中文翻译扩展](https://github.com/dtlnor/stable-diffusion-webui-localization-zh_CN)|
| [**Zhouyi-AIPU/Model Zoo<br/>★★★**](https://github.com/Zhouyi-AIPU/Model_zoo)| 各种Embedded model汇总 |
| [HuggingFace/Pytorch image models<br/>★](https://github.com/huggingface/pytorch-image-models) |最大的 PyTorch 图像 encoders / backbone 集合。包括训练、评估、推理、导出脚本和预训练权重 - ResNet、ResNeXT、EfficientNet、NFNet、Vision Transformer (ViT)、MobileNetV4、MobileNet-V3 & V2、RegNet、DPN、CSPNet、Swin Transformer、MaxViT、CoAtNet、ConvNeXt 等|
| [HuggingFace/Datasets<br/>★](https://github.com/huggingface/datasets) |Datasets 是一个轻量级库，提供两个主要功能：适用于许多公共数据集的单行数据加载器；高效的数据预处理。|
| [HuggingFace/Accelerate<br/>★](https://github.com/huggingface/accelerate) | Accelerate 是为那些喜欢编写 PyTorch 模型训练循环但不愿意编写和维护使用多 GPU/TPU/fp16 所需的样板代码的 PyTorch 用户创建的。 |
| [Stability-AI/Generative Models<br/>★](https://github.com/Stability-AI/generative-models)| Generative Models by Stability AI |
| [Stability-AI/Stable Diffusion<br/>★](https://github.com/Stability-AI/stablediffusion) | 此存储库包含从头开始训练的 Stable Diffusion 模型，并将使用新的检查点不断更新。 |
| [TensorFlow<br/>★](https://github.com/tensorflow/tensorflow) | An Open Source Machine Learning Framework for Everyone |
| [TensorFlow Models<br/>★](https://github.com/tensorflow/models) | Models and examples built with TensorFlow |
| [Ultralytics/ultralytics<br/>★](https://github.com/ultralytics/ultralytics) | Ultralytics YOLOv8 是一款尖端的、最先进的 (SOTA) 模型，它以之前 YOLO 版本的成功为基础，并引入了新功能和改进，以进一步提高性能和灵活性。YOLOv8 旨在快速、准确且易于使用，使其成为各种对象检测和跟踪、实例分割、图像分类和姿势估计任务的绝佳选择。 |
| [Karpathy/llama2.c<br/>★](https://github.com/karpathy/llama2.c)| 在 PyTorch 中训练 Llama 2 LLM 架构，然后使用一个简单的 700 行 C 文件 (run.c) 进行推理。|
| [GPT2-Chinese<br/>★](https://github.com/Morizeyao/GPT2-Chinese)|中文的GPT2训练代码，使用BERT的Tokenizer或Sentencepiece的BPE model |
| [openai-cookbook<br/>★★](https://github.com/openai/openai-cookbook)|使用 OpenAI API 完成常见任务的示例代码和指南。 |
| [ONNX<br/>★★★](https://github.com/onnx/onnx)|开放神经网络交换(ONNX)是一个开放的生态系统，使人工智能开发人员能够随着项目的发展选择合适的工具。ONNX为人工智能模型提供了一种开源格式，包括深度学习和传统ML，它定义了一个可扩展的计算图模型，以及内置运算符和标准数据类型的定义。目前我们专注于推理(评分)所需的功能。 |
| [Microsoft/ONNX Runtime<br/>★](https://github.com/microsoft/onnxruntime)|ONNX Runtime 是一个跨平台推理和训练机器学习加速器。|
| [onnx-tensorrt<br/>★★](https://github.com/onnx/onnx-tensorrt) | 解析 ONNX 模型以便使用 [TensorRT](https://developer.nvidia.com/tensorrt) 执行。 NVIDIA® TensorRT™ 是一个用于高性能深度学习推理的 API 生态系统。TensorRT 包括推理运行时和模型优化，可为生产应用程序提供低延迟和高吞吐量。TensorRT 生态系统包括 TensorRT、TensorRT-LLM、TensorRT 模型优化器和 TensorRT Cloud。|
| [onnx-simplifier<br/>★](https://github.com/daquexian/onnx-simplifier) | ONNX 很棒，但有时太复杂。 |
| [tensorflow-onnx<br/>★](https://github.com/onnx/tensorflow-onnx) | tf2onnx 通过命令行或 python api 将 TensorFlow（tf-1.x 或 tf-2.x）、keras、tensorflow.js 和 tflite 模型转换为 ONNX。 |
| **2. GPU/CUDA/Rocm** |
| [**CUDA-Learn-Notes<br/>★★★★★**](https://github.com/DefTruth/CUDA-Learn-Notes)|CUDA-Learn-Notes: CUDA 笔记、大模型手撕CUDA、C++笔记 |
| [**CUDA-Programming-Guide-in-Chinese<br/>★**](https://github.com/HeKun-NVIDIA/CUDA-Programming-Guide-in-Chinese)| 本项目为 CUDA C Programming Guide 的中文翻译版。|
| [NN-CUDA-Example<br/>★](https://github.com/godweiyang/NN-CUDA-Example)|调用自定义 CUDA 运算符的神经网络工具包（PyTorch、TensorFlow 等）的几个简单示例。 |
| [NVTrust<br/>★](https://github.com/NVIDIA/nvtrust)|nvTrust 是一个存储库，其中包含在受信任的环境（例如机密计算）中使用 NVIDIA 解决方案时利用的许多实用程序和工具、开源代码和 SDK。 |
| [LLM Guard](https://github.com/protectai/llm-guard) | LLM 交互的安全工具包 |
| [ROCm/ROCT-Thunk-Interface<br/>★](https://github.com/ROCm/ROCT-Thunk-Interface)|此存储库包含用于与 (AMD)ROCk 驱动程序交互的用户模式 ​​API 接口。 |
| **3. 免费资源** | |
| [**GitHub中文开源仓库排行榜<br/>★★★★★**](https://github.com/GrowingGit/GitHub-Chinese-Top-Charts)| GitHub中文排行榜，帮助你发现优秀中文项目，可以无语言障碍地、更高效地吸收优秀经验成果 |
| [**free-programming-books<br/>★★★★★**](https://github.com/EbookFoundation/free-programming-books)| 多种语言的免费学习资源列表 |
| [CS EBook<br/>★★★](https://github.com/forthespada/CS-Books)| 超过1000本的计算机经典书籍分享，解压密码：a123654 |
| [CS EBook<br/>★★★](https://github.com/lining808/CS-Ebook) | 本储存库是一些高质量的计算机科学与技术书籍推荐书单，需要学习的可以按照此书单进行学习进阶，包含了计算机大多数软件相关方向。而且敢承诺一直更新。 |
| [zhoucz97/myLearning](https://github.com/zhoucz97/myLearning) | 记录个人的学习历程。包括但不限于算法、机器学习、论文写作等。 |

{% endfold %}

{% fold @PyTorch文档 %}

<iframe src="https://pytorch.org/tutorials/beginner/basics/intro.html" width="100%" height="800" name="topFrame" scrolling="yes"  noresize="noresize" frameborder="0" id="topFrame"></iframe>

{% endfold %}

{% fold @CUDA文档 %}

<iframe src="https://docs.nvidia.com/cuda/" width="100%" height="800" name="topFrame" scrolling="yes"  noresize="noresize" frameborder="0" id="topFrame"></iframe>

{% endfold %}

## 论文

[Attention is All Your Need 中英对照](https://www.yiyibooks.cn/yiyibooks/Attention_Is_All_You_Need/index.html)

## pytorch 知识点整理

《深度学习框架 PyTorch: 入门与实战》

{% fold @Chapter 2, 简单介绍 tensor 和构建 cifar-10 训练模型 %}

- 安装Pytorch
- 基本操作，如cat等
- 准备一个cifar-10模型，并训练推理

{% endfold %}

{% fold @Chapter 3, 介绍 tensor %}

| 概要                   | 内容               |
| ---------------------- | -------------------|
| 基本操作               | Tensor(\*sizes), tensor(data)<br/>ones/zeros/eye(\*sizes)<br/>arrange(start, end, step), linspace(start, end, steps)<br/>rand / randn(\*sizes)<br/>new_\* / \*_like()   |
| 命名张量               | names, refine_names, rename, align_to  |
| 类型                   | set_default_tensor_type, type(new_type) == .float(), .long(), .half(), to(device)  |
| 索引                   | index_select(input, dim, index)<br/>masked_select(input, mask)<br/>gather(input, dim, index)<br/>input.scatter_(dim, index)放回<br/>non_zero(input)非零下标   |
| 元素操作               | abs/sqrt/div/exp/fmod/log/pow, cos/sin/asin/atan2/cosh, ceil/round/floor/trunc, clamp(input,min,max), sigmod/tanh, cumsum/cumprod  |
| 归并操作               | mean/sum/median/mode, norm/dist, std/var, keepdim=True<br/>保留维度, 在哪个维度操作, 哪个维度变成1，或者消失   |
| 比较                   | gt/lt/ge/le/eq/ne, topk, sort, max/min |
| 线性代数               | trace, diag, triu/tril, mm/bmm, addmm/addbmm/addmv, t, dot/cross, inverse, svd   |
| Numpy                  | from_numpy，共享内存<br/>torch.tensor()只进行数据拷贝, 不会共享内存<br/>torch.Tensor()在类型不一致时是复制而非共享内存  |
| Tensor基本结构         | storage()查看是否共享, contiguous()变成连续   |
| Tensor改变形状         | 查看信息, size() = shape, dim() = len(tensor.shape), numel <=> numpy.size<br/>改变维度, reshape(), view(), view_as()<br/>增加减少维度, squeeze()压缩, unsqueeze()新建维度, flatten(start_dim, end_dim)<br/>转置, transpose()仅限二维, t(), T, permute() |
| 线性回归实例           |   手动计算求导函数     |
| autograd               | requires_grad=True, retain_graph=None, is_leaf, backward()    |
| 用autograd实现线性回归 |  自动backward, 梯度下降  |

{% endfold %}

{% fold @Chapter 4, 神经网络工具箱nn %}

torch.nn是专门为深度学习而设计的模块。torch.nn的核心数据结构是`Module`，它是一个抽象的概念，既可以表示神经网络中的某个层（layer），也可以表示一个包含很多层的神经网络。在实际使用中，最常见的做法是继承`nn.Module`，从而编写自己的网络/层。多层感知机的网络结构如图所示，它由两个全连接层组成，采用$sigmoid$函数作为激活函数（图中没有画出）。

![神经网络](/img/ai/multi_perceptron.png)

PyTorch内部实现了神经网络中绝大多数的layer，这些layer都继承于`nn.Module`，封装了可学习参数`parameter`，并实现了`forward`函数。同时，大部分layer都专门针对GPU运算进行了CuDNN优化，其速度和性能都十分优异。关注每一层的信息有：

1. 构造函数的参数，如nn.Linear(in_features, out_features, bias)，需关注这三个参数的作用；
2. 属性、可学习参数和子module。如nn.Linear中有`weight`和`bias`两个可学习参数，不包含子module；
3. 输入输出的形状，如nn.linear的输入形状是(N, input_features)，输出为(N，output_features)，其中N是batch_size。

图像nn包括，卷积层（Conv）、池化层（Pool），池化方式又分为平均池化（AvgPool）、最大值池化（MaxPool）、自适应池化（AdaptiveAvgPool）等。而卷积层除了常用的前向卷积之外，还有逆卷积（TransposeConv）。卷积神经网络的本质就是卷积层、池化层、激活层以及其他层的叠加。池化层可以看作是一种特殊的卷积层，其主要用于下采样，增加池化层可以在保留主要特征的同时降低参数量，从而一定程度上防止了过拟合。池化层没有可学习参数，它的weight是固定的。在`torch.nn`工具箱中封装好了各种池化层，常见的有最大池化（MaxPool）和平均池化（AvgPool)。

{% endfold %}

## 安装 pytorch cpu 版本

<https://pytorch.org/get-started/previous-versions/>

{% fold @点击此处查看步骤 %}

![Pytorch](/img/ai/Pytorch.png)

**安装 Anaconda. 下载地址:** <https://www.anaconda.com/download>

```bash
# For example, Ubuntu
$ wget https://repo.anaconda.com/archive/Anaconda3-2024.02-1-Linux-x86_64.sh
$ chmod +x Anaconda3-2024.02-1-Linux-x86_64.sh
$ ./Anaconda3-2024.02-1-Linux-x86_64.sh
$ echo "export PATH=\"/root/anaconda3/bin\":\$PATH" >> ~/.bashrc   # Check the path
$ source ~/.bashrc

# Check if it is installed successfully:
$ conda --version
conda 24.1.2
```

**换源**

```bash
# Reference https://blog.csdn.net/adreammaker/article/details/123396951
$ conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/
$ conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main/
$ conda config --set show_channel_urls yes
```

**创建Pytorch虚拟环境**

```bash
conda create -n torch-1.6  python=3.6.13
```

<details>
<summary>创建过程的log</summary>

```bash
$ conda create -n torch-1.6  python=3.6.13
Channels:
 - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main
 - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free
 - defaults
Platform: linux-64
Collecting package metadata (repodata.json): done
Solving environment: done

## Package Plan ##

  environment location: /home/shenjianliang/anaconda3/envs/torch-1.6

  added / updated specs:
    - python=3.6.13


The following packages will be downloaded:

    package                    |            build
    ---------------------------|-----------------
    certifi-2021.5.30          |   py36h06a4308_0         139 KB  https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main
    libffi-3.3                 |       he6710b0_2          50 KB  https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main
    openssl-1.1.1w             |       h7f8727e_0         3.7 MB  https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main
    pip-21.2.2                 |   py36h06a4308_0         1.8 MB  https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main
    python-3.6.13              |       h12debd9_1        32.5 MB  https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main
    setuptools-58.0.4          |   py36h06a4308_0         788 KB  https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main
    wheel-0.37.1               |     pyhd3eb1b0_0          33 KB  https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main
    ------------------------------------------------------------
                                           Total:        39.0 MB

The following NEW packages will be INSTALLED:

  _libgcc_mutex      anaconda/pkgs/main/linux-64::_libgcc_mutex-0.1-main
  _openmp_mutex      anaconda/pkgs/main/linux-64::_openmp_mutex-5.1-1_gnu
  ca-certificates    anaconda/pkgs/main/linux-64::ca-certificates-2024.3.11-h06a4308_0
  certifi            anaconda/pkgs/main/linux-64::certifi-2021.5.30-py36h06a4308_0
  ld_impl_linux-64   anaconda/pkgs/main/linux-64::ld_impl_linux-64-2.38-h1181459_1
  libffi             anaconda/pkgs/main/linux-64::libffi-3.3-he6710b0_2
  libgcc-ng          anaconda/pkgs/main/linux-64::libgcc-ng-11.2.0-h1234567_1
  libgomp            anaconda/pkgs/main/linux-64::libgomp-11.2.0-h1234567_1
  libstdcxx-ng       anaconda/pkgs/main/linux-64::libstdcxx-ng-11.2.0-h1234567_1
  ncurses            anaconda/pkgs/main/linux-64::ncurses-6.4-h6a678d5_0
  openssl            anaconda/pkgs/main/linux-64::openssl-1.1.1w-h7f8727e_0
  pip                anaconda/pkgs/main/linux-64::pip-21.2.2-py36h06a4308_0
  python             anaconda/pkgs/main/linux-64::python-3.6.13-h12debd9_1
  readline           anaconda/pkgs/main/linux-64::readline-8.2-h5eee18b_0
  setuptools         anaconda/pkgs/main/linux-64::setuptools-58.0.4-py36h06a4308_0
  sqlite             anaconda/pkgs/main/linux-64::sqlite-3.45.3-h5eee18b_0
  tk                 anaconda/pkgs/main/linux-64::tk-8.6.14-h39e8969_0
  wheel              anaconda/pkgs/main/noarch::wheel-0.37.1-pyhd3eb1b0_0
  xz                 anaconda/pkgs/main/linux-64::xz-5.4.6-h5eee18b_1
  zlib               anaconda/pkgs/main/linux-64::zlib-1.2.13-h5eee18b_1


Proceed ([y]/n)? y


Downloading and Extracting Packages:

Preparing transaction: done
Verifying transaction: done
Executing transaction: done
#
# To activate this environment, use
#
#     $ conda activate torch-1.6
#
# To deactivate an active environment, use
#
#     $ conda deactivate
```

</details>

**激活环境**

```bash
conda activate torch-1.6
```

**安装Pytorch 1.6**

```bash
conda install pytorch==1.6.0 torchvision==0.7.0 -c pytorch
```

<details>
<summary>安装的log</summary>

```bash
$ conda install pytorch==1.6.0 torchvision==0.7.0 -c pytorch
Channels:
 - pytorch
 - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main
 - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free
 - defaults
Platform: linux-64
Collecting package metadata (repodata.json): done
Solving environment: done

## Package Plan ##

  environment location: /home/shenjianliang/anaconda3/envs/torch-1.6

  added / updated specs:
    - pytorch==1.6.0
    - torchvision==0.7.0


The following packages will be downloaded:

    package                    |            build
    ---------------------------|-----------------
    cudatoolkit-10.2.89        |       hfd86e86_1       365.1 MB  https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main
    giflib-5.2.1               |       h5eee18b_3          80 KB  https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main
    intel-openmp-2022.1.0      |    h9e868ea_3769         4.5 MB  https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main
    libgfortran-ng-7.5.0       |      ha8ba4b0_17          22 KB  https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main
    libgfortran4-7.5.0         |      ha8ba4b0_17         995 KB  https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main
    libwebp-1.2.4              |       h11a3e52_1          86 KB  https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main
    libwebp-base-1.2.4         |       h5eee18b_1         376 KB  https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main
    mkl-2022.1.0               |     hc2b9512_224       129.7 MB  https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main
    ninja-1.10.2               |       h06a4308_5           8 KB  https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main
    ninja-base-1.10.2          |       hd09550d_5         109 KB  https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main
    numpy-1.14.2               |   py36hdbf6ddf_0         3.2 MB  https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main
    olefile-0.46               |     pyhd3eb1b0_0          34 KB  https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main
    pillow-8.3.1               |   py36h5aabda8_0         638 KB  https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main
    pytorch-1.6.0              |py3.6_cuda10.2.89_cudnn7.6.5_0       537.3 MB  pytorch
    torchvision-0.7.0          |       py36_cu102        11.0 MB  pytorch
    ------------------------------------------------------------
                                           Total:        1.03 GB

The following NEW packages will be INSTALLED:

  blas               anaconda/pkgs/main/linux-64::blas-1.0-mkl
  cudatoolkit        anaconda/pkgs/main/linux-64::cudatoolkit-10.2.89-hfd86e86_1
  freetype           anaconda/pkgs/main/linux-64::freetype-2.12.1-h4a9f257_0
  giflib             anaconda/pkgs/main/linux-64::giflib-5.2.1-h5eee18b_3
  intel-openmp       anaconda/pkgs/main/linux-64::intel-openmp-2022.1.0-h9e868ea_3769
  jpeg               anaconda/pkgs/main/linux-64::jpeg-9e-h5eee18b_1
  lcms2              anaconda/pkgs/main/linux-64::lcms2-2.12-h3be6417_0
  lerc               anaconda/pkgs/main/linux-64::lerc-3.0-h295c915_0
  libdeflate         anaconda/pkgs/main/linux-64::libdeflate-1.17-h5eee18b_1
  libgfortran-ng     anaconda/pkgs/main/linux-64::libgfortran-ng-7.5.0-ha8ba4b0_17
  libgfortran4       anaconda/pkgs/main/linux-64::libgfortran4-7.5.0-ha8ba4b0_17
  libpng             anaconda/pkgs/main/linux-64::libpng-1.6.39-h5eee18b_0
  libtiff            anaconda/pkgs/main/linux-64::libtiff-4.5.1-h6a678d5_0
  libwebp            anaconda/pkgs/main/linux-64::libwebp-1.2.4-h11a3e52_1
  libwebp-base       anaconda/pkgs/main/linux-64::libwebp-base-1.2.4-h5eee18b_1
  lz4-c              anaconda/pkgs/main/linux-64::lz4-c-1.9.4-h6a678d5_1
  mkl                anaconda/pkgs/main/linux-64::mkl-2022.1.0-hc2b9512_224
  ninja              anaconda/pkgs/main/linux-64::ninja-1.10.2-h06a4308_5
  ninja-base         anaconda/pkgs/main/linux-64::ninja-base-1.10.2-hd09550d_5
  numpy              anaconda/pkgs/main/linux-64::numpy-1.14.2-py36hdbf6ddf_0
  olefile            anaconda/pkgs/main/noarch::olefile-0.46-pyhd3eb1b0_0
  pillow             anaconda/pkgs/main/linux-64::pillow-8.3.1-py36h5aabda8_0
  pytorch            pytorch/linux-64::pytorch-1.6.0-py3.6_cuda10.2.89_cudnn7.6.5_0
  torchvision        pytorch/linux-64::torchvision-0.7.0-py36_cu102
  zstd               anaconda/pkgs/main/linux-64::zstd-1.5.5-hc292b87_2


Proceed ([y]/n)? y


Downloading and Extracting Packages:

Preparing transaction: done
Verifying transaction: done
Executing transaction: done
```

</details>

**Conda删除环境**

```bash
# 退出当前环境
$ conda deactivate

# 列出当前env
$ conda env list
# conda environments:
#
base                     /home/shenjianliang/anaconda3
torch-1.6             *  /home/shenjianliang/anaconda3/envs/torch-1.6

# 删除
$ conda env remove -p /home/shenjianliang/anaconda3/envs/torch-1.6
```

**不用Anaconda，使用python虚拟env**

```bash
sudo apt-get install python3-venv
python3.6 -m venv myenv
source myenv/bin/activate

pip install torch==1.6.0+cpu torchvision==0.7.0+cpu -f https://download.pytorch.org/whl/torch_stable.html
```

{% endfold %}

## 安装 pytorch cuda 版本

在RTX4070S windows中配置WSL相关的AI环境，包括CUDA，PyTorch，Cudnn等

{% fold @步骤 %}

**安装WSL/Docker/Nvidia：**
[Windows 下让 Docker Desktop 关联上 NVidia GPU](https://blog.csdn.net/ndscvipuser/article/details/136610169)
[如何查看wsl是wsl1还是wsl2](https://blog.csdn.net/dghcs18/article/details/134244426)
[Nvidia WSL官方指引](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html)
[linux上cuda相关包与opencv及相关模块安装（wsl+ubuntu22.04）](https://blog.csdn.net/weixin_55274216/article/details/137630257)

>注意：WSL不需要装cuda驱动，丢在win host安装，比如Geforce等驱动软件，安装完成后运行`nvidia-smi`

```bash
# Powershell中测试
docker run --rm --runtime=nvidia --gpus all ubuntu nvidia-smi
```

Docker Desktop中的设置：

![1](/img/ai/docker_set1.png)
![2](/img/ai/docker_set2.png)

测试结果：

![3](/img/ai/docker.png)

WSL中测试：

![4](/img/ai/wsl.png)

安装[cuda toolkit](https://developer.nvidia.cn/cuda-downloads)

```bash
wget https://developer.download.nvidia.com/compute/cuda/repos/wsl-ubuntu/x86_64/cuda-wsl-ubuntu.pin
sudo mv cuda-wsl-ubuntu.pin /etc/apt/preferences.d/cuda-repository-pin-600
wget https://developer.download.nvidia.com/compute/cuda/12.4.1/local_installers/cuda-repo-wsl-ubuntu-12-4-local_12.4.1-1_amd64.deb
sudo dpkg -i cuda-repo-wsl-ubuntu-12-4-local_12.4.1-1_amd64.deb
sudo cp /var/cuda-repo-wsl-ubuntu-12-4-local/cuda-*-keyring.gpg /usr/share/keyrings/
sudo apt-get update
sudo apt-get -y install cuda-toolkit-12-4

# ~/.bashrc
export LD_LIBRARY_PATH=/usr/local/cuda/lib64
export PATH=$PATH:/usr/local/cuda/bin

nvcc -V
nvcc: NVIDIA (R) Cuda compiler driver
Copyright (c) 2005-2024 NVIDIA Corporation
Built on Thu_Mar_28_02:18:24_PDT_2024
Cuda compilation tools, release 12.4, V12.4.131
Build cuda_12.4.r12.4/compiler.34097967_0
```

**安装Conda**

```bash
#创建环境
conda init
conda create -n torch-gpu  python=3.9
conda activate torch-gpu

#换源，参考https://blog.csdn.net/watermelon1123/article/details/88122020
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main/
conda config --set show_channel_urls yes

#安装torch，命令在 https://pytorch.org/ 寻找
conda install pytorch torchvision torchaudio pytorch-cuda=12.1 -c pytorch-nightly -c nvidia

# ~/.bashrc
conda activate torch-gpu
```

**安装cuDNN：<https://developer.nvidia.com/cudnn-downloads>**

```bash
wget https://developer.download.nvidia.com/compute/cudnn/9.2.0/local_installers/cudnn-local-repo-ubuntu2004-9.2.0_1.0-1_amd64.deb
sudo dpkg -i cudnn-local-repo-ubuntu2004-9.2.0_1.0-1_amd64.deb
sudo cp /var/cudnn-local-repo-ubuntu2004-9.2.0/cudnn-*-keyring.gpg /usr/share/keyrings/
sudo apt-get update
sudo apt-get -y install cudnn-cuda-12

# 查看版本
cat /usr/include/cudnn_version.h | grep CUDNN_MAJOR -A 2
```

{% endfold %}

## pytorch book vscode 环境配置

{% fold @步骤 %}

```bash
git clone git@github.com:chenyuntc/pytorch-book.git
```

VScode安装插件

![](/img/ai/torch-book-extension.png)

此时打开任意一个note可以看到代码变成可以执行的框了：

![](/img/ai/vscode-load-python-1.png)

但是会显示torch未导入，点击左侧的执行三角形按钮，会提示选择安装必要的插件，安装完成后再次点击，会提示选择python版本

![](/img/ai/vscode-load-python-11.png)

选择python环境，找到conda路径下的python解释器

![](/img/ai/vscode-load-python-111.png)

再次点击左侧运行，弹出要安装pykernel包，点击安装完成后，可以在右上角看到环境和python版本，也可以点击此处继续更换环境。

{% endfold %}

## 示例

### 线性回归

梯度下降算法和auto grad自动求解梯度函数，参考 [**《深度学习框架PyTorch：入门与实战》代码**](https://github.com/chenyuntc/pytorch-book) 小试牛刀: 用autograd实现线性回归。

### fashion mnist 手写数字分类

参考Pytorch教程: [Quick Start](https://pytorch.org/tutorials/beginner/basics/quickstart_tutorial.html)

{% fold @训练步骤 %}

1. 下载数据集
2. 数据预处理等[Data Tutorial](https://pytorch.org/tutorials/beginner/basics/data_tutorial.html)
3. 定义model，类型、层、前向函数和损失计算函数，并传递至device，如CPU或者CUDA，[Build Model](https://pytorch.org/tutorials/beginner/basics/buildmodel_tutorial.html)
   1. [Sequential](https://pytorch.org/docs/stable/generated/torch.nn.Sequential.html)，构建连续的层[pytorch系列 nn.Sequential讲解](https://blog.csdn.net/dss_dssssd/article/details/82980222)
   2. 一些层级的简单介绍，[nn 网络层：池化层、线性层和激活函数层](https://yey.world/2020/12/16/Pytorch-13/)
      1. [ReLU](https://pytorch.org/docs/stable/generated/torch.nn.ReLU.html)
      2. [Linear](https://pytorch.org/docs/stable/generated/torch.nn.Linear.html)
4. 优化损失计算，[Optimization Tutorials](https://pytorch.org/tutorials/beginner/basics/optimization_tutorial.html)
   1. [交叉熵损失Cross Entropy Loss](https://pytorch.org/docs/stable/generated/torch.nn.CrossEntropyLoss.html)，参考[交叉熵损失函数原理及Pytorch代码简介](https://blog.csdn.net/chao_shine/article/details/89925762)
   2. [Pytorch中常用的四种优化器SGD、Momentum、RMSProp、Adam](https://zhuanlan.zhihu.com/p/78622301)
      1. [SGD](https://pytorch.org/docs/stable/generated/torch.optim.SGD.html)
5. 训练，导入数据，计算损失，调用前向函数
6. 测试，对测试集预测，计算预测结果争取率
7. 保存模型，[Save & Load & Run](https://pytorch.org/tutorials/beginner/basics/saveloadrun_tutorial.html)

{% endfold %}

{% fold @测试模型 %}

```python
import torch
from torch import nn
from torchvision import datasets
from torchvision.transforms import ToTensor

test_data = datasets.FashionMNIST(
    root="data",
    train=False,
    download=True,
    transform=ToTensor(),
)

device = (
    "cuda"
    if torch.cuda.is_available()
    else "mps"
    if torch.backends.mps.is_available()
    else "cpu"
)
print(f"Using {device} device")


class NeuralNetwork(nn.Module):
    def __init__(self):
        super().__init__()
        self.flatten = nn.Flatten()
        self.linear_relu_stack = nn.Sequential(
            nn.Linear(28*28, 512),
            nn.ReLU(),
            nn.Linear(512, 512),
            nn.ReLU(),
            nn.Linear(512, 10)
        )

    def forward(self, x):
        x = self.flatten(x)
        logits = self.linear_relu_stack(x)
        return logits

model = NeuralNetwork().to(device)
model.load_state_dict(torch.load("model.pth"))


classes = [
    "T-shirt/top",
    "Trouser",
    "Pullover",
    "Dress",
    "Coat",
    "Sandal",
    "Shirt",
    "Sneaker",
    "Bag",
    "Ankle boot",
]

model.eval()

x, y = test_data[1][0], test_data[1][1]
with torch.no_grad():
    x = x.to(device)
    pred = model(x)
    predicted, actual = classes[pred[0].argmax(0)], classes[y]
    print(f'Predicted: "{predicted}", Actual: "{actual}"')
```

{% endfold %}

### cifar 10

参考 [**《深度学习框架PyTorch：入门与实战》代码**](https://github.com/chenyuntc/pytorch-book) 2.2.4 小试牛刀：CIFAR-10分类。
