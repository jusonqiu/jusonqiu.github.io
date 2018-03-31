---
layout: "post"
title: "YUV RGB Formats"
date: "2017-08-26 18:29"
---

# YUV 数据存放分类

YUV格式有两大类：planar和packed。

* 对于planar的YUV格式，先连续存储所有像素点的Y，紧接着存储所有像素点的U，随后是所有像素点的V
*对于packed的YUV格式，每个像素点的Y,U,V是连续交错存储的。

YUV，分为三个分量，“Y”表示明亮度（Luminance或Luma），也就是灰度值；而“U”和“V” 表示的则是色度（Chrominance或Chroma），作用是描述影像色彩及饱和度，用于指定像素的颜色

>    与我们熟知的RGB类似，YUV也是一种颜色编码方法，主要用于电视系统以及模拟视频领域，它将亮度信息（Y）与色彩信息（UV）分离，没有UV信息一样可以显示完整的图像，只不过是黑白的，这样的设计很好地解决了彩色电视机与黑白电视的兼容问题。并且，YUV不像RGB那样要求三个独立的视频信号同时传输，所以用YUV方式传送占用极少的频宽。

YUV码流的存储格式其实与其采样的方式密切相关，主流的采样方式有三种，YUV4:4:4，YUV4:2:2，YUV4:2:0，关于其详细原理，可以通过网上其它文章了解，这里我想强调的是如何根据其采样格式来从码流中还原每个像素点的YUV值，因为只有正确地还原了每个像素点的YUV值，才能通过YUV与RGB的转换公式提取出每个像素点的RGB值，然后显示出来。

    用三个图来直观地表示采集的方式吧，以黑点表示采样该像素点的Y分量，以空心圆圈表示采用该像素点的UV分量。

![这里写图片描述](/images/yuv-rgb-formats-20161020195207914.png)

> **说明:**
>YUV 4:4:4采样，每一个Y对应一组UV分量。
>YUV 4:2:2采样，每两个Y共用一组UV分量。
>YUV 4:2:0采样，每四个Y共用一组UV分量。

# 存储方式

下面我用图的形式给出常见的YUV码流的存储方式，并在存储方式后面附有取样每个像素点的YUV数据的方法，其中，Cb、Cr的含义等同于U、V。

 1. YUVY 格式 （属于YUV422）
 YUYV为YUV422采样的存储格式中的一种，相邻的两个Y共用其相邻的两个Cb、Cr，分析，对于像素点Y'00、Y'01 而言，其Cb、Cr的值均为 Cb00、Cr00，其他的像素点的YUV取值依次类推。
![这里写图片描述](/images/yuv-rgb-formats-20161020195506985.png)
 2. UYVY 格式 （属于YUV422）
 UYVY格式也是YUV422采样的存储格式中的一种，只不过与YUYV不同的是UV的排列顺序不一样而已，还原其每个像素点的YUV值的方法与上面一样。
 ![这里写图片描述](/images/yuv-rgb-formats-20161020195750158.png)

 3. YUV422P（属于YUV422）
 YUV422P也属于YUV422的一种，它是一种Plane模式，即平面模式，并不是将YUV数据交错存储，而是先存放所有的Y分量，然后存储所有的U（Cb）分量，最后存储所有的V（Cr）分量，如上图所示。其每一个像素点的YUV值提取方法也是遵循YUV422格式的最基本提取方法，即两个Y共用一个UV。比如，对于像素点Y'00、Y'01 而言，其Cb、Cr的值均为 Cb00、Cr00。
 ![这里写图片描述](/images/yuv-rgb-formats-20161020195849705.png)

 4. YV12，YU12格式（属于YUV420）
 YU12和YV12属于YUV420格式，也是一种Plane模式，将Y、U、V分量分别打包，依次存储。其每一个像素点的YUV数据提取遵循YUV420格式的提取方式，即4个Y分量共用一组UV。注意，上图中，Y'00、Y'01、Y'10、Y'11共用Cr00、Cb00，其他依次类推。
 ![这里写图片描述](/images/yuv-rgb-formats-20161020195948581.png)

 5. NV12、NV21（属于YUV420）
 NV12和NV21属于YUV420格式，是一种two-plane模式，即Y和UV分为两个Plane，但是UV（CbCr）为交错存储，而不是分为三个plane。其提取方式与上一种类似，即Y'00、Y'01、Y'10、Y'11共用Cr00、Cb00;
 ![这里写图片描述](/images/yuv-rgb-formats-20161020200102004.png)

6. 其他
```
I420: YYYYYYYY UU VV  => YUV420P
YV12: YYYYYYYY VV UU  => YUV420P
NV12: YYYYYYYY UV UV  => YUV420SP
NV21: YYYYYYYY VU VU  => YUV420SP
```

## 空间及转换实例
**例如:**
YUV420 planar数据， 以720×488大小图象YUV420 planar为例，其存储格式是： 共大小为**(720×480×3>>1) 字节**，分为三个部分:Y,U和V

| 分量 | 大小 | 范围|
|:----:|:-----:|:----:|
|Y|720×480|0 ~ 720×480|
|U(Cb)|720×480>>2|720×480 ~ 720×480×5/4|
|V(Cr)|720×480>>2|720×480×5/4 ~ 720×480×3/2|

三个部分内部均是行优先存储，三个部分之间是Y,U,V 顺序存储。

** 例如 : **
  4 ：2： 2 和 4：2：0  转换, YUV4:2:2 ---> YUV4:2:0  Y不变，将U和V信号值在行(垂直方向)在进行一次隔行抽样。 YUV4:2:0 ---> YUV4:2:2  Y不变，将U和V信号值的每一行分别拷贝一份形成连续两行数据.

在YUV420中，一个像素点对应一个Y，一个4X4的小方块对应一个U和V。对于所有YUV420图像，它们的Y值排列是完全相同的，因为只有Y的图像就是灰度图像。YUV420sp与YUV420p的数据格式它们的UV排列在原理上是完全不同的。420p它是先把U存放完后，再存放V，也就是说UV它们是连续的。而420sp它是UV、UV这样交替存放的。(见下图) 有了上面的理论，我就可以准确的计算出一个YUV420在内存中存放的大小。 width * hight =Y（总和） U = Y / 4   V = Y / 4.

**例如:**
假设一个分辨率为8X4的YUV图像，它们的格式如下图：

` YUV420sp格式如下图: `
![ YUV420sp格式如下图 ](/images/yuv-rgb-formats-20161020201217929.png)

`  YUV420p数据格式如下图:`
![  YUV420p数据格式如下图](/images/yuv-rgb-formats-20161020201347988.png)

**例如: 旋转90度的算法**
```c
public static void rotateYUV240SP(byte[] src,byte[] des,int width,int height){
  int wh = width * height;
  //旋转Y
  int k = 0;
  for(int i=0;i<width;i++) {
   for(int j=0;j<height;j++)
   {
         des[k] = src[width*j + i];   
         k++;
   }
  }
  // 旋转UV
  for(int i=0;i<width;i+=2) {
   for(int j=0;j<height/2;j++)
   {
               des[k] = src[wh+ width*j + i];
               des[k+1]=src[wh + width*j + i+1];
         k+=2;
   }
  }
 }
```

> from [http://blog.csdn.net/qzs_kaka/article/details/52831378](http://blog.csdn.net/qzs_kaka/article/details/52831378)
