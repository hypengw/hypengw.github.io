+++
title = 'Intel B580 显卡使用体验'
date = 2025-01-07T00:44:03+08:00
draft = false
tags = ['硬件', 'Linux']

+++

想着需要给自己买张显卡了，原本打算等着核显继续迭代，但感觉还是太慢了。  
我的要求不太高，能 1080p 60fps 全高画质玩大部分游戏，能跑一些 ai 应用就更好了。

原本打算 4060 的，但是显存有点小了，最近刚好 Intel 发了 B580，然后就决定是它了。  
对 Intel 印象还是不错，虽然最近发展不太行，不过那也不影响，现有的相关设施已经够用了。  
同时也一直打算试试 oneapi，我还挺看好 SYCL 的，就是不知道何时才能进 upstream llvm。

抱着还需要废心思折腾的预期，结果拿到就能直接在 Linux 桌面下用了。  

## 基本配置

| Specifications            |             |
| :------------------------ | ----------- |
| Xe-cores                  | 20          |
| Graphics Clock            | 2670 MHz    |
| Memory                    | 12 GB GDDR6 |
| Graphics Memory Interface | 192 bit     |
| Graphics Memory Bandwidth | 456 GB/s    |
| GPU Peak TOPS (Int8)      | 233         |
| TBP                       | 190 W       |

## 系统环境

kernal: 6.12.11  
mesa: 24.3.4  
os: fedora 41  
egpu: m2 to oculink

### PCIe Gen 1x1

`lspci` 对于 Intel Arc 显卡总是显示 PCIe 1x1 的速度  
这似乎是预期行为，并不是实际的速度，[Troubleshooting](https://www.intel.com/content/www/us/en/support/articles/000094587/graphics.html)， [Intel Support Forums](https://community.intel.com/t5/Graphics/Intel-Arc-A770-PCIe-Speed-2-5GT-s-Width-x1/td-p/1455448)

具体的连接速度可以用 [xpumanager](https://github.com/intel/xpumanager) 来测试

### Resizable **BAR**

Intel 说明 Arc 显卡需要开启 **Resizable BAR** 才能完全发挥性能  
查了下资料，**Base Address Register** 用于把 pcie 设备的资源映射到系统内存地址，内部有一套同步逻辑

台式主板一般有直接的选项开启  
笔记本和mini主机需要把分配给核显的内存开到 4G 以上

```text
$ sudo lspci -s 03:00.0 -vv | grep -i bar 
     Capabilities: [420 v1] Physical Resizable BAR 
         BAR 2: current size: 16GB, supported: 256MB 512MB 1GB 2GB 4GB 8GB 16GB
```

## AI 应用

### llama.cpp

支持 SYCL 和 Vulkan 后端，我写好了 [Dockerfile](https://github.com/hypengw/dockerfiles/tree/main/llama.cpp).   
SYCL 版本需要安装 intel oneapi，然后在构建时挂载进容器  
`llama-bench -ngl 99` 结果([统计参考](https://github.com/ggerganov/llama.cpp/discussions/10879))：

- SYCL

  | model         |     size | params | backend |  ngl |  test |            t/s |
  | ------------- | -------: | -----: | ------- | ---: | ----: | -------------: |
  | llama 7B Q4_0 | 3.56 GiB | 6.74 B | SYCL    |   99 | pp512 | 1982.69 ± 4.02 |
  | llama 7B Q4_0 | 3.56 GiB | 6.74 B | SYCL    |   99 | tg128 |   34.83 ± 0.11 |

- Vulkan
    ```text
    ggml_vulkan: 0 = Intel(R) Graphics (BMG G21) (Intel open-source Mesa driver) | uma: 0 | fp16: 1 | warp size: 32 | matrix cores: none 
    ```
    | model         |     size | params | backend |  ngl |  test |           t/s |
    | ------------- | -------: | -----: | ------- | ---: | ----: | ------------: |
    | llama 7B Q4_0 | 3.56 GiB | 6.74 B | Vulkan  |   99 | pp512 | 175.56 ± 2.65 |
    | llama 7B Q4_0 | 3.56 GiB | 6.74 B | Vulkan  |   99 | tg128 |  44.12 ± 0.09 |

### ollama

ollama 官方没有给 SYCL 支持说明，但是它的后端是 llama.cpp, 改改就能运行了。  
配合 [open-webui](https://github.com/open-webui/open-webui)，能建立知识库和联网搜索，还挺不错。   
[Dockerfile](https://github.com/hypengw/dockerfiles/tree/main/ollama)

- `OLLAMA_GPU_OVERHEAD`  
  保留的不使用的显存(bytes)，一开始我还以为是最多能使用的显存，发现 ollama 怎么不使用 gpu，才发现是搞反了  
  不知道出于什么原因，linux 这边没有有效手段获取 ARC GPU 可用的 vram，为了避免 oom，这个必须配置

- `ollama ps`  
  可以看到每个模型的运行情况

TODO

## 游戏

### mc

平均FPS: 280  

- mc 1.21.4 1080p
- Iris + Sodium(默认配置)
- BSL shader 8.4 High

![mc](https://alist.bluempty.com/d/Public/Blog/b580/mc.webp)

### 赛博朋克 2077

需要 24.3.4 及以上的 mesa，不然会有 shader 渲染错误  
光追似乎不工作

1080p 最高画质  
平均FPS: 68  
最低FPS: 56  
最高FPS: 82

![2077](https://alist.bluempty.com/d/Public/Blog/b580/2077.webp)

### 极限竞速地平线 4

博主 4 还没玩完，并没有买 5  

1080p 超高  
平均FPS：136  
最低FPS: 119  
最高FPS: 166

![极限竞速地平线4](https://alist.bluempty.com/d/Public/Blog/b580/forza_horizon_4.webp)

## 媒体

需要 24.4.4 及以上的 [intel media-driver](https://github.com/intel/media-driver.git)。  
[media features 文档](https://github.com/intel/media-driver/blob/master/docs/media_features.md)，b580 是 BMG

### 解码

2160p 120fps hevc main10 [mmd](https://steamcommunity.com/sharedfiles/filedetails/?l=schinese&id=2156267111):  **5路**  
av1 main 和 hevc rext 422 也是5路

![初音 - 拼湊的斷音](https://alist.bluempty.com/d/Public/Blog/b580/decoder.webp)

2160p(21:9) 120fps hevc main [mmd](https://www.bilibili.com/video/BV16m411Q7pa/): **4路**

![世界第一的公主殿下](https://alist.bluempty.com/d/Public/Blog/b580/decoder_21_9.webp)

### 编码

原视频为上面测试解码的两个视频  

| 分辨率      | 帧率 | 编码 | 格式      | 编码配置          | 编码速率 |
| :---------- | ---- | ---- | --------- | ----------------- | -------- |
| 2160p(21:9) | 120  | hevc | p010      | main10,QVBR,qp:12 | 1.04x    |
| 2160p       | 120  | hevc | p010      | main10,QVBR,qp:12 | 1.37x    |
| 2160p       | 120  | hevc | y210(422) | rext,QVBR,qp:12   | 1.32x    |
| 2160p       | 120  | av1  | p010      | main,VBR,crf:24   | 1.37x    |
| 2160p(21:9) | 60   | hevc | p010      | main10,QVBR,qp:12 | 2.06x    |
| 2160p       | 60   | hevc | p010      | main10,QVBR,qp:12 | 1.81x    |
| 2160p       | 60   | av1  | p010      | main,VBR,crf:24   | 1.81x    |

```bash
ffmpeg -hwaccel vaapi -hwaccel_output_format vaapi -i input.mp4 -vf 'scale_vaapi=format=p010' -c:v hevc_vaapi -profile:v main10 -tier high -rc_mode QVBR -qp 12 -b:v 40M output3.mp4
```

## 性能测试

### GravityMark

[统计网站](https://gravitymark.tellusim.com/leaderboard/)

- Vulkan 1920x1080 Asteroids: 200000 Score: **24896**
- Opengl 1920x1080 Asteroids: 200000 Score: **20477**

### clpeak

[统计网站](https://openbenchmarking.org/test/pts/clpeak)

`Single-precision compute` 和 4060 差不多  
`Half-precision compute` 网站没有数据，看下面 `vkpeak`, 和 4070 super 差不多  
`Integer compute` 差不多 4060 的一半

```text
$ ./clpeak 

Platform: Intel(R) OpenCL Graphics
  Device: Intel(R) Graphics [0xe20b]
    Driver version  : 24.35.30872.32 (Linux x64)
    Compute units   : 160
    Clock frequency : 2850 MHz

    Global memory bandwidth (GBPS)
      float   : 417.71
      float2  : 429.27
      float4  : 434.78
      float8  : 446.30
      float16 : 449.03

    Single-precision compute (GFLOPS)
      float   : 14083.21
      float2  : 14401.35
      float4  : 14096.05
      float8  : 12096.30
      float16 : 13596.31

    Half-precision compute (GFLOPS)
      half   : 24831.84
      half2  : 28076.27
      half4  : 28199.86
      half8  : 27916.44
      half16 : 27768.14

    Double-precision compute (GFLOPS)
      double   : 891.12
      double2  : 893.37
      double4  : 898.90
      double8  : 888.26
      double16 : 842.54

    Integer compute (GIOPS)
      int   : 4433.77
      int2  : 4461.17
      int4  : 4456.32
      int8  : 4364.93
      int16 : 4192.12

    Integer compute Fast 24bit (GIOPS)
      int   : 4473.72
      int2  : 4448.66
      int4  : 4470.87
      int8  : 4379.65
      int16 : 4225.50

    Integer char (8bit) compute (GIOPS)
      char   : 21487.19
      char2  : 26128.29
      char4  : 25867.58
      char8  : 25227.17
      char16 : 24600.19

    Integer short (16bit) compute (GIOPS)
      short   : 20924.35
      short2  : 26189.49
      short4  : 25835.04
      short8  : 25127.30
      short16 : 23874.53

    Transfer bandwidth (GBPS)
      enqueueWriteBuffer              : 6.77
      enqueueReadBuffer               : 6.71
      enqueueWriteBuffer non-blocking : 7.08
      enqueueReadBuffer non-blocking  : 7.01
      enqueueMapBuffer(for read)      : 6.97
        memcpy from mapped ptr        : 26.67
      enqueueUnmap(after write)       : 7.20
        memcpy to mapped ptr          : 26.41

    Kernel launch latency : 49.15 us
```

### vkpeak

[统计网站](https://openbenchmarking.org/test/pts/vkpeak)

`fp32` 和 4060 差不多  
`int32` 特别差  
其他的居然能跑到和 4070 super 差不多

```text
$ ./vkpeak 0 
device       = Intel(R) Graphics (BMG G21)

fp32-scalar  = 7277.51 GFLOPS
fp32-vec4    = 10706.95 GFLOPS

fp16-scalar  = 21068.69 GFLOPS
fp16-vec4    = 23754.69 GFLOPS
fp16-matrix  = 0.00 GFLOPS

fp64-scalar  = 799.67 GFLOPS
fp64-vec4    = 779.47 GFLOPS

int32-scalar = 3024.92 GIOPS
int32-vec4   = 3102.06 GIOPS

int16-scalar = 10801.67 GIOPS
int16-vec4   = 13525.01 GIOPS
```

### VkFFT

[统计网站](https://openbenchmarking.org/test/pts/vkfft)

- single precision

```text
$ ./VkFFT_TestSuite -vkfft 0
0 - VkFFT FFT + iFFT C2C benchmark 1D batched in single precision
VkFFT System: 3 8x16777216 Buffer: 1024 MB avg_time_per_step: 10.984 ms std_error: 0.054 num_iter: 3 benchmark: 95462 bandwidth: 364.2
VkFFT System: 4 16x8388608 Buffer: 1024 MB avg_time_per_step: 11.302 ms std_error: 0.378 num_iter: 3 benchmark: 92774 bandwidth: 353.9
VkFFT System: 5 32x4194304 Buffer: 1024 MB avg_time_per_step: 11.164 ms std_error: 0.332 num_iter: 3 benchmark: 93921 bandwidth: 358.3
VkFFT System: 6 64x2097152 Buffer: 1024 MB avg_time_per_step: 10.819 ms std_error: 0.024 num_iter: 3 benchmark: 96918 bandwidth: 369.7
VkFFT System: 7 128x1048576 Buffer: 1024 MB avg_time_per_step: 10.816 ms std_error: 0.022 num_iter: 3 benchmark: 96946 bandwidth: 369.8
VkFFT System: 8 256x524288 Buffer: 1024 MB avg_time_per_step: 10.918 ms std_error: 0.016 num_iter: 3 benchmark: 96037 bandwidth: 366.4
VkFFT System: 9 512x262144 Buffer: 1024 MB avg_time_per_step: 10.963 ms std_error: 0.045 num_iter: 3 benchmark: 95645 bandwidth: 364.9
VkFFT System: 10 1024x131072 Buffer: 1024 MB avg_time_per_step: 10.990 ms std_error: 0.052 num_iter: 3 benchmark: 95414 bandwidth: 364.0
VkFFT System: 11 2048x65536 Buffer: 1024 MB avg_time_per_step: 11.018 ms std_error: 0.067 num_iter: 3 benchmark: 95172 bandwidth: 363.1
VkFFT System: 12 4096x32768 Buffer: 1024 MB avg_time_per_step: 10.981 ms std_error: 0.080 num_iter: 3 benchmark: 95486 bandwidth: 364.3
VkFFT System: 13 8192x16384 Buffer: 1024 MB avg_time_per_step: 11.638 ms std_error: 1.004 num_iter: 3 benchmark: 90095 bandwidth: 343.7
```

- half precision

```text
$ ./VkFFT_TestSuite -vkfft 2
2 - VkFFT FFT + iFFT C2C benchmark 1D batched in half precision
VkFFT System: 3 8x16777216 Buffer: 512 MB avg_time_per_step: 5.755 ms std_error: 0.054 num_iter: 2 benchmark: 182207 bandwidth: 347.5
VkFFT System: 4 16x8388608 Buffer: 512 MB avg_time_per_step: 5.855 ms std_error: 0.018 num_iter: 2 benchmark: 179090 bandwidth: 341.6
VkFFT System: 5 32x4194304 Buffer: 512 MB avg_time_per_step: 6.394 ms std_error: 0.066 num_iter: 2 benchmark: 163989 bandwidth: 312.8
VkFFT System: 6 64x2097152 Buffer: 512 MB avg_time_per_step: 6.731 ms std_error: 0.127 num_iter: 2 benchmark: 155790 bandwidth: 297.1
VkFFT System: 7 128x1048576 Buffer: 512 MB avg_time_per_step: 7.127 ms std_error: 0.059 num_iter: 2 benchmark: 147137 bandwidth: 280.6
VkFFT System: 8 256x524288 Buffer: 512 MB avg_time_per_step: 5.517 ms std_error: 0.100 num_iter: 2 benchmark: 190062 bandwidth: 362.5
VkFFT System: 9 512x262144 Buffer: 512 MB avg_time_per_step: 5.403 ms std_error: 0.004 num_iter: 2 benchmark: 194084 bandwidth: 370.2
VkFFT System: 10 1024x131072 Buffer: 512 MB avg_time_per_step: 5.575 ms std_error: 0.168 num_iter: 2 benchmark: 188085 bandwidth: 358.7
VkFFT System: 11 2048x65536 Buffer: 512 MB avg_time_per_step: 5.560 ms std_error: 0.126 num_iter: 2 benchmark: 188598 bandwidth: 359.7
VkFFT System: 12 4096x32768 Buffer: 512 MB avg_time_per_step: 5.590 ms std_error: 0.108 num_iter: 2 benchmark: 187580 bandwidth: 357.8
VkFFT System: 13 8192x16384 Buffer: 512 MB avg_time_per_step: 10.461 ms std_error: 1.182 num_iter: 2 benchmark: 100241 bandwidth: 191.2
```

