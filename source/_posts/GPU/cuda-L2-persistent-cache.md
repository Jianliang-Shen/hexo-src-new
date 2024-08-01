---
layout: post
title: CUDA L2 持久缓存
index_img: /img/gpu/L2-cache.png
date: 2024-08-01 20:11:58
tags: 
    - Cache
    - GPU
    - CUDA
categories: 
    - GPU
---

从 CUDA 11.0 开始，计算能力 8.0 及以上的设备能够影响 L2 缓存中数据的持久性。由于 L2 缓存位于片上，因此它有可能为全局内存提供更高的带宽和更低的延迟访问。

<!-- more -->

转载：[CUDA L2 Persistent Cache](https://leimao.github.io/blog/CUDA-L2-Persistent-Cache/)

更多参考：

- [L2 Persistence Example](https://docs.nvidia.com/cuda/cuda-c-programming-guide/index.html#l2-persistence-example)
- [L2 persistence clarifications](https://forums.developer.nvidia.com/t/l2-persistence-clarifications/281031/2)

## 介绍

在这篇博文中，我创建了一个 CUDA 示例来演示如何使用 L2 持久缓存来加速数据流量。

## CUDA L2 持久缓存

在此示例中，我将有一个具有某些值的小型常量缓冲区，用于重置大型流缓冲区。例如，如果常量缓冲区的大小为 4，值为[5, 2, 1, 4] ，并且要重置的大流缓冲区的大小为 100，则重置后，大流缓冲区的值为[5, 2, 1, 4, 5, 2, 1, 4, ...] ，即重复常量缓冲区的值。

由于流式缓冲区比常量缓冲区大得多，因此常量缓冲区中的每个元素比流式缓冲区的访问频率更高。从全局内存访问缓冲区非常昂贵。如果我们能够将频繁访问的常量缓冲区缓存在二级缓存中，则可以加速对经常访问的常量缓冲区的访问。

## CUDA 数据重置

对于数据重置 CUDA 内核，我创建了一个在不使用持久 L2 缓存的情况下启动内核的基线，一个使用 3 MB 持久 L2 缓存启动内核但在恒定缓冲区大小超过 3 MB 时出现数据抖动的变体，以及一个优化变体它使用 3 MB 持久 L2 缓存启动内核，但消除了数据抖动。

{% fold @l2-persistent.cu %}

```c
#include <algorithm>
#include <cassert>
#include <cstdlib>
#include <functional>
#include <iomanip>
#include <iostream>
#include <vector>

#include <cuda_runtime.h>

#define CHECK_CUDA_ERROR(val) check((val), #val, __FILE__, __LINE__)
void check(cudaError_t err, char const* const func, char const* const file,
           int const line)
{
    if (err != cudaSuccess)
    {
        std::cerr << "CUDA Runtime Error at: " << file << ":" << line
                  << std::endl;
        std::cerr << cudaGetErrorString(err) << " " << func << std::endl;
        std::exit(EXIT_FAILURE);
    }
}

#define CHECK_LAST_CUDA_ERROR() checkLast(__FILE__, __LINE__)
void checkLast(char const* const file, int const line)
{
    cudaError_t const err{cudaGetLastError()};
    if (err != cudaSuccess)
    {
        std::cerr << "CUDA Runtime Error at: " << file << ":" << line
                  << std::endl;
        std::cerr << cudaGetErrorString(err) << std::endl;
        std::exit(EXIT_FAILURE);
    }
}

template <class T>
float measure_performance(std::function<T(cudaStream_t)> bound_function,
                          cudaStream_t stream, int num_repeats = 100,
                          int num_warmups = 100)
{
    cudaEvent_t start, stop;
    float time;

    CHECK_CUDA_ERROR(cudaEventCreate(&start));
    CHECK_CUDA_ERROR(cudaEventCreate(&stop));

    for (int i{0}; i < num_warmups; ++i)
    {
        bound_function(stream);
    }

    CHECK_CUDA_ERROR(cudaStreamSynchronize(stream));

    CHECK_CUDA_ERROR(cudaEventRecord(start, stream));
    for (int i{0}; i < num_repeats; ++i)
    {
        bound_function(stream);
    }
    CHECK_CUDA_ERROR(cudaEventRecord(stop, stream));
    CHECK_CUDA_ERROR(cudaEventSynchronize(stop));
    CHECK_LAST_CUDA_ERROR();
    CHECK_CUDA_ERROR(cudaEventElapsedTime(&time, start, stop));
    CHECK_CUDA_ERROR(cudaEventDestroy(start));
    CHECK_CUDA_ERROR(cudaEventDestroy(stop));

    float const latency{time / num_repeats};

    return latency;
}

__global__ void reset_data(int* data_streaming, int const* lut_persistent,
                           size_t data_streaming_size,
                           size_t lut_persistent_size)
{
    size_t const idx{blockDim.x * blockIdx.x + threadIdx.x};
    size_t const stride{blockDim.x * gridDim.x};
    for (size_t i{idx}; i < data_streaming_size; i += stride)
    {
        data_streaming[i] = lut_persistent[i % lut_persistent_size];
    }
}

/**
 * @brief Reset the data_streaming using lut_persistent so that the
 * data_streaming is lut_persistent repeatedly.
 *
 * @param data_streaming The data for reseting.
 * @param lut_persistent The values for resetting data_streaming.
 * @param data_streaming_size The size for data_streaming.
 * @param lut_persistent_size The size for lut_persistent.
 * @param stream The CUDA stream.
 */
void launch_reset_data(int* data_streaming, int const* lut_persistent,
                       size_t data_streaming_size, size_t lut_persistent_size,
                       cudaStream_t stream)
{
    dim3 const threads_per_block{1024};
    dim3 const blocks_per_grid{32};
    reset_data<<<blocks_per_grid, threads_per_block, 0, stream>>>(
        data_streaming, lut_persistent, data_streaming_size,
        lut_persistent_size);
    CHECK_LAST_CUDA_ERROR();
}

bool verify_data(int* data, int n, size_t size)
{
    for (size_t i{0}; i < size; ++i)
    {
        if (data[i] != i % n)
        {
            return false;
        }
    }
    return true;
}

int main(int argc, char* argv[])
{
    size_t num_megabytes_persistent_data{3};
    if (argc == 2)
    {
        num_megabytes_persistent_data = std::atoi(argv[1]);
    }

    constexpr int const num_repeats{100};
    constexpr int const num_warmups{10};

    cudaDeviceProp device_prop{};
    int current_device{0};
    CHECK_CUDA_ERROR(cudaGetDevice(&current_device));
    CHECK_CUDA_ERROR(cudaGetDeviceProperties(&device_prop, current_device));
    std::cout << "GPU: " << device_prop.name << std::endl;
    std::cout << "L2 Cache Size: " << device_prop.l2CacheSize / 1024 / 1024
              << " MB" << std::endl;
    std::cout << "Max Persistent L2 Cache Size: "
              << device_prop.persistingL2CacheMaxSize / 1024 / 1024 << " MB"
              << std::endl;

    size_t const num_megabytes_streaming_data{1024};
    if (num_megabytes_persistent_data > num_megabytes_streaming_data)
    {
        std::runtime_error(
            "Try setting persistent data size smaller than 1024 MB.");
    }
    size_t const size_persistent(num_megabytes_persistent_data * 1024 * 1024 /
                                 sizeof(int));
    size_t const size_streaming(num_megabytes_streaming_data * 1024 * 1024 /
                                sizeof(int));
    std::cout << "Persistent Data Size: " << num_megabytes_persistent_data
              << " MB" << std::endl;
    std::cout << "Steaming Data Size: " << num_megabytes_streaming_data << " MB"
              << std::endl;
    cudaStream_t stream;

    std::vector<int> lut_persistent_vec(size_persistent, 0);
    for (size_t i{0}; i < lut_persistent_vec.size(); ++i)
    {
        lut_persistent_vec[i] = i;
    }
    std::vector<int> data_streaming_vec(size_streaming, 0);

    int* d_lut_persistent;
    int* d_data_streaming;
    int* h_lut_persistent = lut_persistent_vec.data();
    int* h_data_streaming = data_streaming_vec.data();

    CHECK_CUDA_ERROR(
        cudaMalloc(&d_lut_persistent, size_persistent * sizeof(int)));
    CHECK_CUDA_ERROR(
        cudaMalloc(&d_data_streaming, size_streaming * sizeof(int)));
    CHECK_CUDA_ERROR(cudaStreamCreate(&stream));
    CHECK_CUDA_ERROR(cudaMemcpy(d_lut_persistent, h_lut_persistent,
                                size_persistent * sizeof(int),
                                cudaMemcpyHostToDevice));

    launch_reset_data(d_data_streaming, d_lut_persistent, size_streaming,
                      size_persistent, stream);
    CHECK_CUDA_ERROR(cudaStreamSynchronize(stream));
    CHECK_CUDA_ERROR(cudaMemcpy(h_data_streaming, d_data_streaming,
                                size_streaming * sizeof(int),
                                cudaMemcpyDeviceToHost));
    assert(verify_data(h_data_streaming, size_persistent, size_streaming));

    std::function<void(cudaStream_t)> const function{
        std::bind(launch_reset_data, d_data_streaming, d_lut_persistent,
                  size_streaming, size_persistent, std::placeholders::_1)};
    float const latency{
        measure_performance(function, stream, num_repeats, num_warmups)};
    std::cout << std::fixed << std::setprecision(3)
              << "Latency Without Using Persistent L2 Cache: " << latency
              << " ms" << std::endl;

    // Start to use persistent cache.
    cudaStream_t stream_persistent_cache;
    size_t const num_megabytes_persistent_cache{3};
    CHECK_CUDA_ERROR(cudaStreamCreate(&stream_persistent_cache));

    CHECK_CUDA_ERROR(
        cudaDeviceSetLimit(cudaLimitPersistingL2CacheSize,
                           num_megabytes_persistent_cache * 1024 * 1024));

    cudaStreamAttrValue stream_attribute_thrashing;
    stream_attribute_thrashing.accessPolicyWindow.base_ptr =
        reinterpret_cast<void*>(d_lut_persistent);
    stream_attribute_thrashing.accessPolicyWindow.num_bytes =
        num_megabytes_persistent_data * 1024 * 1024;
    stream_attribute_thrashing.accessPolicyWindow.hitRatio = 1.0;
    stream_attribute_thrashing.accessPolicyWindow.hitProp =
        cudaAccessPropertyPersisting;
    stream_attribute_thrashing.accessPolicyWindow.missProp =
        cudaAccessPropertyStreaming;

    CHECK_CUDA_ERROR(cudaStreamSetAttribute(
        stream_persistent_cache, cudaStreamAttributeAccessPolicyWindow,
        &stream_attribute_thrashing));

    float const latency_persistent_cache_thrashing{measure_performance(
        function, stream_persistent_cache, num_repeats, num_warmups)};
    std::cout << std::fixed << std::setprecision(3) << "Latency With Using "
              << num_megabytes_persistent_cache
              << " MB Persistent L2 Cache (Potentially Thrashing): "
              << latency_persistent_cache_thrashing << " ms" << std::endl;

    cudaStreamAttrValue stream_attribute_non_thrashing{
        stream_attribute_thrashing};
    stream_attribute_non_thrashing.accessPolicyWindow.hitRatio =
        std::min(static_cast<double>(num_megabytes_persistent_cache) /
                     num_megabytes_persistent_data,
                 1.0);
    CHECK_CUDA_ERROR(cudaStreamSetAttribute(
        stream_persistent_cache, cudaStreamAttributeAccessPolicyWindow,
        &stream_attribute_non_thrashing));

    float const latency_persistent_cache_non_thrashing{measure_performance(
        function, stream_persistent_cache, num_repeats, num_warmups)};
    std::cout << std::fixed << std::setprecision(3) << "Latency With Using "
              << num_megabytes_persistent_cache
              << " MB Persistent L2 Cache (Non-Thrashing): "
              << latency_persistent_cache_non_thrashing << " ms" << std::endl;

    CHECK_CUDA_ERROR(cudaFree(d_lut_persistent));
    CHECK_CUDA_ERROR(cudaFree(d_data_streaming));
    CHECK_CUDA_ERROR(cudaStreamDestroy(stream));
    CHECK_CUDA_ERROR(cudaStreamDestroy(stream_persistent_cache));
}
```

{% endfold %}

为了避免数据抖动， `accessPolicyWindow.hitRatio`和`accessPolicyWindow.num_bytes`的乘积应小于或等于`cudaLimitPersistingL2CacheSize`。`accessPolicyWindow.hitRatio`参数可用于指定接收`accessPolicyWindow.hitProp`属性的访问的比例，该属性通常是`cudaAccessPropertyPersisting`。`accessPolicyWindow.num_bytes`参数可用于指定访问策略窗口覆盖的字节数，通常是持久数据的大小。

实际上，我们可以将`accessPolicyWindow.hitRatio`设置为持久二级缓存大小与持久数据大小的比率。例如，如果持久二级缓存大小为 3 MB，持久数据大小为 4 MB，我们可以将`accessPolicyWindow.hitRatio`设置为 3 / 4 = 0.75。

## 运行 CUDA 数据重置

我们可以在 NVIDIA Ampere GPU 上构建并运行该示例。就我而言，我使用了 NVIDIA RTX 3090 GPU。

```bash
$ nvcc l2-persistent.cu -o l2-persistent -std=c++14 --gpu-architecture=compute_80
$ ./l2-persistent
GPU: NVIDIA GeForce RTX 3090
L2 Cache Size: 6 MB
Max Persistent L2 Cache Size: 4 MB
Persistent Data Size: 3 MB
Steaming Data Size: 1024 MB
Latency Without Using Persistent L2 Cache: 3.071 ms
Latency With Using 3 MB Persistent L2 Cache (Potentially Thrashing): 2.436 ms
Latency With Using 3 MB Persistent L2 Cache (Non-Thrashing): 2.443 ms
```

我们可以看到，当持久数据大小为 3 MB、持久二级缓存为 3 MB 时，应用程序的性能提高了大约 20%。

## Benchmarking

我们还可以通过改变持久数据大小来运行一些小型基准测试。

{% fold @test log %}

```bash
$ ./l2-persistent 1
GPU: NVIDIA GeForce RTX 3090
L2 Cache Size: 6 MB
Max Persistent L2 Cache Size: 4 MB
Persistent Data Size: 1 MB
Steaming Data Size: 1024 MB
Latency Without Using Persistent L2 Cache: 1.754 ms
Latency With Using 3 MB Persistent L2 Cache (Potentially Thrashing): 1.685 ms
Latency With Using 3 MB Persistent L2 Cache (Non-Thrashing): 1.674 ms

$ ./l2-persistent 2
GPU: NVIDIA GeForce RTX 3090
L2 Cache Size: 6 MB
Max Persistent L2 Cache Size: 4 MB
Persistent Data Size: 2 MB
Steaming Data Size: 1024 MB
Latency Without Using Persistent L2 Cache: 2.158 ms
Latency With Using 3 MB Persistent L2 Cache (Potentially Thrashing): 1.997 ms
Latency With Using 3 MB Persistent L2 Cache (Non-Thrashing): 2.002 ms

$ ./l2-persistent 3
GPU: NVIDIA GeForce RTX 3090
L2 Cache Size: 6 MB
Max Persistent L2 Cache Size: 4 MB
Persistent Data Size: 3 MB
Steaming Data Size: 1024 MB
Latency Without Using Persistent L2 Cache: 3.095 ms
Latency With Using 3 MB Persistent L2 Cache (Potentially Thrashing): 2.510 ms
Latency With Using 3 MB Persistent L2 Cache (Non-Thrashing): 2.533 ms

$ ./l2-persistent 4
GPU: NVIDIA GeForce RTX 3090
L2 Cache Size: 6 MB
Max Persistent L2 Cache Size: 4 MB
Persistent Data Size: 4 MB
Steaming Data Size: 1024 MB
Latency Without Using Persistent L2 Cache: 3.906 ms
Latency With Using 3 MB Persistent L2 Cache (Potentially Thrashing): 3.632 ms
Latency With Using 3 MB Persistent L2 Cache (Non-Thrashing): 3.706 ms

$ ./l2-persistent 5
GPU: NVIDIA GeForce RTX 3090
L2 Cache Size: 6 MB
Max Persistent L2 Cache Size: 4 MB
Persistent Data Size: 5 MB
Steaming Data Size: 1024 MB
Latency Without Using Persistent L2 Cache: 4.120 ms
Latency With Using 3 MB Persistent L2 Cache (Potentially Thrashing): 4.554 ms
Latency With Using 3 MB Persistent L2 Cache (Non-Thrashing): 3.920 ms

$ ./l2-persistent 6
GPU: NVIDIA GeForce RTX 3090
L2 Cache Size: 6 MB
Max Persistent L2 Cache Size: 4 MB
Persistent Data Size: 6 MB
Steaming Data Size: 1024 MB
Latency Without Using Persistent L2 Cache: 4.194 ms
Latency With Using 3 MB Persistent L2 Cache (Potentially Thrashing): 4.583 ms
Latency With Using 3 MB Persistent L2 Cache (Non-Thrashing): 4.255 ms
```

{% endfold %}

我们可以看到，即使持久数据大小大于持久二级缓存，使用无抖动的持久二级缓存的延迟通常也不会比基线更差。

## FAQ

Q: 持久缓存 VS 共享内存？

持久缓存与共享内存不同。持久缓存对GPU中的所有线程都可见，而共享内存仅对同一块中的线程可见。对于经常访问的小数据，我们还可以使用共享内存来加速数据访问。但是，每个线程块的共享内存限制为 48 到 96 KB，具体取决于 GPU，而持久缓存则限制为每个 GPU 几 MB。

## 参考

[L2 Cache Window](https://docs.nvidia.com/cuda/archive/11.7.0/cuda-c-best-practices-guide/index.html#L2-cache-window)
[Function Binding and Performance Measurement](https://docs.nvidia.com/cuda/archive/11.7.0/cuda-c-best-practices-guide/index.html#L2-cache-window)
