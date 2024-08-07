---
layout: post
title: LLM 算法分析
index_img: /img/gpt/index.jpg
date: 2024-08-07 13:49:09
archive: false
tags:
    - AI
    - GPU
    - LLM
categories: 
    - GPU
---

一文了解 GPT 推理原理。

<!-- more -->

# AI演化趋势

AI模型规模越来越庞大，越来越简洁，人工参与度要求越来越低，和硬件发展相匹配。

- 2000年前百家齐放，主要环境PC机：
  - 专家系统、贝叶斯网络、[支持向量机](https://charlesliuyx.github.io/2017/09/19/%E6%94%AF%E6%8C%81%E5%90%91%E9%87%8F%E6%9C%BASVM%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/)、[决策树](https://blog.csdn.net/weixin_46323666/article/details/125843236)、神经网络、…
  - IBM深蓝(国际象棋), Doctor Watson

{% gi 4 3-1 %}
    ![专家系统](/img/gpt/expert.png)
    ![决策树](/img/gpt/tree.png)
    ![向量机](/img/gpt/svm.png)
    ![更深的蓝](/img/gpt/deepblue.jpg)
{% endgi %}

- 2000s深度学习兴起，和GPU适配
  - 1940s早期神经网络，1980s [CNN](https://yann.lecun.com/exdb/lenet/index.html)提出，1990s RNN&[LSTM](https://developer.aliyun.com/article/1112196)，复杂多变的网络结构和算子
    - [Lenet论文学习笔记](https://blog.csdn.net/hduxiejun/article/details/53571768)
  - 2006 NV CUDA框架， 2016 AlphaGo

{% gi 3 1-2 %}
    ![卷积神经网络](/img/gpt/cnn.png)
    ![贝叶斯长期短期记忆模型（LSTM）](/img/gpt/LSTM.png)
    ![阿法狗](/img/gpt/alphago.jpg)
{% endgi %}

- 2020s大语言模型兴起, 得益于GPU与AI相互促进发展
  - 2017 **Attention is all you need**
  - 2018 GPT/BERT
  - 2022 chatGPT
    - **矩阵乘，残差网络，LayerNormalization，Attention**
    - **Elementwise 非线性激活函数**
    - **基于 Transformer 结构**

![LLM 应用](/img/gpt/llmApp.png)

# 神经网络简介

![神经网络](/img/gpt/nn.png)

# 残差神经网络

解决退化问题

{% gi 2 2 %}
![Degradation Problem](/img/gpt/degradation-problem.png)
![多层残差神经网络](/img/gpt/resnet.png)
{% endgi %}

计算残差:

![算子](/img/gpt/res.png)

# 编码器与解码器

Seq2Seq : RNN, LSTM, Transformer

- [REAL TIMESPEECH TRANSLATION USING RECURRENT NEURAL NETWORKS](https://ir.vignan.ac.in/id/eprint/623/1/19.ANAND%20REAL%20TIMESPEECH%20TRANSLATION%20USING%20RNN.pdf)

![Encode & Decoder](/img/gpt/encdec.gif)

# LLM 分类

{% gi 2 2 %}
  ![Attention 架构](/img/gpt/attention.png)
  ![分类](/img/gpt/class.jpg)
{% endgi %}

# Encoder & Decoder功能对比

- Masked Language Model （BERT Training）
- Next Token Generation （GPT）：我爱北京天安，计算出下一个字 “门”

![bert](/img/gpt/bert.png)
