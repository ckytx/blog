---
title: "FFmpeg视频转码"
date: 2020-07-28T10:38:53+08:00
tags: [ffmpeg]
categories: [ffmpeg]
---
<!--more-->
由于iPhone拍摄视频尺寸较大，所以将视频转码后保存
### 转码命令
```shell
# transcode cpu
ffmpeg -noautorotate -i xxxx.mov -c:v h264 -b:v 8500k -s 1920x1080 -c:a aac -b:a 128k -map_metadata 0 out.mp4

# vaapi
ffmpeg -hwaccel vaapi -vaapi_device /dev/dri/renderD128 -i in.mov -vf 'format=nv12,hwupload,scale_vaapi=w=1920:h=1080' -c:v h264_vaapi -b:v 8500k -c:a aac -b:a 128k -map_metadata 0 out-vaapi.mp4

# qsv Hevc qsv decode + qsv scaling to h264 qsv encode
ffmpeg -init_hw_device qsv=hw -filter_hw_device hw -c:v hevc_qsv -i in.mov -vf hwupload=extra_hw_frames=64,format=qsv,scale_qsv=w=1920:h=1080 -c:v h264_qsv -b:v 8500k -c:a aac -b:a 128k -map_metadata 0 out-qsv.mp4
```

### 视频拼接
```shell
# list.txt
file 1.mov
file 2.mov
file 3.mov
...
# 拼接，注意视频格式要一致
ffmpeg -f concat -i concat.txt -c copy -metadata creation_time="2020-07-28T02:32:39.000000Z" concat-out.mp4
```

### 常用参数说明

```
ffmpeg [global_options] {[input_file_options] -i input_url} ... {[output_file_options] output_url} ...
```

| option | desc | example |
| ----- | ---- | ---- |
| -noautorotate | 忽略元数据中的rotate，配合保留元数据 ||
| -i | 要转码的文件 | in.mov |
| -c:v | 视频编码 | h264, hevc, h264_vaapi... |
| -b:v | 视频比特率 | 1080p - 8500k, 720p - 3500k, 480p - 1800k |
| -s | 分辨率 | 1920x1080, vaapi/qsv在-vf参数中设置 |
| -r | 帧率 | 60, 30, 24... |
| -c:a | 音频编码 | aac, mp3... |
| -b:a | 音频比特率 | 128k |
| -metadata | 设置元数据 | creation_time="2020-07-28T02:32:39.000000Z" |
| -map_metadata | 元数据 | 为 0 保留全部元数据，详细用法参考ffmpeg文档 |
| -y | 直接覆盖输出文件 ||

### 参考

- [http://ffmpeg.org/ffmpeg.html]()
- [https://einverne.github.io/post/2015/12/ffmpeg-first.html]()