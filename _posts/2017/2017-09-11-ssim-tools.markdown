---
layout: "post"
title: "ssim tools"
date: "2017-09-11 15:17"
---

## VQMT

[**VQMT**](https://github.com/Rolinh/VQMT) 是 GitHub 上的一个开源项目，使用 C++ 编写的一个 SSIM、MSSIM、PSNR 等算法统计工具。其中依赖 OpenCV，可以直接使用 API 对单帧，多帧或 YUV 文件进行计算。如果没有 ``opencv`` 环境，可以到附近直接下载 vqmt 文件。

- **自带 VQMT 工具使用**

```shell
vqmt <OriginalVideo> <ProcessedVideo> <Height> <Width> <NumberOfFrames> <ChromaFormat> <Output> <Metrics>
```

>
>* OriginalVideo: the original video as raw YUV video file, progressively scanned, and 8 bits per sample
>* ProcessedVideo: the processed video as raw YUV video file, progressively scanned, and 8 bits per sample
>* Height: the height of the video
>* Width: the width of the video
>* NumberOfFrames: the number of frames to process
>* ChromaFormat: the chroma subsampling format. 0: YUV400, 1: YUV420, 2: YUV422, 3: YUV444 Output: the name of the output file(s)
>* Metrics: the list of metrics to use

- **API 快速开始**

 (1) 创建输入流

 ```c++
 VideoYUV::VideoYUV(const char *f, int h, int w, int nbf, int chroma_format)
 //example:
 VideoYUV *yuv1 = new VideoYUV(...);
 VideoYUV *yuv2 = new VideoYUV(...);
 ```
 (2) 创建 SSIM 计算实例

 ```c++
 	SSIM *ssim     = new SSIM(height, width);
 ```

 (3) 逐帧计算

 ```c++
 	cv::Mat original_frame(height,width,CV_32F), processed_frame(height,width,CV_32F);
	for (int frame=0; frame<nbframes; frame++) {

		// Grab frame
		if (!original->readOneFrame() || !processed->readOneFrame())
			exit(EXIT_FAILURE);
		original->getLuma(original_frame, CV_32F);
		processed->getLuma(processed_frame, CV_32F);

		// ssim calc
		result = ssim->compute(original_frame, processed_frame);

		// show
		cout << "frame: "<< frame << " ssim: "<< result << endl;
	}
 ```

## x264 编码器

- **命令行调试**

 ```shell
 x264 [options] --log-level debug --ssim
 ```
- **编程**

 在 x264 源代码 *encoder/encoder.c* 文件 ``#line 3960`` 变量 ``pic_out->prop.f_ssim;`` 的值。

## 使用 ``pyssim``

- **Install**

 ```shell
 pip install pyssim
 ```

- **Running**

 ```shell
 $ pyssim --help
   usage: pyssim [-h] image1.png image path with* or image2.png

  Compares an image with a list of images using the SSIM metric.
  Example:
    pyssim test-images/test1-1.png "test-images/*"

  positional arguments:
  image1.png
  image path with* or image2.png

  optional arguments:
  -h, --help            show this help message and exit
  --cw                  compute the complex wavelet SSIM
  --width WIDTH         scales the image before computing SSIM
  --height HEIGHT       scales the image before computing SSIM

 ```
