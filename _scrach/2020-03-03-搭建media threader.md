---
title: 搭建家庭影院
categories: others
---

# 方案选择

先尝试了DLNA（DIGITAL LIVING NETWORK ALLIANCE），其为数字媒体的共享提供了解决方案。但是海信电视dlna的实现有问题，老出各种问题；而且DLNA只支持同网段发现。

于是干脆搭建媒体服务器，主流的有`plex`，`emby`，`jellyfin`。作为程序员，当然选择免费开源的`jellyfin`了。

# 使用

安装使用跟着`quick start`一气呵成，还是比较简单。但是如果要进阶使用，缺需要查看很多资料了。
- 修改`datadir`。jellyfin将`datadir`存放在不同的地方，所以想要修改有点麻烦，具体顺序需要搜索源码。

## 元数据

元数据可以存在`.nfo`文件中，`xml`格式，可以直接文本打开。

自动补全影片的元数据。中文一般称作`搜刮器`，根据文件名，在web上匹配并下载对应的简介、封面、演员介绍、字幕等。所有工具主要从`TheMovieDB.org`, `Imdb.com`，`TheTvDB.com`下载元数据。因为jelly自带的搜刮器并不准确，所以需要借助第三方的软件。
- [tinymediamanager](https://www.tinymediamanager.org/)开源工具，能更准确的匹配元数据，但是在电影目录下新生成文件和只有国外网站的搜刮。
- [javsdt](https://github.com/junerain123/javsdt)和[AV_Data_Captur](https://github.com/yoshiko2/)github开源工具，主要抓取av类型的元数据，目前选用capur，javsdt的介绍目前失败。



## 编码

对服务器的负载从低到高
- 直接播放，直接传输文件到client。
- Remux：Changes the container but leaves both audio and video streams untouched.
- Direct Stream: Transcodes audio but leaves original video untouched.
- Transcode: Transcodes the video stream.


`H.264`是一种视频编码。`H.264`通常封装成`.mp4`,`.ts`。
下载支持NVIDIA GPU 加速的`FFmpeg`。运行`FFmpeg`看是否有`--enable-cuda-sdk`。
### jellyfin

在播放里面开启硬件加速。
```bash
# vuvid硬件加速，h264_cuvid视频解码，拷贝音频，h_264_nvenc视频编码，-b:v video bitrate
ffmpeg -y -vsync 0 -hwaccel cuvid -c:v h264_cuvid -i input.mp4 -c:a copy -c:v h264_nvenc -b:v 5M output.mp4
```

## 字幕

有些视频自带字幕，不能隐藏，更改，因为这些字幕是压制进去的，像水印样无法轻易去除，常见于`MP4`、`rmvb`格式等。所以尽量找找不包含字幕的视频，俗称生肉。

寻找合适字幕的时候，需要注意它的命名`Breaking.Bad.S03E04.Green.Light.HDTV.XviD-FQM`。`HDTV`：来源于高清电视，`XviD`：视频编码，`FQM`：制作组。能和视频文件匹配是最完美的。

但是往往命名都是十分糟糕的。

## 下载视频

国外影视，推荐在`1337x.to`上下载，挺全的。
在线视频用`Youtube-dl`工具下载
```bash
#看支持下载的文件列表
youtube-dl -F  https://www.bilibili.com/video/av36793752
#下載視頻文件，默认下载标识best的视频
youtube-dl url
```

## 外网访问

成都电信，打10000号就可以让客户开通公网ip了（2020年2月）。然后获得光猫的超级账号，开启upnp。DDNS用http://freedns.afraid.org的，python定时请求更新ip地址就行了。

## 参考
>1. `Using_FFmpeg_with_NVIDIA_GPU_Hardware_Acceleration`: https://developer.download.nvidia.com/designworks/ffmpeg/secure/Using_FFmpeg_with_NVIDIA_GPU_Hardware_Acceleration_v01.4.pdf
>2. `HWAccelIntro`: https://trac.ffmpeg.org/wiki/HWAccelIntro