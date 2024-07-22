---
title: Karpathy llm.c测试与代码分析
index_img: /img/post_pics/gpu/gpt2.png
date: 2024-04-28 10:07:29
tags:
    - 算法
    - AI
categories: 
    - GPU
---

仓库地址: [llm.c](https://github.com/karpathy/llm.c)
模型镜像: [HF-Mirror](https://hf-mirror.com/)

<!-- more -->

## 参考

[如何理解Nvidia英伟达的Multi-GPU多卡通信框架NCCL？](https://www.zhihu.com/question/63219175)
[GPT（三）GPT2原理和代码详解](https://zhuanlan.zhihu.com/p/637782385)
[GPT （一）transformer原理和代码详解](https://zhuanlan.zhihu.com/p/632880248)
[GPT-2通俗详解](https://www.cnblogs.com/zhongzhaoxie/p/13064404.html)
[详细理解GPT2模型结构及其训练过程—GPT系列训练与部署](https://blog.csdn.net/suiyingy/article/details/130937792)
[NVIDIA Developer Tools](https://developer.nvidia.com/tools-overview)
[llm.c代码详细解读（一）](https://zhuanlan.zhihu.com/p/692116370)
[C语言写的LLM训练](https://blog.csdn.net/eidolon_foot/article/details/138474675)

## 安装NCCL

[nccl-download](https://developer.nvidia.com/nccl/nccl-download)
[Source Download](https://developer.nvidia.com/downloads/compute/machine-learning/nccl/secure/2.21.5/agnostic/ppc64le/nccl_2.21.5-1+cuda12.2_ppc64le.txz)

```bash
# Network Installer for Ubuntu20.04
$ wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2004/x86_64/cuda-keyring_1.0-1_all.deb
$ sudo dpkg -i cuda-keyring_1.0-1_all.deb
$ sudo apt-get update
```

## 测试对比

|<span style="display:inline-block;width:300px">Platform</span>|<span style="display:inline-block;width:250px">Time</span>|<span style="display:inline-block;width:250px">Loss</span>|
|-----------|------------|------|
| **4070S**     | 198.34 ms  | 3.48 |
| **H100**      | 36.90 ms   | 3.52 |
| **2 H100**    | 30.94 ms   | 3.70 |
| **Intel CPU** | 2635.72 ms | 4.02 |
| **Arm CPU**   | 12103.80 ms| 4.02 |
| **M1 CPU**    | 8128.85 ms | 4.02 |

## 使用GPU训练

```bash
pip install -r requirements.txt
sudo apt install openmpi-bin openmpi-doc libopenmpi-dev

python prepro_tinyshakespeare.py
python train_gpt2.py
make train_gpt2fp32cu
./train_gpt2fp32cu
```

<details>
<summary>RTX4070S LOG</summary>

```txt
+-----------------------+----------------------------------------------------+
| Parameter             | Value                                              |
+-----------------------+----------------------------------------------------+
| train data pattern    | dev/data/tinyshakespeare/tiny_shakespeare_train.bin |
| val data pattern      | dev/data/tinyshakespeare/tiny_shakespeare_val.bin  |
| output log file       | NULL                                               |
| batch size B          | 4                                                  |
| sequence length T     | 1024                                               |
| learning rate         | 0.000300                                           |
| val_loss_every        | 20                                                 |
| val_max_steps         | 20                                                 |
| sample_every          | 20                                                 |
| genT                  | 64                                                 |
+-----------------------+----------------------------------------------------+
| device                | NVIDIA GeForce RTX 4070 SUPER                      |
| TF32                  | enabled                                            |
+-----------------------+----------------------------------------------------+
| max_sequence_length T | 1024                                               |
| vocab_size V          | 50257                                              |
| padded_vocab_size Vp  | 50304                                              |
| num_layers L          | 12                                                 |
| num_heads NH          | 12                                                 |
| channels C            | 768                                                |
| num_parameters        | 124475904                                          |
+-----------------------+----------------------------------------------------+
| train_num_batches     | 74                                                 |
| val_num_batches       | 8                                                  |
+-----------------------+----------------------------------------------------+
allocated 474 MiB for model parameters
allocated 5706 MiB for activations
val loss 4.508645
allocated 474 MiB for parameter gradients
allocated 252 MiB for activation gradients
allocated 474 MiB for AdamW optimizer state m
allocated 474 MiB for AdamW optimizer state v
step    1/74: train loss 4.287312 (216.329368 ms, 18934 tok/s)
step    2/74: train loss 4.725657 (196.385098 ms, 20856 tok/s)
step    3/74: train loss 4.393068 (195.683672 ms, 20931 tok/s)
step    4/74: train loss 3.628445 (196.121049 ms, 20885 tok/s)
step    5/74: train loss 3.717055 (194.049300 ms, 21108 tok/s)
step    6/74: train loss 4.013161 (197.751605 ms, 20712 tok/s)
step    7/74: train loss 4.111417 (195.673638 ms, 20932 tok/s)
step    8/74: train loss 3.623600 (199.146686 ms, 20567 tok/s)
step    9/74: train loss 3.965506 (196.338315 ms, 20861 tok/s)
step   10/74: train loss 3.581475 (195.944831 ms, 20903 tok/s)
step   11/74: train loss 4.028997 (197.172886 ms, 20773 tok/s)
step   12/74: train loss 3.813888 (195.415474 ms, 20960 tok/s)
step   13/74: train loss 4.029539 (195.721443 ms, 20927 tok/s)
step   14/74: train loss 3.885392 (196.801713 ms, 20812 tok/s)
step   15/74: train loss 3.711839 (196.184539 ms, 20878 tok/s)
step   16/74: train loss 3.553492 (195.104225 ms, 20993 tok/s)
step   17/74: train loss 3.988450 (196.938082 ms, 20798 tok/s)
step   18/74: train loss 3.809328 (195.232320 ms, 20980 tok/s)
step   19/74: train loss 3.534165 (196.839770 ms, 20808 tok/s)
step   20/74: train loss 3.678106 (196.428516 ms, 20852 tok/s)
val loss 3.705425
generating:
---
O, but Vanlin, you no less light the cell of our men,
Look to thy light! How much an eye will kiss the seat, save
staying and:
That way unsayers

<|endoftext|>IETERABETH:
Your good shrewescy and
to thy fingers tailorate
---
step   21/74: train loss 3.608904 (198.159644 ms, 20670 tok/s)
step   22/74: train loss 3.611267 (197.791646 ms, 20708 tok/s)
step   23/74: train loss 3.146728 (197.174480 ms, 20773 tok/s)
step   24/74: train loss 3.344270 (196.746858 ms, 20818 tok/s)
step   25/74: train loss 3.707907 (198.275599 ms, 20658 tok/s)
step   26/74: train loss 3.539583 (203.417195 ms, 20135 tok/s)
step   27/74: train loss 3.844320 (200.666546 ms, 20411 tok/s)
step   28/74: train loss 3.445682 (197.969591 ms, 20690 tok/s)
step   29/74: train loss 3.318048 (196.887630 ms, 20803 tok/s)
step   30/74: train loss 3.279650 (197.256815 ms, 20764 tok/s)
step   31/74: train loss 3.154556 (198.877454 ms, 20595 tok/s)
step   32/74: train loss 3.444809 (200.677574 ms, 20410 tok/s)
step   33/74: train loss 3.566215 (199.827710 ms, 20497 tok/s)
step   34/74: train loss 3.487424 (197.150262 ms, 20776 tok/s)
step   35/74: train loss 3.722091 (199.232841 ms, 20558 tok/s)
step   36/74: train loss 3.380269 (198.515393 ms, 20633 tok/s)
step   37/74: train loss 3.199764 (198.736697 ms, 20610 tok/s)
step   38/74: train loss 3.179839 (198.228997 ms, 20662 tok/s)
step   39/74: train loss 3.601362 (199.353707 ms, 20546 tok/s)
step   40/74: train loss 3.083126 (202.057997 ms, 20271 tok/s)
val loss 3.582962
generating:
---
Lightaxe hunt,
USA, sir!
I come to make a brief knightle
The fool his sorceries
In wounding the good and that of this fellowship.
Intence remains unwholesome to be run,
Like the one that is ere the countenance is waked.
---
step   41/74: train loss 3.830101 (199.004444 ms, 20582 tok/s)
step   42/74: train loss 3.362655 (199.871726 ms, 20493 tok/s)
step   43/74: train loss 3.355859 (198.784472 ms, 20605 tok/s)
step   44/74: train loss 3.418773 (198.443265 ms, 20640 tok/s)
step   45/74: train loss 3.608499 (198.736102 ms, 20610 tok/s)
step   46/74: train loss 3.128481 (199.459168 ms, 20535 tok/s)
step   47/74: train loss 3.464015 (196.227803 ms, 20873 tok/s)
step   48/74: train loss 3.975485 (197.719352 ms, 20716 tok/s)
step   49/74: train loss 3.224785 (197.451708 ms, 20744 tok/s)
step   50/74: train loss 3.417883 (195.725265 ms, 20927 tok/s)
step   51/74: train loss 3.761238 (197.567030 ms, 20732 tok/s)
step   52/74: train loss 3.769763 (197.559271 ms, 20733 tok/s)
step   53/74: train loss 3.410849 (197.411474 ms, 20748 tok/s)
step   54/74: train loss 3.389037 (197.022081 ms, 20789 tok/s)
step   55/74: train loss 3.274219 (196.771373 ms, 20816 tok/s)
step   56/74: train loss 3.260659 (197.592729 ms, 20729 tok/s)
step   57/74: train loss 3.028346 (197.423359 ms, 20747 tok/s)
step   58/74: train loss 3.479192 (196.417625 ms, 20853 tok/s)
step   59/74: train loss 3.271112 (198.220795 ms, 20663 tok/s)
step   60/74: train loss 3.496041 (196.792773 ms, 20813 tok/s)
val loss 3.490089
generating:
---

Ladies and gentlemen,
I with these ancient deaths,

<|endoftext|>LEONTES:
Now I am a saint.
Hear: Prevento,
Verona Oremo! Thy love center is
Would take her.
More than that one fellow who descends from heaven at capone
---
step   61/74: train loss 3.211886 (200.864481 ms, 20391 tok/s)
step   62/74: train loss 3.451102 (202.447595 ms, 20232 tok/s)
step   63/74: train loss 3.371627 (202.120002 ms, 20265 tok/s)
step   64/74: train loss 3.408307 (203.412583 ms, 20136 tok/s)
step   65/74: train loss 3.579690 (197.981410 ms, 20688 tok/s)
step   66/74: train loss 3.029516 (201.803798 ms, 20296 tok/s)
step   67/74: train loss 3.296247 (198.256757 ms, 20660 tok/s)
step   68/74: train loss 3.676572 (197.812296 ms, 20706 tok/s)
step   69/74: train loss 3.297943 (197.705935 ms, 20717 tok/s)
step   70/74: train loss 3.647188 (199.209822 ms, 20561 tok/s)
step   71/74: train loss 3.566411 (198.794319 ms, 20604 tok/s)
step   72/74: train loss 3.731066 (199.354373 ms, 20546 tok/s)
step   73/74: train loss 3.825200 (199.243927 ms, 20557 tok/s)
step   74/74: train loss 3.380793 (201.802198 ms, 20297 tok/s)
val loss 3.475538
generating:
---
BUCKINGHAM:
But in my heart his fiendy rascal,
By silky hand, shouted more satisfied,
Than to my breath with long informal desperate quips
Your brother alike sir.
And brace me not that with his woes:
Never to sit or play this
---
total average iteration time: 198.341601 ms

```
</details>

<details>
<summary>H100 LOG</summary>

```txt
+-----------------------+----------------------------------------------------+
| Parameter             | Value                                              |
+-----------------------+----------------------------------------------------+
| input dataset prefix  | data/tiny_shakespeare                              |
| output log file       | NULL                                               |
| batch size B          | 4                                                  |
| sequence length T     | 1024                                               |
| learning rate         | 0.000300                                           |
| val_loss_every        | 20                                                 |
| val_max_batches       | 20                                                 |
| sample_every          | 20                                                 |
| genT                  | 64                                                 |
+-----------------------+----------------------------------------------------+
| device                | NVIDIA H800                                        |
| TF32                  | enabled                                            |
+-----------------------+----------------------------------------------------+
| max_sequence_length T | 1024                                               |
| vocab_size V          | 50257                                              |
| num_layers L          | 12                                                 |
| num_heads NH          | 12                                                 |
| channels C            | 768                                                |
| num_parameters        | 124439808                                          |
+-----------------------+----------------------------------------------------+
| train_num_batches     | 74                                                 |
| val_num_batches       | 20                                                 |
+-----------------------+----------------------------------------------------+
allocated 474 MiB for model parameters
allocated 5706 MiB for activations
val loss 4.506288
allocated 474 MiB for parameter gradients
allocated 252 MiB for activation gradients
allocated 474 MiB for AdamW optimizer state m
allocated 474 MiB for AdamW optimizer state v
step    1/74: train loss 4.367558 (42.291931 ms, 96850 tok/s)
step    2/74: train loss 4.435496 (36.873423 ms, 111082 tok/s)
step    3/74: train loss 4.346745 (36.917661 ms, 110949 tok/s)
step    4/74: train loss 3.916155 (36.930912 ms, 110909 tok/s)
step    5/74: train loss 3.576688 (36.882907 ms, 111054 tok/s)
step    6/74: train loss 3.752822 (36.804162 ms, 111291 tok/s)
step    7/74: train loss 3.543940 (36.823446 ms, 111233 tok/s)
step    8/74: train loss 3.691794 (36.872353 ms, 111085 tok/s)
step    9/74: train loss 3.292197 (36.917963 ms, 110948 tok/s)
step   10/74: train loss 3.420270 (36.912048 ms, 110966 tok/s)
step   11/74: train loss 3.839151 (36.832944 ms, 111204 tok/s)
step   12/74: train loss 3.459940 (36.855074 ms, 111138 tok/s)
step   13/74: train loss 3.617168 (36.834898 ms, 111198 tok/s)
step   14/74: train loss 3.230776 (36.871742 ms, 111087 tok/s)
step   15/74: train loss 3.669937 (36.886197 ms, 111044 tok/s)
step   16/74: train loss 3.859764 (36.910485 ms, 110971 tok/s)
step   17/74: train loss 3.851466 (36.913714 ms, 110961 tok/s)
step   18/74: train loss 3.920131 (36.908567 ms, 110976 tok/s)
step   19/74: train loss 3.639019 (36.922885 ms, 110933 tok/s)
step   20/74: train loss 3.733764 (36.988406 ms, 110737 tok/s)
val loss 3.687801
generating:
---
O, my cousin: that is so.

<|endoftext|>O<|endoftext|>Trussell, thy father's son, son of the Roman king Hardsley, heir<|endoftext|>LUTHER, for whom shall Scotland draw her throne, except for England?
In England, consulate marcius:
And bind together Egbert
---
step   21/74: train loss 3.719804 (36.868217 ms, 111098 tok/s)
step   22/74: train loss 3.586144 (36.921155 ms, 110939 tok/s)
step   23/74: train loss 3.551655 (36.905093 ms, 110987 tok/s)
step   24/74: train loss 3.351520 (36.939467 ms, 110884 tok/s)
step   25/74: train loss 3.454527 (36.997209 ms, 110711 tok/s)
step   26/74: train loss 3.761025 (36.983948 ms, 110750 tok/s)
step   27/74: train loss 3.779032 (36.952961 ms, 110843 tok/s)
step   28/74: train loss 3.636410 (36.875919 ms, 111075 tok/s)
step   29/74: train loss 3.448576 (36.891939 ms, 111026 tok/s)
step   30/74: train loss 3.574333 (36.936213 ms, 110893 tok/s)
step   31/74: train loss 3.509148 (36.920664 ms, 110940 tok/s)
step   32/74: train loss 3.362097 (36.908129 ms, 110978 tok/s)
step   33/74: train loss 3.421195 (36.975713 ms, 110775 tok/s)
step   34/74: train loss 3.684764 (36.974622 ms, 110778 tok/s)
step   35/74: train loss 3.381419 (36.919138 ms, 110945 tok/s)
step   36/74: train loss 3.401418 (36.891895 ms, 111027 tok/s)
step   37/74: train loss 3.812751 (36.899238 ms, 111005 tok/s)
step   38/74: train loss 3.623131 (36.911724 ms, 110967 tok/s)
step   39/74: train loss 3.489853 (36.926676 ms, 110922 tok/s)
step   40/74: train loss 3.137516 (36.902834 ms, 110994 tok/s)
val loss 3.635424
generating:
---
Diademorns,
God, thou wilt be king:
It's busy with summer, like day before.
Is it sorrows, is it stories, Waters, and naked lamentations?
Were they such a office, for marriage?
Who wilt party the day chronicling,
And
---
step   41/74: train loss 3.476893 (36.845553 ms, 111166 tok/s)
step   42/74: train loss 3.330724 (36.863462 ms, 111112 tok/s)
step   43/74: train loss 3.477123 (36.838678 ms, 111187 tok/s)
step   44/74: train loss 3.366669 (36.889697 ms, 111033 tok/s)
step   45/74: train loss 3.979407 (36.901669 ms, 110997 tok/s)
step   46/74: train loss 3.866721 (36.905043 ms, 110987 tok/s)
step   47/74: train loss 3.774495 (36.912187 ms, 110966 tok/s)
step   48/74: train loss 3.962839 (36.842677 ms, 111175 tok/s)
step   49/74: train loss 4.036259 (36.851610 ms, 111148 tok/s)
step   50/74: train loss 3.857388 (36.853193 ms, 111143 tok/s)
step   51/74: train loss 3.604754 (36.845790 ms, 111166 tok/s)
step   52/74: train loss 3.579455 (36.831378 ms, 111209 tok/s)
step   53/74: train loss 3.824139 (36.857464 ms, 111130 tok/s)
step   54/74: train loss 3.766292 (36.894130 ms, 111020 tok/s)
step   55/74: train loss 3.487747 (36.934630 ms, 110898 tok/s)
step   56/74: train loss 3.151821 (36.968007 ms, 110798 tok/s)
step   57/74: train loss 3.344814 (36.882516 ms, 111055 tok/s)
step   58/74: train loss 3.522471 (36.902438 ms, 110995 tok/s)
step   59/74: train loss 3.373972 (37.128153 ms, 110320 tok/s)
step   60/74: train loss 3.433309 (36.883408 ms, 111052 tok/s)
val loss 3.529820
generating:
---
01:<|endoftext|>Look, I am an o'erthompson by blood, I can live with my blood, saying to the ford, thou liest! That doeth think, in Wynldota's my noble soul;
I'll do it here for York: and though thy lordship do
---
step   61/74: train loss 3.323022 (36.834574 ms, 111199 tok/s)
step   62/74: train loss 3.263776 (36.836349 ms, 111194 tok/s)
step   63/74: train loss 3.288344 (36.908309 ms, 110977 tok/s)
step   64/74: train loss 3.792862 (36.868552 ms, 111097 tok/s)
step   65/74: train loss 3.561376 (36.875281 ms, 111077 tok/s)
step   66/74: train loss 3.339633 (36.894187 ms, 111020 tok/s)
step   67/74: train loss 3.232719 (36.929932 ms, 110912 tok/s)
step   68/74: train loss 3.424346 (36.951805 ms, 110847 tok/s)
step   69/74: train loss 3.259733 (36.961079 ms, 110819 tok/s)
step   70/74: train loss 3.071201 (36.967520 ms, 110799 tok/s)
step   71/74: train loss 3.048391 (36.890885 ms, 111030 tok/s)
step   72/74: train loss 3.058076 (36.883778 ms, 111051 tok/s)
step   73/74: train loss 3.697609 (36.878680 ms, 111066 tok/s)
step   74/74: train loss 3.497812 (36.910939 ms, 110969 tok/s)
val loss 3.515306
generating:
---
A faint noise was heard in the air. Gentlemen, I have fairly said, than should be the report he shall hear, and you'll that have only no prison made for you.

<|endoftext|>RICHARD VINCENTIO:
Why, how you sounded.

<|endoftext|>CAM<|endoftext|>K AS
---
total average iteration time: 36.974027 ms
```

</details>

两张卡，注意要配置nccl

```bash
sudo apt install openmpi-bin openmpi-doc libopenmpi-dev
pip install -r requirements.txt
python prepro_tinyshakespeare.py
python train_gpt2.py
make train_gpt2cu
mpirun -np <number of GPUs on your machine> ./train_gpt2cu
```

<details>
<summary>2 H100 LOG</summary>

```txt
mpirun -np 2 ./train_gpt2cu
+-----------------------+----------------------------------------------------+
| Parameter             | Value                                              |
+-----------------------+----------------------------------------------------+
| input dataset prefix  | data/tiny_shakespeare                              |
| output log file       | NULL                                               |
| batch size B          | 4                                                  |
| sequence length T     | 1024                                               |
| learning rate         | 0.000300                                           |
| val_loss_every        | 20                                                 |
| val_max_batches       | 20                                                 |
| sample_every          | 20                                                 |
| genT                  | 64                                                 |
+-----------------------+----------------------------------------------------+
| device                | NVIDIA H800                                        |
| TF32                  | enabled                                            |
+-----------------------+----------------------------------------------------+
| max_sequence_length T | 1024                                               |
| vocab_size V          | 50257                                              |
| num_layers L          | 12                                                 |
| num_heads NH          | 12                                                 |
| channels C            | 768                                                |
| num_parameters        | 124439808                                          |
+-----------------------+----------------------------------------------------+
| train_num_batches     | 37                                                 |
| val_num_batches       | 20                                                 |
+-----------------------+----------------------------------------------------+
| num_processes         | 2                                                  |
+-----------------------+----------------------------------------------------+
num_parameters: 124439808 ==> bytes: 248956416
allocated 237 MiB for model parameters
allocated 2853 MiB for activations
val loss -inf
val loss -inf
allocated 237 MiB for parameter gradients
allocated 126 MiB for activation gradients
allocated 474 MiB for AdamW optimizer state m
allocated 474 MiB for AdamW optimizer state v
step    1/37: train loss -inf (acc -inf) (34.738357 ms, 235820 tok/s)
step    2/37: train loss 4.705958 (acc 4.676888) (31.091222 ms, 263482 tok/s)
step    3/37: train loss 3.801152 (acc -inf) (30.325266 ms, 270137 tok/s)
step    4/37: train loss 3.712616 (acc 3.812249) (30.541328 ms, 268226 tok/s)
step    5/37: train loss 3.442210 (acc 3.527371) (30.376322 ms, 269683 tok/s)
step    6/37: train loss 3.959154 (acc 3.777182) (30.439979 ms, 269119 tok/s)
step    7/37: train loss 3.730492 (acc 3.543638) (30.784793 ms, 266105 tok/s)
step    8/37: train loss 3.780181 (acc 3.866994) (30.837841 ms, 265647 tok/s)
step    9/37: train loss 3.943718 (acc 4.001076) (30.502230 ms, 268570 tok/s)
step   10/37: train loss 3.712828 (acc 3.797800) (30.952566 ms, 264663 tok/s)
step   11/37: train loss 3.848890 (acc 3.785396) (31.113195 ms, 263296 tok/s)
step   12/37: train loss 3.666040 (acc 3.574438) (30.766566 ms, 266263 tok/s)
step   13/37: train loss 3.543876 (acc 3.707925) (30.671046 ms, 267092 tok/s)
step   14/37: train loss 3.869239 (acc 3.814487) (30.945898 ms, 264720 tok/s)
step   15/37: train loss 3.563170 (acc 3.632645) (30.741986 ms, 266475 tok/s)
step   16/37: train loss 3.596956 (acc 3.518488) (31.099593 ms, 263411 tok/s)
step   17/37: train loss 3.525343 (acc 3.662786) (31.034419 ms, 263964 tok/s)
step   18/37: train loss 3.423222 (acc 3.451016) (30.746931 ms, 266433 tok/s)
step   19/37: train loss 3.886096 (acc 3.810491) (31.060373 ms, 263744 tok/s)
step   20/37: train loss 3.584194 (acc 3.401700) (31.060387 ms, 263744 tok/s)
val loss 3.702823
val loss 3.702823
generating:
---
O, which mighty holy body and armow day with love in good time:
Are you wise to the garden of the Cyprian?
O<|endoftext|>Farewell,
O fighters, brothers: the dragons
Join by fallen lightning,
FIelwine unmindful, and grieflike weeping:
---
step   21/37: train loss 3.541171 (acc 3.480718) (30.991152 ms, 264333 tok/s)
step   22/37: train loss 3.572929 (acc 3.517702) (30.916142 ms, 264974 tok/s)
step   23/37: train loss 4.025887 (acc 3.984698) (31.284707 ms, 261853 tok/s)
step   24/37: train loss 3.851679 (acc 3.963296) (31.397653 ms, 260911 tok/s)
step   25/37: train loss 4.115643 (acc 4.023971) (30.779578 ms, 266150 tok/s)
step   26/37: train loss 3.674400 (acc 3.667000) (30.940606 ms, 264765 tok/s)
step   27/37: train loss 3.915539 (acc 3.891176) (31.033721 ms, 263970 tok/s)
step   28/37: train loss 3.526467 (acc 3.364087) (30.914958 ms, 264984 tok/s)
step   29/37: train loss 3.406913 (acc 3.516932) (31.026487 ms, 264032 tok/s)
step   30/37: train loss 3.427009 (acc 3.471455) (30.607531 ms, 267646 tok/s)
step   31/37: train loss 3.387648 (acc 3.368385) (30.865644 ms, 265408 tok/s)
step   32/37: train loss 3.399536 (acc 3.620977) (30.517480 ms, 268436 tok/s)
step   33/37: train loss 3.621998 (acc 3.517930) (30.633715 ms, 267417 tok/s)
step   34/37: train loss 3.306428 (acc 3.412587) (30.562156 ms, 268043 tok/s)
step   35/37: train loss 3.304569 (acc 3.233234) (30.509134 ms, 268509 tok/s)
step   36/37: train loss 3.127993 (acc 3.148825) (30.534061 ms, 268290 tok/s)
step   37/37: train loss 3.707109 (acc 3.523119) (31.372644 ms, 261119 tok/s)
val loss inf
generating:
---
val loss inf
Come elephantaman!
Come a competed-fire, my lord,
So jungle it does;

<|endoftext|>CLEARINGEN
Ay, thine not to this owl, to such climbing beast
As tame me and as white-robed;
And in an ancient and impotent manner, to any
---

total average iteration time: 30.938315 ms

```

</details> 

## 使用CPU训练

```bash
pip install -r requirements.txt
python prepro_tinyshakespeare.py
python train_gpt2.py
make train_gpt2
OMP_NUM_THREADS=8 ./train_gpt2
```

<details>
<summary>Intel CPU log</summary>

```txt
OMP_NUM_THREADS=8 ./train_gpt2
[GPT-2]
max_seq_len: 1024
vocab_size: 50257
num_layers: 12
num_heads: 12
channels: 768
num_parameters: 124439808
train dataset num_batches: 1192
val dataset num_batches: 128
num_activations: 73323776
val loss 5.325522
step 0: train loss 5.356185 (took 3682.965439 ms)
step 1: train loss 4.301033 (took 3351.367950 ms)
step 2: train loss 4.623316 (took 2785.125469 ms)
step 3: train loss 4.600415 (took 2972.771599 ms)
step 4: train loss 4.616777 (took 2991.414681 ms)
step 5: train loss 4.231482 (took 3310.190622 ms)
step 6: train loss 3.754166 (took 2839.633690 ms)
step 7: train loss 3.652230 (took 3017.137520 ms)
step 8: train loss 4.183515 (took 2826.837910 ms)
step 9: train loss 4.199315 (took 2691.491697 ms)
val loss 4.323445
step 10: train loss 4.288396 (took 2605.089097 ms)
step 11: train loss 3.558984 (took 2799.268435 ms)
step 12: train loss 3.730804 (took 2951.344529 ms)
step 13: train loss 4.159164 (took 2675.097650 ms)
step 14: train loss 3.886458 (took 2578.160226 ms)
step 15: train loss 3.764933 (took 2580.616568 ms)
step 16: train loss 4.143034 (took 2726.701810 ms)
step 17: train loss 3.962718 (took 2708.172316 ms)
step 18: train loss 3.796120 (took 2617.401951 ms)
step 19: train loss 3.371638 (took 2590.776841 ms)
val loss 4.186637
generating:
---
I was so exceptionally drunk:
You would spake for seen'st
Threaten beyond you, 'twas so far too?

<|endoftext|>STRUCTINIUS:
Scheduled since 1539 is
Welcome to Rome:
We meet the indignation of lesser nations' countenance? thank me
---
step 20: train loss 3.880942 (took 3111.667706 ms)
step 21: train loss 4.198619 (took 2580.162790 ms)
step 22: train loss 4.426098 (took 2578.943997 ms)
step 23: train loss 3.685762 (took 2580.436384 ms)
step 24: train loss 3.642307 (took 2623.821726 ms)
step 25: train loss 3.729648 (took 2580.534433 ms)
step 26: train loss 3.549645 (took 2630.321238 ms)
step 27: train loss 3.339360 (took 2578.629251 ms)
step 28: train loss 4.338965 (took 2627.450328 ms)
step 29: train loss 3.812843 (took 2579.089464 ms)
val loss 4.020430
step 30: train loss 4.028022 (took 2632.939240 ms)
step 31: train loss 4.114379 (took 2594.114480 ms)
step 32: train loss 3.575101 (took 2635.722184 ms)
step 33: train loss 4.366093 (took 2576.848847 ms)
step 34: train loss 4.516504 (took 2587.968289 ms)
step 35: train loss 4.434158 (took 2576.720293 ms)
step 36: train loss 4.097423 (took 2609.961644 ms)
step 37: train loss 3.739693 (took 2579.032073 ms)
step 38: train loss 4.612139 (took 2578.933900 ms)
step 39: train loss 3.970823 (took 2619.630001 ms)
val loss 4.016672
generating:
---
Come Kurultan,
Among the geopolitical
Coers and My scullers take one word.

<|endoftext|>Shutth out of the yacht,
Sone of dejected glories
draw'd like an everlasting flame;
But: prying out, as a look in a good canopy
Fairs with
---
step 40: train loss 4.377796 (took 2961.353965 ms)
```

</details>

<details>
<summary>Arm CPU log</summary>

```txt
[GPT-2]
max_seq_len: 1024
vocab_size: 50257
num_layers: 12
num_heads: 12
channels: 768
num_parameters: 124439808
train dataset num_batches: 1192
val dataset num_batches: 128
num_activations: 73323776
val loss 5.325415
step 0: train loss 5.356085 (took 12606.966341 ms)
step 1: train loss 4.300643 (took 12219.476918 ms)
step 2: train loss 4.623083 (took 12207.496260 ms)
step 3: train loss 4.599365 (took 12148.045433 ms)
step 4: train loss 4.616661 (took 12401.153808 ms)
step 5: train loss 4.231428 (took 12116.966958 ms)
step 6: train loss 3.753162 (took 12224.808617 ms)
step 7: train loss 3.650456 (took 12192.318357 ms)
step 8: train loss 4.182243 (took 12269.556216 ms)
step 9: train loss 4.199580 (took 12101.010577 ms)
val loss 4.323766
step 10: train loss 4.288661 (took 12230.598978 ms)
step 11: train loss 3.560643 (took 12231.405416 ms)
step 12: train loss 3.731442 (took 12093.194701 ms)
step 13: train loss 4.158509 (took 12191.899776 ms)
step 14: train loss 3.885638 (took 12142.622528 ms)
step 15: train loss 3.766486 (took 12458.911155 ms)
step 16: train loss 4.144007 (took 12300.180748 ms)
step 17: train loss 3.961168 (took 12232.756950 ms)
step 18: train loss 3.796045 (took 12046.576110 ms)
step 19: train loss 3.371045 (took 12077.808580 ms)
val loss 4.187855
generating:
---
I was so frightened with your face: to come and though they would not do it any more than as
Let us; but who ever can turn
Against a world so full,
That there'll have been none of our fightmen but
Weaver-bats and tearing men, and stir them utterly;
---
step 20: train loss 3.882792 (took 12222.624648 ms)
step 21: train loss 4.199980 (took 12317.641401 ms)
step 22: train loss 4.428427 (took 12266.228655 ms)
step 23: train loss 3.685924 (took 12392.576155 ms)
step 24: train loss 3.643298 (took 12075.278688 ms)
step 25: train loss 3.729694 (took 12030.850856 ms)
step 26: train loss 3.550648 (took 12212.194214 ms)
step 27: train loss 3.338631 (took 12221.727207 ms)
step 28: train loss 4.342020 (took 12070.876624 ms)
step 29: train loss 3.814729 (took 12146.005022 ms)
val loss 4.022702
step 30: train loss 4.032417 (took 12264.562893 ms)
step 31: train loss 4.118070 (took 12183.779377 ms)
step 32: train loss 3.577005 (took 12179.788109 ms)
step 33: train loss 4.369797 (took 12296.633846 ms)
step 34: train loss 4.524115 (took 12094.045750 ms)
step 35: train loss 4.438815 (took 12103.800767 ms)
step 36: train loss 4.101098 (took 12304.076168 ms)
step 37: train loss 3.740980 (took 12288.887784 ms)
step 38: train loss 4.618742 (took 12264.409146 ms)
step 39: train loss 3.972258 (took 12218.437701 ms)
val loss 4.017348
generating:
---
CLAUSE:
I cannot, sir, can; as I would
do @scended
da drawn breath
to love
Ferrante, the fourth Receiver: the king must leave this
matter for our own use,
who will
roll the first wine-tureen and
press the
---
step 40: train loss 4.378431 (took 12246.907331 ms)
```

</details>

<details>
<summary>M1 CPU log</summary>

```txt
[GPT-2]
max_seq_len: 1024
vocab_size: 50257
num_layers: 12
num_heads: 12
channels: 768
num_parameters: 124439808
train dataset num_batches: 1192
val dataset num_batches: 128
num_activations: 73323776
val loss 5.325529
step 0: train loss 5.356189 (took 8810.320000 ms)
step 1: train loss 4.301069 (took 10129.255000 ms)
step 2: train loss 4.623322 (took 9316.635000 ms)
step 3: train loss 4.600470 (took 9327.144000 ms)
step 4: train loss 4.616786 (took 9088.823000 ms)
step 5: train loss 4.231483 (took 8100.120000 ms)
step 6: train loss 3.754234 (took 7593.923000 ms)
step 7: train loss 3.652349 (took 7673.950000 ms)
step 8: train loss 4.183590 (took 7672.052000 ms)
step 9: train loss 4.199314 (took 7741.869000 ms)
val loss 4.323434
step 10: train loss 4.288389 (took 7588.665000 ms)
step 11: train loss 3.558898 (took 7701.314000 ms)
step 12: train loss 3.730761 (took 7512.813000 ms)
step 13: train loss 4.159196 (took 7499.602000 ms)
step 14: train loss 3.886509 (took 7493.241000 ms)
step 15: train loss 3.764848 (took 7683.579000 ms)
step 16: train loss 4.142992 (took 7532.287000 ms)
step 17: train loss 3.962821 (took 7699.390000 ms)
step 18: train loss 3.796132 (took 7782.585000 ms)
step 19: train loss 3.371673 (took 7662.459000 ms)
val loss 4.186563
generating:
---
I was so upright that I would have never heard you had any talk. I have heard you sometimes datts you, Eats could or should be choked, crows and
Fearsome snakes say it right.

<|endoftext|>Second Servingman:
I flagged him down in a half dozen ships AND don
---
step 20: train loss 3.880836 (took 10487.761000 ms)
step 21: train loss 4.198525 (took 8940.160000 ms)
step 22: train loss 4.425973 (took 8598.007000 ms)
step 23: train loss 3.685762 (took 8337.916000 ms)
step 24: train loss 3.642260 (took 8356.999000 ms)
step 25: train loss 3.729658 (took 8146.168000 ms)
step 26: train loss 3.549591 (took 8178.455000 ms)
step 27: train loss 3.339406 (took 8098.217000 ms)
step 28: train loss 4.338812 (took 8378.764000 ms)
step 29: train loss 3.812741 (took 8280.067000 ms)
val loss 4.020304
step 30: train loss 4.027764 (took 8294.253000 ms)
step 31: train loss 4.114197 (took 8132.258000 ms)
step 32: train loss 3.574986 (took 8328.408000 ms)
step 33: train loss 4.365894 (took 8154.654000 ms)
step 34: train loss 4.516072 (took 8327.388000 ms)
step 35: train loss 4.433900 (took 8041.963000 ms)
step 36: train loss 4.097214 (took 8346.843000 ms)
step 37: train loss 3.739647 (took 8232.633000 ms)
step 38: train loss 4.611735 (took 8103.667000 ms)
step 39: train loss 3.970751 (took 8065.254000 ms)
val loss 4.016658
generating:
---
Come Running Away,
Greater conquer
With the Imperial blood
the heaviest host of the gods
into this wondrous world beyond.
I will not back thee, for how sweet after birth
Netflix against repounder,
will not
flourish against the earlocks of
Allay
---
step 40: train loss 4.377756 (took 8128.859000 ms)
```

</details>
