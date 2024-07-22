---
title: R329模型仿真测试
date: 2021-07-15 22:07:29
tags:
    - Linux
    - 算法
    - AI
categories: 
    - 边缘计算
---

## Docker安装及使用

参考这个帖子安装docker[在Ubuntu中安装Docker和docker的使用](https://www.cnblogs.com/blog-rui/p/11244023.html)
安装完毕后拉取环境并启动

<!-- more -->  

```bash
sudo docker pull zepan/zhouyi
sudo docker run -i -t zepan/zhouyi  /bin/bash
```

### 运行中保存

```bash
docker ps
docker commit b4c875ede137 zepan/zhouyi
```

### 传递文件

```bash
docker cp [DOCKER ID]:[path] [path]
docker cp [path] [DOCKER ID]:[path]
```

## 模型处理

模型下载：[Github](https://github.com/tensorflow/models/tree/master/research/slim#pre-trained-models)
这里下载vgg_16_2016_08_28.tar.gz

```bash
tar xvf vgg_16_2016_08_28.tar.gz
vgg_16.ckpt
```

### 导出图

```bash
git clone git@github.com:tensorflow/models.git
cd models/research/slim
python setup.py install
python3 export_inference_graph.py \
        --alsologtostderr \
        --model_name=vgg_16 \
        --image_size=224 \
        --labels_offset=1 \
        --output_file=/root/test/vgg/model/vgg.pb
```

### 冻结

下载1.15的tensorflow

```bash
git clone -b r1.15 --single-branch https://github.com/tensorflow/tensorflow.git
cd tensorflow/tensorflow/python/tools
python3 freeze_graph.py \
            --input_graph=/root/test/vgg/model/vgg.pb \
            --input_checkpoint=/root/test/vgg/vgg_16.ckpt \
            --input_binary=true \
            --output_node_names=vgg_16/fc8/BiasAdd \
            --output_graph=/root/test/vgg/model/vgg_frozen.pb
```

其中vgg_16/fc8/BiasAdd通过将vgg.pb上传至[Netron](https://netron.app/)网站查看。vgg_frozen.pb存放在工程model路径下。

## 准备数据集

下载ILSVRC2012，通过脚本生成numpy格式的文件

```python
import tensorflow as tf
import numpy as np
import sys
import os
import cv2

sys.path.append('/project/ai/scratch01/salyua01/sharing/guide_acc')
img_dir='./img/'
label_file='./label.txt'

#RESNET PARAM
input_height=224
input_width=224
input_channel = 3
mean = [123.68, 116.78, 103.94]
var = 1

tf.enable_eager_execution()

def smallest_size_at_least(height, width, resize_min):
    """Computes new shape with the smallest side equal to `smallest_side`.

    Computes new shape with the smallest side equal to `smallest_side` while
    preserving the original aspect ratio.

    Args:
      height: an int32 scalar tensor indicating the current height.
      width: an int32 scalar tensor indicating the current width.
      resize_min: A python integer or scalar `Tensor` indicating the size of
        the smallest side after resize.

    Returns:
      new_height: an int32 scalar tensor indicating the new height.
      new_width: an int32 scalar tensor indicating the new width.
    """
    resize_min = tf.cast(resize_min, tf.float32)

    # Convert to floats to make subsequent calculations go smoothly.
    height, width = tf.cast(height, tf.float32), tf.cast(width, tf.float32)

    smaller_dim = tf.minimum(height, width)
    scale_ratio = resize_min / smaller_dim

    # Convert back to ints to make heights and widths that TF ops will accept.
    new_height = tf.cast(tf.round(height * scale_ratio), tf.int32)
    new_width = tf.cast(tf.round(width * scale_ratio), tf.int32)

    return new_height, new_width

def resize_image(image, height, width, method='BILINEAR'):
    """Simple wrapper around tf.resize_images.

    This is primarily to make sure we use the same `ResizeMethod` and other
    details each time.

    Args:
      image: A 3-D image `Tensor`.
      height: The target height for the resized image.
      width: The target width for the resized image.

    Returns:
      resized_image: A 3-D tensor containing the resized image. The first two
        dimensions have the shape [height, width].
    """
    resize_func = tf.image.ResizeMethod.NEAREST_NEIGHBOR if method == 'NEAREST' else tf.image.ResizeMethod.BILINEAR
    return tf.image.resize_images(image, [height, width], method=resize_func, align_corners=False)


def aspect_preserving_resize(image, resize_min, channels=3, method='BILINEAR'):
    """Resize images preserving the original aspect ratio.

    Args:
      image: A 3-D image `Tensor`.
      resize_min: A python integer or scalar `Tensor` indicating the size of
        the smallest side after resize.

    Returns:
      resized_image: A 3-D tensor containing the resized image.
    """
    shape = tf.shape(image)
    height, width = shape[0], shape[1]
    new_height, new_width = smallest_size_at_least(height, width, resize_min)
    return resize_image(image, new_height, new_width, method)


def central_crop(image, crop_height, crop_width, channels=3):
    """Performs central crops of the given image list.

    Args:
      image: a 3-D image tensor
      crop_height: the height of the image following the crop.
      crop_width: the width of the image following the crop.

    Returns:
      3-D tensor with cropped image.
    """
    shape = tf.shape(image)
    height, width = shape[0], shape[1]
    amount_to_be_cropped_h = height - crop_height
    crop_top = amount_to_be_cropped_h // 2
    amount_to_be_cropped_w = width - crop_width
    crop_left = amount_to_be_cropped_w // 2
    # return tf.image.crop_to_bounding_box(image, crop_top, crop_left, crop_height, crop_width)

    size_assertion = tf.Assert(
        tf.logical_and(
            tf.greater_equal(height, crop_height),
            tf.greater_equal(width, crop_width)),
        ['Crop size greater than the image size.']
    )
    with tf.control_dependencies([size_assertion]):
        if channels == 1:
            image = tf.squeeze(image)
            crop_start = [crop_top, crop_left, ]
            crop_shape = [crop_height, crop_width, ]
        elif channels >= 3:
            crop_start = [crop_top, crop_left, 0]
            crop_shape = [crop_height, crop_width, -1]

        image = tf.slice(image, crop_start, crop_shape)

    return tf.reshape(image, [crop_height, crop_width, -1])

label_data = open(label_file)
filename_list = []
label_list = []
for line in label_data:
    filename_list.append(line.rstrip('\n').split(' ')[0])
    label_list.append(int(line.rstrip('\n').split(' ')[1]))
label_data.close()
img_num = len(label_list)

images = np.zeros([img_num, input_height, input_width, input_channel], np.float32)
for file_name, img_idx in zip(filename_list, range(img_num)):
    image_file = os.path.join(img_dir, file_name)
    img_s = tf.gfile.GFile(image_file, 'rb').read()
    image = tf.image.decode_jpeg(img_s)
    image = tf.cast(image, tf.float32)
    image = tf.clip_by_value(image, 0., 255.)
    image = aspect_preserving_resize(image, min(input_height, input_width), input_channel) 
    image = central_crop(image, input_height, input_width)
    image = tf.image.resize_images(image, [input_height, input_width])
    image = (image - mean) / var
    image = image.numpy()
    _, _, ch = image.shape
    if ch == 1:
        image = tf.tile(image, multiples=[1,1,3])
        image = image.numpy()
    images[img_idx] = image

np.save('dataset.npy', images)

labels = np.array(label_list)
np.save('label.npy', labels)
```

label.npy和dataset.npy存放在dataset路径下。

## 仿真配置文件

编写run.cfg

```python
[Common]
mode=run

[Parser]
model_name = vgg_16
detection_postprocess =
model_domain = image_classification
output = vgg_16/fc8/BiasAdd
input_model = ./model/vgg_frozen.pb
input = input
input_shape = [1,224,224,3]
output_dir = ./

[AutoQuantizationTool]
model_name = vgg_16
quantize_method = SYMMETRIC
ops_per_channel = DepthwiseConv
calibration_data = ./dataset/dataset.npy
calibration_label = ./dataset/label.npy
preprocess_mode = normalize
quant_precision=int8
reverse_rgb = False
label_id_offset = 0

[GBuilder]
inputs=./model/input.bin
simulator=aipu_simulator_z1
outputs=output_vgg.bin
profile= True
target=Z1_0701
```

Input.bin文件可以在/root/demos/tflite/model下找到。开始仿真

```bash
aipubuild run.cfg
[I]     step1: get max/min statistic value DONE
[I]     step2: quantization each op DONE
[I]     step3: build quantization forward DONE
[I]     step4: show output scale of end node:
[I]             layer_id: 21, layer_top:vgg_16/fc8/BiasAdd_0, output_scale:[10.010092]
[I] ==== auto-quantization DONE =
[I] Quantize model complete
[I] Building ...
[I] [common_options.h: 276] BuildTool version: 4.0.175. Build for target Z1_0701 at frequency 800MHz
[I] [common_options.h: 297] using default profile events to profile AIFF

[I] [IRChecker] Start to check IR: /tmp/AIPUBuilder_1626512017.731944/vgg_16_int8.txt
[I] [IRChecker] model_name: vgg_16
[I] [IRChecker] IRChecker: All IR pass
[I] [graph.cpp : 846] loading graph weight: /tmp/AIPUBuilder_1626512017.731944/vgg_16_int8.bin size: 0x83fc860
[I] [builder.cpp:1059] Total memory for this graph: 0x9098c90 Bytes
[I] [builder.cpp:1060] Text   section:  0x0000fb90 Bytes
[I] [builder.cpp:1061] RO     section:  0x00001700 Bytes
[I] [builder.cpp:1062] Desc   section:  0x00002000 Bytes
[I] [builder.cpp:1063] Data   section:  0x083f6800 Bytes
[I] [builder.cpp:1064] BSS    section:  0x00c4ee00 Bytes
[I] [builder.cpp:1065] Stack         :  0x00040400 Bytes
[I] [builder.cpp:1066] Workspace(BSS):  0x00310000 Bytes
[I] [main.cpp  : 467] # autogenrated by aipurun, do NOT modify!
LOG_FILE=log_default
FAST_FWD_INST=0
INPUT_INST_CNT=1
INPUT_DATA_CNT=2
CONFIG=Z1-0701
LOG_LEVEL=0
INPUT_INST_FILE0=/tmp/temp_c68782b79b33b6f5c74dd8da6592.text
INPUT_INST_BASE0=0x0
INPUT_INST_STARTPC0=0x0
INPUT_DATA_FILE0=/tmp/temp_c68782b79b33b6f5c74dd8da6592.ro
INPUT_DATA_BASE0=0x10000000
INPUT_DATA_FILE1=/tmp/temp_c68782b79b33b6f5c74dd8da6592.data
INPUT_DATA_BASE1=0x20000000
OUTPUT_DATA_CNT=2
OUTPUT_DATA_FILE0=output_vgg.bin
OUTPUT_DATA_BASE0=0x29328800
OUTPUT_DATA_SIZE0=0x3e8
OUTPUT_DATA_FILE1=profile_data.bin
OUTPUT_DATA_BASE1=0x28436c00
OUTPUT_DATA_SIZE1=0x300
RUN_DESCRIPTOR=BIN[0]

[I] [main.cpp  : 118] run simulator:
aipu_simulator_z1 /tmp/temp_c68782b79b33b6f5c74dd8da6592.cfg
[INFO]:SIMULATOR START!
[INFO]:========================================================================
[INFO]:                             STATIC CHECK
[INFO]:========================================================================
[INFO]:  INST START ADDR : 0x0(0)
[INFO]:  INST END ADDR   : 0xfb8f(64399)
[INFO]:  INST SIZE       : 0xfb90(64400)
[INFO]:  PACKET CNT      : 0xfb9(4025)
[INFO]:  INST CNT        : 0x3ee4(16100)
[INFO]:------------------------------------------------------------------------
[WARN]:[0803] INST WR/RD REG CONFLICT! PACKET 0x1c4: 0x472021b(POP R27,Rc7) vs 0x5f00000(MVI R0,0x0,Rc7), PACKET:0x1c4(452) SLOT:0 vs 3
[WARN]:[0803] INST WR/RD REG CONFLICT! PACKET 0x1d1: 0x472021b(POP R27,Rc7) vs 0x5f00000(MVI R0,0x0,Rc7), PACKET:0x1d1(465) SLOT:0 vs 3
[WARN]:[0803] INST WR/RD REG CONFLICT! PACKET 0x336: 0x472021b(POP R27,Rc7) vs 0x9f80020(ADD.S R0,R0,0x1,Rc7), PACKET:0x336(822) SLOT:0 vs 3
[WARN]:[0803] INST WR/RD REG CONFLICT! PACKET 0x4d3: 0x4520180(BRL R0) vs 0x47a03e5(ADD R5,R0,R31,Rc7), PACKET:0x4d3(1235) SLOT:0 vs 3
[INFO]:========================================================================
[INFO]:                             STATIC CHECK END
[INFO]:========================================================================

[INFO]:AIPU START RUNNING: BIN[0]

[INFO]:TOTAL TIME: 25.355962s.
[INFO]:SIMULATOR EXIT!
[I] [main.cpp  : 135] Simulator finished.
Total errors: 0,  warnings: 0
```

## 测试

通过quant_predict.py脚本进行测试：

```python
from PIL import Image
import cv2
from matplotlib import pyplot as plt
import matplotlib.patches as patches
import numpy as np
import os
import imagenet_classes as class_name

current_dir = os.getcwd()
label_offset = 1
outputfile = current_dir + '/output_vgg.bin'
npyoutput = np.fromfile(outputfile, dtype=np.int8)
outputclass = npyoutput.argmax()
head5p = npyoutput.argsort()[-5:][::-1]

labelfile = current_dir + '/output_ref.bin'
npylabel = np.fromfile(labelfile, dtype=np.int8)
labelclass = npylabel.argmax()
head5t = npylabel.argsort()[-5:][::-1]

print("predict first 5 label:")
for i in head5p:
    print("    index %4d, prob %3d, name: %s"%(i, npyoutput[i], class_name.class_names[i-label_offset]))

print("true first 5 label:")
for i in head5t:
    print("    index %4d, prob %3d, name: %s"%(i, npylabel[i], class_name.class_names[i-label_offset]))

# Show input picture
print('Detect picture save to result.jpeg')

input_path = './model/input.bin'
npyinput = np.fromfile(input_path, dtype=np.int8)
image = np.clip(np.round(npyinput)+128, 0, 255).astype(np.uint8)
image = np.reshape(image, (224, 224, 3))
im = Image.fromarray(image)
im.save('result.jpeg')
```

其中output_ref.bin、imagenet_classes.py可以在/root/demos/tflite下找到。运行结果：

```bash
python3 quant_predict.py
predict first 5 label:
    index  230, prob 116, name: Old English sheepdog, bobtail
    index  231, prob 108, name: Shetland sheepdog, Shetland sheep dog, Shetland
    index  157, prob  71, name: Blenheim spaniel
    index  169, prob  58, name: redbone
    index  259, prob  53, name: Samoyed, Samoyede
true first 5 label:
    index  232, prob  83, name: collie
    index  231, prob  83, name: Shetland sheepdog, Shetland sheep dog, Shetland
    index  158, prob  41, name: papillon
    index  170, prob  40, name: borzoi, Russian wolfhound
    index  161, prob  39, name: Afghan hound, Afghan
```

可以看到正确预测了图片231。文件见[AIPU-Zhouyi-VGG-1](https://github.com/sjl3110/AIPU-Zhouyi-VGG-16)
