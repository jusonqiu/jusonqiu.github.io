---
title: ffmpeg 解析SDP
---

使用ffmpeg直接打开sdp文件播放里面流媒体.

# 0x0 使用Custom I/O
ffmpeg 有一个非常好用的模块: AVIO, 这个模块是用ffmpeg Custom I/O 的主要接口,来看API;

- 结构体
	- AVIOInterruptCB : Callback for checking whether to abort blocking functions
	- AVIODirEntry
	- AVIODirContext
	- AVIOContext : Bytestream IO Context,

- 常用函数
	- avio_open, avio_open2
	- avio_alloc_context,
	- avio_read
	- avio_write

这些函数的详细使用可以参考官方示例:[http_multiclinet.c](https://www.ffmpeg.org/doxygen/3.0/http_multiclient_8c-example.html#a17) 和 [官方文档](https://www.ffmpeg.org/doxygen/3.0/avio_8h.html#a853f5149136a27ffba3207d8520172a5)

# 解析SDP

## 初始话必要的网络和解码模块
```
av_register_all()
avformat_network_init()
```
## Alloc AVIO Context
```
unsigned char *iobuffer; // sdp
AVIOContext *avio = avio_alloc_context(iobuffer, sdp_size, 0, (void*)NULL, NULL, NULL,NULL);

// 把io context 设置到 avformat context 上去,
//这样ffmpeg就进入了Custom模式;
//详细查看源码里面类是 if (formatCtx->pb != NULL) { //custom io }

pFormatCtx->pb = avio
```
## Demuxer, Decoder

```

pFormatCtx->iformat = av_find_input_format("sdp");

int err = avformat_open_input(&pFormatCtx, "nothing", NULL, NULL);
if(err) {
printf("avformat_open_input error: %d\n", err);
return -1;
}

// Retrieve stream information
if(avformat_find_stream_info(pFormatCtx, NULL)<0) {
printf("avformat_find_stream_info error!!!\n");
return -1; // Couldn't find stream information
}
```

## 完整示例
```
#include <libavcodec/avcodec.h>
#include <libavformat/avformat.h>
#include <libswscale/swscale.h>
#include <SDL/SDL.h>
#include <SDL/SDL_thread.h>
#include <stdio.h>
#include <assert.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <unistd.h>

#if LIBAVCODEC_VERSION_INT < AV_VERSION_INT(55,28,1)
#define av_frame_alloc avcodec_alloc_frame
#define av_frame_free avcodec_free_frame
#endif

#include <string.h>

int read_sdp(void *p, uint8_t *buf, int bufsize){
    char *file = (char *)p;
    char *tmp = NULL;
    struct stat buffer;
    int ret = 0;
    int fd = open(file, O_RDONLY);
    if (fd < 0) goto end;
    int status = fstat(fd, &buffer);
    if (status < 0) goto end;
    if (buffer.st_size > 0 && bufsize >= buffer.st_size){
        tmp = av_mallocz(buffer.st_size+1);
        ret = read(fd, tmp, buffer.st_size);
        if (ret != buffer.st_size){
            printf("read file error\n");
            av_free(tmp);
            goto end;
        }
        tmp[buffer.st_size] = '\0';
        strcpy(buf, tmp);
        av_free(tmp);
    }else{
        goto end;
    }
    printf("SDP:\n%s\n", buf);
    if (fd > 0) close(fd);
    return strlen(buf);
end:
    if (fd > 0) close(fd);
    return 0;
}


int main(int argc, char *argv[]) {
  AVFormatContext *pFormatCtx = NULL;
  int             i, videoStream;
  AVCodecContext  *pCodecCtxOrig = NULL;
  AVCodecContext  *pCodecCtx = NULL;
  AVCodec         *pCodec = NULL;
  AVFrame         *pFrame = NULL;
  AVPacket        packet;
  int             frameFinished;
  float           aspect_ratio;
  struct SwsContext *sws_ctx = NULL;

  AVInputFormat *piFmt = NULL;

  if (argc != 2){
      printf("Usage: %s <file>\n", argv[0]);
      return -1;
  }

  char *file = strdup(argv[1]);

  SDL_Overlay     *bmp;
  SDL_Surface     *screen;
  SDL_Rect        rect;
  SDL_Event       event;

  av_register_all();
  avformat_network_init();
  av_log_set_level(AV_LOG_DEBUG);

  if(SDL_Init(SDL_INIT_VIDEO | SDL_INIT_AUDIO | SDL_INIT_TIMER)) {
    fprintf(stderr, "Could not initialize SDL - %s\n", SDL_GetError());
    exit(1);
  }

  pFormatCtx = NULL;
  pFormatCtx = avformat_alloc_context();
  unsigned char * iobuffer = (unsigned char *)av_malloc(32768);

  int size = read_sdp((void*)file, iobuffer, 32768);

  AVIOContext * avio = avio_alloc_context(iobuffer, size, 0, (void*)NULL, NULL, NULL, NULL);
  pFormatCtx->pb = avio;

  if(!avio) {
      printf("avio_alloc_context error!!!\n");
      return -1;
  }

#if 0
  if(av_probe_input_buffer(avio, &piFmt, "", NULL, 0, 0) < 0) {
      printf("av_probe_input_buffer error!\n");
      return -1;
  } else {
      printf("probe success\n");
      printf("format: %s[%s]\n", piFmt->name, piFmt->long_name);
  }
#endif
  pFormatCtx->pb = avio;
  pFormatCtx->iformat = av_find_input_format("sdp");

  int err = avformat_open_input(&pFormatCtx, "nothing", NULL, NULL);
  if(err) {
      printf("avformat_open_input error: %d\n", err);
      return -1;
  }

  // Retrieve stream information
  if(avformat_find_stream_info(pFormatCtx, NULL)<0) {
      printf("avformat_find_stream_info error!!!\n");
    return -1; // Couldn't find stream information
  }

  av_dump_format(pFormatCtx, 0, "", 0);

  videoStream=-1;
  for(i=0; i<pFormatCtx->nb_streams; i++)
    if(pFormatCtx->streams[i]->codec->codec_type==AVMEDIA_TYPE_VIDEO) {
      videoStream=i;
      break;
    }
  if(videoStream==-1) {
      printf("videoStream error!!!\n");
    return -1;
  }

  pCodecCtxOrig=pFormatCtx->streams[videoStream]->codec;

  pCodec=avcodec_find_decoder(pCodecCtxOrig->codec_id);
  if(pCodec==NULL) {
    fprintf(stderr, "Unsupported codec!\n");
    return -1; // Codec not found
  }

  // Copy context
  pCodecCtx = avcodec_alloc_context3(pCodec);
  if(avcodec_copy_context(pCodecCtx, pCodecCtxOrig) != 0) {
    fprintf(stderr, "Couldn't copy codec context");
    return -1; // Error copying codec context
  }

  // Open codec
  if(avcodec_open2(pCodecCtx, pCodec, NULL)<0) {
      printf("avcodec_open2 error!!!\n");
    return -1; // Could not open codec
  }

  pFrame=av_frame_alloc();

  printf("Everything OK\n");

#ifndef __DARWIN__
        screen = SDL_SetVideoMode(pCodecCtx->width, pCodecCtx->height, 0, 0);
#else
        screen = SDL_SetVideoMode(pCodecCtx->width, pCodecCtx->height, 24, 0);
#endif
  if(!screen) {
    fprintf(stderr, "SDL: could not set video mode - exiting\n");
    exit(1);
  }

  // Allocate a place to put our YUV image on that screen
  bmp = SDL_CreateYUVOverlay(pCodecCtx->width,
                 pCodecCtx->height,
                 SDL_YV12_OVERLAY,
                 screen);

  // initialize SWS context for software scaling
  sws_ctx = sws_getContext(pCodecCtx->width,
               pCodecCtx->height,
               pCodecCtx->pix_fmt,
               pCodecCtx->width,
               pCodecCtx->height,
               AV_PIX_FMT_YUV420P,
               SWS_BILINEAR,
               NULL,
               NULL,
               NULL
               );

  i=0;
  while(av_read_frame(pFormatCtx, &packet)>=0) {
    if(packet.stream_index==videoStream) {
      avcodec_decode_video2(pCodecCtx, pFrame, &frameFinished, &packet);

      if(frameFinished) {
          SDL_LockYUVOverlay(bmp);

          AVPicture pict;
          pict.data[0] = bmp->pixels[0];
          pict.data[1] = bmp->pixels[2];
          pict.data[2] = bmp->pixels[1];

          pict.linesize[0] = bmp->pitches[0];
          pict.linesize[1] = bmp->pitches[2];
          pict.linesize[2] = bmp->pitches[1];

          // Convert the image into YUV format that SDL uses
          sws_scale(sws_ctx, (uint8_t const * const *)pFrame->data,
                    pFrame->linesize, 0, pCodecCtx->height,
                    pict.data, pict.linesize);

          SDL_UnlockYUVOverlay(bmp);

          rect.x = 0;
          rect.y = 0;
          rect.w = pCodecCtx->width;
          rect.h = pCodecCtx->height;
          SDL_DisplayYUVOverlay(bmp, &rect);

      }
    }

    av_free_packet(&packet);
    SDL_PollEvent(&event);
    switch(event.type) {
    case SDL_QUIT:
      SDL_Quit();
      exit(0);
      break;
    default:
      break;
    }
  }

  if (file != NULL) free(file);
  // Free the YUV frame
  av_frame_free(&pFrame);

  // Close the codec
  avcodec_close(pCodecCtx);
  avcodec_close(pCodecCtxOrig);

  // Close the video file
  avformat_close_input(&pFormatCtx);
  return 0;
}
```
