---
layout: post
title: SDMA 3D 拷贝底层机理
index_img: /img/post_pics/gpu/sdma/index.png
date: 2024-05-17 20:00:44
tags: 
    - DMA
categories: 
    - GPU
---

GPU MEM3DCPY是怎么实现的。

<!-- more -->
## 目录

- [目录](#目录)
- [定义](#定义)
- [案例](#案例)
- [测试](#测试)

## 定义


这个操作主要依靠 SDMA Linear Sub-Window Copy。该命令用于在两个具有不同大小和起始点(应该是相同元素大小)的表面之间执行子窗口复制。

![](/img/post_pics/gpu/sdma/0.png)
![](/img/post_pics/gpu/sdma/0-1.png)
![](/img/post_pics/gpu/sdma/0-2.png)

其中主要涉及：

- 像素大小：8~128
- 源地址：以三维的形式展开
- 源地址坐标XYZ，在大多数编程和图形上下文中，X和Y通常用来指代水平和垂直空间维度，而Z维则表示深度或从观察点向前的维度。在内存操作的场景中，X、Y和Z可以用来描述要操作的数据区域的三维范围或大小。
- 源切片slice，用来描述沿Z轴（深度轴）的单个二维截面。例如，如果有一个3D图像或体积数据块，一个切片可能代表该数据块中的一个特定深度层。
- 源间距pitch：在图像处理或内存管理中，"pitch"指的是存储单个图像行（二维数据）或切片（三维数据的一部分）所需的内存字节数。
- 目标地址等同上

## 案例

HIP/tests/src/runtimeApi/memory/hipMemset3D.cpp

创建源，目标，存入hipMemcpy3D(&myparms)的参数myparms中，其中地址分别为 0x7ffff3700000 和 0x7ffff7f2f010

![](/img/post_pics/gpu/sdma/1.png)

此参数结构体和内部重要参数结构体定义为

```c
typedef struct hipMemcpy3DParms {
    hipArray_t srcArray;
    struct hipPos srcPos;
    struct hipPitchedPtr srcPtr;
    hipArray_t dstArray;
    struct hipPos dstPos;
    struct hipPitchedPtr dstPtr;
    struct hipExtent extent;
    enum hipMemcpyKind kind;
} hipMemcpy3DParms;

typedef struct hipPitchedPtr {
    void* ptr;
    size_t pitch;
    size_t xsize;
    size_t ysize;
}hipPitchedPtr;
typedef struct hipExtent {
    size_t width;  // Width in elements when referring to array memory, in bytes when referring to
                   // linear memory
    size_t height;
    size_t depth;
}hipExtent;
typedef struct hipPos {
    size_t x;
    size_t y;
    size_t z;
}hipPos;
```

跳转到函数 getDrvMemcpy3DDesc 中后，将入参转化为结构体 HIP_MEMCPY3D desc：

```c
const HIP_MEMCPY3D desc = hip::getDrvMemcpy3DDesc(*p);
```

![](/img/post_pics/gpu/sdma/2.png)


注意这是device到host的拷贝。

```c
typedef struct HIP_MEMCPY3D {
  unsigned int srcXInBytes;
  unsigned int srcY;
  unsigned int srcZ;
  unsigned int srcLOD;
  hipMemoryType srcMemoryType;
  const void* srcHost;
  hipDeviceptr_t srcDevice;
  hipArray_t srcArray;
  unsigned int srcPitch;
  unsigned int srcHeight;
  unsigned int dstXInBytes;
  unsigned int dstY;
  unsigned int dstZ;
  unsigned int dstLOD;
  hipMemoryType dstMemoryType;
  void* dstHost;
  hipDeviceptr_t dstDevice;
  hipArray_t dstArray;
  unsigned int dstPitch;
  unsigned int dstHeight;
  unsigned int WidthInBytes;
  unsigned int Height;
  unsigned int Depth;
} HIP_MEMCPY3D;
```

进入函数 ihipMemcpyParam3D，根据源地址类型和目标地址类型决策拷贝的方向：

```c
if ((srcMemoryType == hipMemoryTypeHost) && (dstMemoryType == hipMemoryTypeHost)) {
    //...
  } else {
    amd::Command* command;
    status = ihipGetMemcpyParam3DCommand(command, pCopy, hip::getQueue(stream));
    if (status != hipSuccess) return status;
    return ihipMemcpyCmdEnqueue(command, isAsync);
  }
```

ihipGetMemcpyParam3DCommand 进一步决策方向：

```c
  amd::Coord3D srcOrigin = {pCopy->srcXInBytes, pCopy->srcY, pCopy->srcZ};
  amd::Coord3D dstOrigin = {pCopy->dstXInBytes, pCopy->dstY, pCopy->dstZ};
  amd::Coord3D copyRegion = {pCopy->WidthInBytes, pCopy->Height, pCopy->Depth};
// ...  

} else if ((srcMemoryType == hipMemoryTypeDevice) && (dstMemoryType == hipMemoryTypeHost)) {
    // Device to Host.
    return ihipMemcpyDtoHCommand(command, pCopy->srcDevice, pCopy->dstHost, srcOrigin, dstOrigin,
                                 copyRegion, pCopy->srcPitch, pCopy->srcPitch * pCopy->srcHeight,
                                 pCopy->dstPitch, pCopy->dstPitch * pCopy->dstHeight, queue);
  }
// slicepitch = pitch * height
```

ihipMemcpyDtoHCommand → ihipMemcpyDtoHValidate

![](/img/post_pics/gpu/sdma/3.png)


这里构造了待拷贝的rect：

![](/img/post_pics/gpu/sdma/4.png)

这里创建了一个read memory 的command，并执行完 validatePeerMemory

向下层传递数据至Runtime层，构造SDMA packet：

```c
template <typename RingIndexTy, bool HwIndexMonotonic, int SizeToCountOffset, bool useGCR>
void BlitSdma<RingIndexTy, HwIndexMonotonic, SizeToCountOffset, useGCR>::BuildCopyRectCommand(
    const std::function<void*(size_t)>& append, const hsa_pitched_ptr_t* dst,
    const hsa_dim3_t* dst_offset, const hsa_pitched_ptr_t* src, const hsa_dim3_t* src_offset,
    const hsa_dim3_t* range) {
  // Returns the index of the first set bit (ie log2 of the largest power of 2 that evenly divides
  // width), the largest element that perfectly covers width.
  // width | 16 ensures that we don't return a higher element than is supported and avoids
  // issues with 0.
  auto maxAlignedElement = [](size_t width) {
    return __builtin_ctz(width | 16);
  };

  // Limits in terms of element count
  const uint32_t max_pitch = 1 << SDMA_PKT_COPY_LINEAR_RECT::pitch_bits;
  const uint32_t max_slice = 1 << SDMA_PKT_COPY_LINEAR_RECT::slice_bits;
  const uint32_t max_x = 1 << SDMA_PKT_COPY_LINEAR_RECT::rect_xy_bits;
  const uint32_t max_y = 1 << SDMA_PKT_COPY_LINEAR_RECT::rect_xy_bits;
  const uint32_t max_z = 1 << SDMA_PKT_COPY_LINEAR_RECT::rect_z_bits;

  // Find maximum element that describes the pitch and slice.
  // Pitch and slice must both be represented in units of elements.  No element larger than this
  // may be used in any tile as the pitches would not be exactly represented.
  int max_ele = Min(maxAlignedElement(src->pitch), maxAlignedElement(dst->pitch));
  if (range->z != 1)  // Only need to consider slice if HW will copy along Z.
    max_ele = Min(max_ele, maxAlignedElement(src->slice), maxAlignedElement(dst->slice));

  /*
  Find the minimum element size that will be needed for any tile.

  No subdivision of a range admits a larger element size for the smallest element in any subdivision
  than the element size that covers the whole range, though some can be worse (this is easily model
  checked).  Subdividing with any element larger than the covering element won't change the covering
  element of the remainder
  ( Range%Element = (Range-N*LargerElement)%Element since LargerElement%Element=0 ).
    Ex. range->x=71, assume max range is 16 elements:  We can break at 64 giving tiles:
    [0,63], [64-70] (width 64 & 7).  64 is covered by element 4 (16B) and 7 is covered by element 0
    (1B).  Exactly covering 71 requires using element 0.

  Base addresses in each tile must be DWORD aligned, if not then the offset from an aligned address
  must be represented in elements.  This may reduce the size of the element, but since elements are
  integer multiples of each other this is harmless.

  src and dst base has already been checked for DWORD alignment so we only need to consider the
  offset here.
  */
  int min_ele = Min(max_ele, maxAlignedElement(range->x), maxAlignedElement(src_offset->x % 4),
                    maxAlignedElement(dst_offset->x % 4));

  // Check that pitch and slice can be represented in the tile with the smallest element
  if ((src->pitch >> min_ele) > max_pitch || (dst->pitch >> min_ele) > max_pitch)
    throw AMD::hsa_exception(HSA_STATUS_ERROR_INVALID_ARGUMENT, "Copy rect pitch out of limits.\n");
  if (range->z != 1) {  // Only need to consider slice if HW will copy along Z.
    if ((src->slice >> min_ele) > max_slice || (dst->slice >> min_ele) > max_slice)
      throw AMD::hsa_exception(HSA_STATUS_ERROR_INVALID_ARGUMENT,
                               "Copy rect slice out of limits.\n");
  }

  // Break copy into tiles
  for (uint32_t z = 0; z < range->z; z += max_z) {
    for (uint32_t y = 0; y < range->y; y += max_y) {
      uint32_t x = 0;
      while (x < range->x) {
        uint32_t width = range->x - x;

        // Get largest element which describes the start of this tile after its base address has
        // been aligned.  Base addresses must be DWORD (4 byte) aligned.
        int aligned_ele = Min(maxAlignedElement((src_offset->x + x) % 4),
                              maxAlignedElement((dst_offset->x + x) % 4), max_ele);

        // Get largest permissible element which exactly covers width
        int element = Min(maxAlignedElement(width), aligned_ele);
        int xcount = width >> element;

        // If width is too large then width is at least max_x bytes (bigger than any element) so
        // drop the width restriction and clip element count to max_x.
        if (xcount > max_x) {
          element = aligned_ele;
          xcount = Min(width >> element, max_x);
        }

        // Get base addresses and offsets for this tile.
        uintptr_t sbase = (uintptr_t)src->base + src_offset->x + x +
            (src_offset->y + y) * src->pitch + (src_offset->z + z) * src->slice;
        uintptr_t dbase = (uintptr_t)dst->base + dst_offset->x + x +
            (dst_offset->y + y) * dst->pitch + (dst_offset->z + z) * dst->slice;
        uint soff = (sbase % 4) >> element;
        uint doff = (dbase % 4) >> element;
        sbase &= ~3ull;
        dbase &= ~3ull;

        x += xcount << element;

        SDMA_PKT_COPY_LINEAR_RECT* pkt =
            (SDMA_PKT_COPY_LINEAR_RECT*)append(sizeof(SDMA_PKT_COPY_LINEAR_RECT));
        *pkt = {};
        pkt->HEADER_UNION.op = SDMA_OP_COPY;
        pkt->HEADER_UNION.sub_op = SDMA_SUBOP_COPY_LINEAR_RECT;
        pkt->HEADER_UNION.element = element;
        pkt->SRC_ADDR_LO_UNION.src_addr_31_0 = sbase;
        pkt->SRC_ADDR_HI_UNION.src_addr_63_32 = sbase >> 32;
        pkt->SRC_PARAMETER_1_UNION.src_offset_x = soff;
        pkt->SRC_PARAMETER_2_UNION.src_pitch = (src->pitch >> element) - 1;
        pkt->SRC_PARAMETER_3_UNION.src_slice_pitch =
            (range->z == 1) ? 0 : (src->slice >> element) - 1;
        pkt->DST_ADDR_LO_UNION.dst_addr_31_0 = dbase;
        pkt->DST_ADDR_HI_UNION.dst_addr_63_32 = dbase >> 32;
        pkt->DST_PARAMETER_1_UNION.dst_offset_x = doff;
        pkt->DST_PARAMETER_2_UNION.dst_pitch = (dst->pitch >> element) - 1;
        pkt->DST_PARAMETER_3_UNION.dst_slice_pitch =
            (range->z == 1) ? 0 : (dst->slice >> element) - 1;
        pkt->RECT_PARAMETER_1_UNION.rect_x = xcount - 1;
        pkt->RECT_PARAMETER_1_UNION.rect_y = Min(range->y - y, max_y) - 1;
        pkt->RECT_PARAMETER_2_UNION.rect_z = Min(range->z - z, max_z) - 1;
      }
    }
  }
}
```

在包中，分别填入指定的数据。对其描述参考下述测试。

## 测试

在KFDTest中直接构造待拷贝的数据结构，这里使用的是统一内存。

- 源地址和目标地址为一维数组，大小为 128 x 128 x 128 bytes
- 待拷贝的大小为 32 x 32 x 1 bytes
- 目标相对偏移是居中
- 注意：在填充包的时候，pitch, slice_picth，以及copy rect width都需要转换为pixel单位
- 注意：picth和rect width大小不能小于一个像素的长度，例如element = 4，表示一个像素为连续16bytes，则最小跨度为16 bytes

![](/img/post_pics/gpu/sdma/5.png)

对每个element测试，测试结果为：

```txt
[ RUN      ] SDMACopyDataRectTest.CopyRectMultiPixel/0
[          ] ************************************
[          ] *        SDMA Copy Rect Info       *
[          ] ************************************
[          ] One pixel size:    1 bytes
[          ] SRC rect size:     128 * 16384 bytes
[          ] DST rect size:     128 * 16384 bytes
[          ] Copy rect size:    32 * 32 * 1 bytes
[          ] SRC offset:        (0, 0, 0) pixels
[          ] DST offset:        (48, 48, 0) pixels
[       OK ] SDMACopyDataRectTest.CopyRectMultiPixel/0 (28 ms)
[ RUN      ] SDMACopyDataRectTest.CopyRectMultiPixel/1
[          ] ************************************
[          ] *        SDMA Copy Rect Info       *
[          ] ************************************
[          ] One pixel size:    2 bytes
[          ] SRC rect size:     128 * 16384 bytes
[          ] DST rect size:     128 * 16384 bytes
[          ] Copy rect size:    32 * 32 * 1 bytes
[          ] SRC offset:        (0, 0, 0) pixels
[          ] DST offset:        (24, 48, 0) pixels
[       OK ] SDMACopyDataRectTest.CopyRectMultiPixel/1 (8 ms)
[ RUN      ] SDMACopyDataRectTest.CopyRectMultiPixel/2
[          ] ************************************
[          ] *        SDMA Copy Rect Info       *
[          ] ************************************
[          ] One pixel size:    4 bytes
[          ] SRC rect size:     128 * 16384 bytes
[          ] DST rect size:     128 * 16384 bytes
[          ] Copy rect size:    32 * 32 * 1 bytes
[          ] SRC offset:        (0, 0, 0) pixels
[          ] DST offset:        (12, 48, 0) pixels
[       OK ] SDMACopyDataRectTest.CopyRectMultiPixel/2 (9 ms)
[ RUN      ] SDMACopyDataRectTest.CopyRectMultiPixel/3
[          ] ************************************
[          ] *        SDMA Copy Rect Info       *
[          ] ************************************
[          ] One pixel size:    8 bytes
[          ] SRC rect size:     128 * 16384 bytes
[          ] DST rect size:     128 * 16384 bytes
[          ] Copy rect size:    32 * 32 * 1 bytes
[          ] SRC offset:        (0, 0, 0) pixels
[          ] DST offset:        (6, 48, 0) pixels
[       OK ] SDMACopyDataRectTest.CopyRectMultiPixel/3 (8 ms)
[ RUN      ] SDMACopyDataRectTest.CopyRectMultiPixel/4
[          ] ************************************
[          ] *        SDMA Copy Rect Info       *
[          ] ************************************
[          ] One pixel size:    16 bytes
[          ] SRC rect size:     128 * 16384 bytes
[          ] DST rect size:     128 * 16384 bytes
[          ] Copy rect size:    32 * 32 * 1 bytes
[          ] SRC offset:        (0, 0, 0) pixels
[          ] DST offset:        (3, 48, 0) pixels
[       OK ] SDMACopyDataRectTest.CopyRectMultiPixel/4 (8 ms)
```

打印数组如下，阴影部分为目标拷贝区域。

![](/img/post_pics/gpu/sdma/6.png)
