---
title: ffmpeg command
category: 多媒体开发
date: 2016-07-15 16:45:13
tags: [ffmpeg]
---

# ffmpeg常用参数一览

## 基本选项

|选项|说明 |
|:------|:----------|
|-formats |输出所有可用格式 |
|-f fmt   |指定格式(音频或视频格式) |
|-formats |输出所有可用格式 |
|-i filename |指定输入文件名，在linux下当然也能指定:0.0(屏幕录制)或摄像头 |
|-y  | 覆盖已有文件 |
|-t duration |记录时长为t   |
|-fs limit_size     |设置文件大小上限|
|-ss time_off |从指定的时间(s)开始， [-]hh:mm:ss[.xxx]的格式也支持|
|-itsoffset time_off | 设置时间偏移(s)，该选项影响所有后面的输入文件。该偏移被加到输入文件的时戳，定义一个正偏移意味着相应的流被延迟了 offset秒。 [-]hh:mm:ss[.xxx]的格式也支持 |
|-title string |标题 |
|-timestamp time |时间戳 |
|-author string |作者 |
|-copyright string |版权信息 |
|-comment string |评论 |
|-album string |album名 |
|-v verbose    | 与log相关的 |
|-target type |设置目标文件类型("vcd", "svcd", "dvd", "dv", "dv50", "pal-vcd", "ntsc-svcd", ...) |
|-dframes number|设置要记录的帧数 |
|:------|:----------|

## 视频选项

|选项|说明 |
|:------|:----------|
|-b |指定比特率(bits/s)，似乎ffmpeg是自动VBR的，指定了就大概是平均比特率 |
|-vb |指定视频比特率(bits/s) |
|-vframes number | 设置转换多少桢(frame)的视频 |
|-r rate |桢速率(fps) |
|-s size|分辨率 |
|-aspect aspect |设置视频长宽比(4:3, 16:9 or 1.3333, 1.7777) |
|-croptop size|设置顶部切除尺寸(in pixels) |
|-cropbottom size|设置底部切除尺寸(in pixels) |
|-cropleft size     |设置左切除尺寸 (in pixels) |
|-cropright size|设置右切除尺寸 (in pixels) |
|-padtop size|设置顶部补齐尺寸(in pixels) |
|-padbottom size|底补齐(in pixels) |
|-padleft size|左补齐(in pixels) |
|-padright size|右补齐(in pixels) |
|-padcolor color|补齐带颜色(000000-FFFFFF) |
|-vn|取消视频 |
|-vcodec codec|强制使用codec编解码方式('copy' to copy stream) |
|-sameq   |使用同样视频质量作为源（VBR） |
|-pass n|选择处理遍数（1或者2）。两遍编码非常有用。第一遍生成统计信息，第二遍生成精确的请求的码率 |
|-passlogfile file|选择两遍的纪录文件名为file |
|-newvideo|在现在的视频流后面加入新的视频流 |

## 高级视频选项

|选项|说明 |
|:------|:----------|
|-pix_fmt format |set pixel format, 'list' as argument shows all the pixel formats supported |
|-intra |仅适用帧内编码 |
|-qscale q |以<数值>质量为基础的VBR，取值0.01-255，约小质量越好 |
|-loop_input|设置输入流的循环数(目前只对图像有效) |
|-loop_output|设置输出视频的循环数，比如输出gif时设为0表示无限循环 |
|-g int|设置图像组大小 |
|-cutoff int|设置截止频率 |
|-qmin int|设定最小质量 |
|-qmax int|设定最大质量 |
|-qdiff int|量化标度间最大偏差 (VBR) |
|-bf int|使用frames B 帧，支持mpeg1,mpeg2,mpeg4 |

## 音频选项

|选项|说明 |
|:------|:----------|
|-ab|设置比特率(单位：bit/s，也许老版是kb/s) |
|-aframes number|设置转换多少桢(frame)的音频 |
|-aq quality|设置音频质量 (指定编码) |
|-ar rate|设置音频采样率 (单位：Hz) |
|-ac channels|设置声道数 |
|-an|取消音频 |
|-acodec codec|指定音频编码('copy' to copy stream) |
|-vol volume|设置录制音量大小(默认为256) |
|-newaudio|在现在的音频流后面加入新的音频流 |

## 字幕选项

|选项|说明 |
|:------|:----------|
|-sn|取消字幕 |
|-scodec codec|设置字幕编码('copy' to copy stream) |
|-newsubtitle|在当前字幕后新增 |
|-slang code|设置字幕所用的ISO 639编码(3个字母) |
|Audio/Video|抓取选项: |
|-vc channel|设置视频捕获通道(只对DV1394) |
|-tvstd standard|设置电视标准 NTSC PAL(SECAM) |

# Stream
## UDP
1. 发送H.264裸流至组播地址
> 注：组播地址指的范围是224.0.0.0—239.255.255.255

```bash
ffmpeg -re -i test.h264 -vcodec copy -f h264 udp://233.233.233.223:8000
#注1：-re一定要加，代表按照帧率发送，否则ffmpeg会一股脑地按最高的效率发送数据。
#注2：-vcodec copy要加，否则ffmpeg会重新编码输入的H.264裸流。
```
2.播放承载H.264裸流的UDP

```bash
ffplay -f h264 udp://233.233.233.223:8000
# 注：需要使用-f说明数据类型是H.264
ffplay -max_delay 100000 -f h264 udp://233.233.233.223:8000  
#将-max_delay设置为100ms
```
3.发送MPEG2裸流至组播地址

```bash

ffmpeg -re -i test.h264 -vcodec mpeg2video -f mpeg2video udp://233.233.233.223:8000
#读取本地摄像头的数据，编码为MPEG2，发送至地址udp://233.233.233.223:8000。
```
4.播放MPEG2裸流

```bash
#指定-vcodec为mpeg2video即可。
ffplay -vcodec mpeg2video udp://233.233.233.223:8000
```
## RTP
1.发送H.264裸流至组播地址。
最右边的“>test.sdp”用于将ffmpeg的输出信息存储下来形成一个sdp文件。该文件用于RTP的接收。当不加“>test.sdp”的时候，ffmpeg会直接把sdp信息输出到控制台。将该信息复制出来保存成一个后缀是.sdp文本文件，也是可以用来接收该RTP流的。加上“>test.sdp”后，可以直接把这些sdp信息保存成文本。

```bash
#发送H.264裸流“test.h264”至地址rtp://233.233.233.223:8000
ffmpeg -re -i test.h264 -vcodec copy -f rtp rtp://233.233.233.223:8000>test.sdp
```
2.播放承载H.264裸流的RTP

```bash
ffplay test.sdp  
```
## RTMP
1. 发送H.264裸流至RTMP服务器（FlashMedia Server，Red5等）
面命令实现了发送H.264裸流“test.h264”至主机为localhost，Application为oflaDemo，Path为livestream的RTMP URL.

```bash
ffmpeg -re -i test.h264 -vcodec copy -f flv rtmp://localhost/oflaDemo/livestream  
```
2. 播放RTMP
ffplay播放的RTMP URL最好使用双引号括起来，并在后面添加live=1参数，代表实时流。实际上这个参数是传给了ffmpeg的libRTMP的。
```bash
ffplay “rtmp://localhost/oflaDemo/livestream live=1”  
```
[ffmpeg处理RTMP流媒体的命令大全](http://blog.csdn.net/leixiaohua1020/article/details/12029543)

## 测延时

1.测延时
测延时有一种方式，即一路播放发送端视频，另一路播放流媒体接收下来的流。播放发送端的流有2种方式：FFmpeg和FFplay。
通过FFplay播放是一种众所周知的方法，例如：

```bash
ffplay -f dshow -i video="摄像头名称"  
```
即可播放本地名称为“Integrated Camera”的摄像头。
此外通过FFmpeg也可以进行播放，通过指定参数“-f sdl”即可。例如：
```bash
#一边通过SDL播放视频，一边发送视频流至RTMP服务器
ffmpeg -re -i test.h264 -pix_fmt yuv420p –f sdl xxxx.yuv -vcodec copy -f flv rtmp://localhost/oflaDemo/livestream  
```
## VlC
使用vlc 串流到vlc.
```
#/bin/bash

if [ ! -f "$1" ]; then
        echo "usage: $0 <media file> [port] "

elif [ ! -z "$2" ]; then

        echo "stream $1 to rtsp://192.168.8.155:$2/stream"
        cvlc  $1 --sout "#rtp{dst=192.168.8.155,port=1234,sdp=rtsp://192.168.8.155:$2/stream}" --loop

else
        echo "stream $1 to rtsp://192.168.8.155:8080/stream"
        cvlc  $1 --sout '#rtp{dst=192.168.8.155,port=1234,sdp=rtsp://192.168.8.155:8080/stream}' --loop

        #cvlc -vvv $1 --sout '#rtp{dst=192.168.8.155,port=1234,sdp=rtsp://192.168.8.155:8080/test.sdp}'
fi

```

# Encode Decode Muxer Demuxer

## Raw Video
```
# decode
ffmpeg -i  input.mp4  -c:v rawvideo -pix_fmt yuv420p output.yuv
#show
ffplay -f rawvideo -video_size 480x360  output.yuv
```

# TS 转码

1. 直接把媒体文件转为ts

```shell
ffmpeg -i cat.mp4 -c copy -bsf h264_mp4toannexb cat.ts
```

2. 使用segment参数进行切片

```shell
ffmpeg -i cat.ts -c copy -map 0 -f segment -segment_list playlist.m3u8 -segment_time 2 cat_output%03d.ts

```

3. ffmpeg切片命令，以H264和AAC的形式对视频进行输出

```shell
ffmpeg -i input.mp4 -c:v libx264 -c:a aac -strict -2 -f hls output.m3u8
```

>-hls_time n: 设置每片的长度，默认值为2。单位为秒
>-hls_list_size n:设置播放列表保存的最多条目，设置为0会保存有所片信息，默认值为5
>-hls_wrap n:设置多少片之后开始覆盖，如果设置为0则不会覆盖，默认值为0.这个选项能够避免在磁盘上存储过多的片，而且能够限制写入磁盘的最多的片的数量
>-hls_start_number n:设置播放列表中sequence number的值为number，默认值为0

4. 对ffmpeg切片指令的使用

```shell
ffmpeg -i output.mp4 -c:v libx264 -c:a aac -strict -2 -f hls -hls_list_size 0 -hls_time 5 data/output.m3u8
```
