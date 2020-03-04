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



# 下载视频

国外影视，推荐在`1337x.to`上下载，挺全的。
在线视频用`Youtube-dl`工具下载
```bash
#看支持下载的文件列表
youtube-dl -F  https://www.bilibili.com/video/av36793752
#下載視頻文件，默认下载标识best的视频
youtube-dl url
```

## 参考
>1. `Using_FFmpeg_with_NVIDIA_GPU_Hardware_Acceleration`: https://developer.download.nvidia.com/designworks/ffmpeg/secure/Using_FFmpeg_with_NVIDIA_GPU_Hardware_Acceleration_v01.4.pdf
>2. `HWAccelIntro`: https://trac.ffmpeg.org/wiki/HWAccelIntro