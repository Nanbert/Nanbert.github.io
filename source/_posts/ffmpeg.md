---
title: ffmpeg
date: 2024-04-07 04:03:15
tags:
---
# 视频剪裁
- 裁剪:`ffmpeg -i xx.mp4 -vcodec copy -acodec copy -ss 00:00:00 -to 01:18:08 output.mp4`
# 合并:
## 通用
 先建立个文本文档file,格式如下:
```bash
file '1.mp4'
file '2.mp4'
```
`ffmpeg -f concat -i file -c copy output.mkv`
或者支持不好先转换ts
`ffmpeg -i 1.mp4 -vcodec copy -acodec copy -vbsf h264_mp4toannexb 1.ts`

`ffmpeg -i 2.mp4 -vcodec copy -acodec copy -vbsf h264_mp4toannexb 2.ts`

`ffmpeg -i "concat:1.ts|2.ts" -acodec copy -vcodec copy -absf aac_adtstoasc output.mp4`
## 重新编码合并
适用不同编码
`ffmpeg -i input1.mp4 -i input2.webm -i input3.avi -filter_complex '[0:0] [0:1] [1:0] [1:1] [2:0] [2:1] concat=n=3:v=1:a=1 [v] [a]' -map '[v]' -map '[a]' <编码器选项> output.mkv`
[0:0] [0:1] [1:0] [1:1] [2:0] [2:1] 分别表示第一个输入文件的视频、音频、第二个输入文件的视频、音频、第三个输入文件的视频、音频。concat=n=3:v=1:a=1 表示有三个输入文件，输出一条视频流和一条音频流。[v] [a] 就是得到的视频流和音频流的名字，注意在 bash 等 shell 中需要用引号，防止通配符扩展。
# 视频倒放
## 视频倒放，无音频
`ffmpeg -i 123.mp4 -filter_complex [0:v]reverse[v] -map [v] -preset superfast out.mp4 `
## 视频倒放，音频不变
`ffmpeg -i 123.mp4 -vf reverse out.mp4`
## 音频倒放，视频不变
`ffmpeg -i 123.mp4 -map 0 -c:v copy -af "areverse" out.mp4`
## 音视频同时倒放
`fmpeg -i 123.mp4 -vf reverse -af areverse -preset superfast out.mp4`


# 视频格式转换
- rmvb->mp4:
`ffmpeg -i name1.rmvb -c:v libx264 -strict -2 name2.mp4 `
# 字幕
- 添加字幕:
`ffmpeg -i 2020-07-13\ 08-20-26.mkv -vf subtitles=test.srt -y output.mkv`
srt格式:
```
1
00:00:03,000 --> 00:00:06,000
Hi,I am Nanbert Don De Niro

2
00:00:06,000 --> 00:00:08,000
Hi,I am Donald Trump
  
3
00:00:08,444 --> 00:00:10,000
It's you,Assole!
```
# 视频与声音
- 静音:`ffmpeg -i 10.mp4 -af "volume=0" 10Silent.mp4`
- 静音一部分:`ffmpeg -i 10.mp4 -af "volume=enable='between(t,0,8)':volume=0" 10Silent.mp4`
# 流媒体
- 一边播放一边保存流媒体:`ffmpeg -i host/input.m3u8 -c copy out.mkv -c copy -f matroska - | ffplay - `
# 音频
- 波形图模式：`ffplay -showmode 1 xx.mp3`
- 频谱图模式：`ffplay -showmode 2 xx.mp3`
- 音频淡出效果: `ffmpeg -i xx.mp3 -filter_complex afade=t=out:st=16:d=4 xx2.mp3`

