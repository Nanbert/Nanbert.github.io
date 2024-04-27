---
title: ffmpeg
date: 2024-04-07 04:03:15
tags:
---
# 视频剪裁
- 裁剪:`ffmpeg -i xx.mp4 -vcodec copy -acodec copy -ss 00:00:00 -to 01:18:08 output.mp4`
## 精确剪切
ffmpeg 会在你输入的时间点附近调整到最接近的关键帧处，这会导致时间不准确，可以先转换为帧内编码（每一帧都是关键帧）
`ffmpeg -i xx.flv -strict -2 -qscale 0 -intra xx2.flv`
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
- 去掉原声:`ffmpeg -i 12.mp4 -c:v copy -an xx.mp4`
- 合并指定音频:`ffmpeg -i out.mp4 -i xx.mp3 xx.mp4`
- 指定合成时间：`ffmpeg -i out.mp4 -i xx.mp3 -t 30 xx.mp4`
- 保留视频原声并添加新音频:`ffmpeg -i xx.mp3 -i 22.mp4 -filter_complex amix=inputs=2 output.mp4`
- 替换原来的音频：`ffmpeg -i xx.wav -i xx.flv -shortest -c copy xx.flv`
- 提取音频：`ffmpeg -i xx.flv -vn -acodec copy xx.wav`
- 调整音量：`ffmpeg -i input.wav -af 'volume=1.5' xx.wav`
# 流媒体
- 一边播放一边保存流媒体:`ffmpeg -i host/input.m3u8 -c copy out.mkv -c copy -f matroska - | ffplay - `
# 音频
- 波形图模式：`ffplay -showmode 1 xx.mp3`
- 频谱图模式：`ffplay -showmode 2 xx.mp3`
- 音频淡出效果: `ffmpeg -i xx.mp3 -filter_complex afade=t=out:st=16:d=4 xx2.mp3`
# GIF
## 视频全部转换成gif
- `ffmpeg -i 123.mp4 out.gif`
## 截取部分片段
- `ffmpeg -t 3 -ss 00:00:02 -i xx.mp4 xx.gif`
## 增加GIF质量
- `ffmpeg -i  OUTPUT_VIDEO.mp4 -b 2048k OUTPUT_VIDEO.gif`:尽可能高质量转换
## 将GIF转成视频
- `ffmpeg -f gif -i xx.gif xx.mp4`:将GIF转换为MP4
## 设置循环次数
默认是0,即无限循环
`ffmpeg -ss 9 -t 5 -i xx.mp4 -loop 2 xx.gif`
## 设置低分辨率
`ffmpeg -ss 9 -t 5 -i 1.mp4 -loop 0 -vf scale=iw/2:-1:flags=lanczos 1.gif`
scale=iw/2:-1:flags=lanczos（lanczos为缩放算法），将会设置gif图片的宽度为源视频一半，高度为比例缩放
## 设置fps(每秒帧数)
`ffmpeg -ss 9 -t 5 -i 1.mp4 -loop 0 -vf "scale=iw/2:-1:flags=lanczos,fps=15" 1.gif`
## 视频画面的一半设成gif
`ffmpeg -ss 9 -t 5 -i 1.mp4 -loop 0 -vf "scale=iw/2:-1:flags=lanczos,fps=15,crop=iw/2:ih:0:0" 1.gif`
可能我们只需要将视频画面的一部分转成gif图片，可以使用crop，具体为crop=宽度:高度:宽度起始:高度起始，比如crop=200:200:0:0，将会从横向0像素，纵向0像素开始，从画面裁剪200x200的区域。
## 提高画面的质量
`fmpeg -ss 9 -t 5 -i 1.mp4 -loop 0 -vf "scale=iw/2:-1:flags=lanczos,fps=15,crop=iw/2:ih:0:0,split[s1][s2];[s1]palettegen[p];[s2][p]paletteuse" 1.gif`
# 水印
## 图片水印
`ffmpeg -i xx.mp4 -vf "movie=xx.png[wm];[in][wm]overlay=30:10[out]" output.mp4`
30:10代表图片的像素位置
## 左下角添加gif动态水印
`ffmpeg -y -i test2.mp4 -ignore_loop 0 -i test.gif  -filter_complex overlay=0:H-h test_out2.mp4`
## 设置显示时间段
`ffmpeg -hide_banner -i big_buck_bunny.mp4 -i doggie2.png -filter_complex "overlay=enable='between(t,5,10)'" out.mp4 -y`
让水印在5-10s时间段显示
## 设置两个水印轮番出现
`ffmpeg -i big_buck_bunny.mp4 -i doggie1.png -i doggie2.png -filter_complex "overlay=enable='lte(mod(t,10),4)',overlay=enable='gt(mod(t,10),6)'" out.mp4 -y`
上面的命令作用是：第一个水印显示4秒后消失，2秒后第二个水印显示4秒后消失。
## 水印位置平移
- `fmpeg -i big_buck_bunny.mp4 -ignore_loop 0 -i doggie3.gif -lavfi "overlay=x=t*20" -shortest out.mp4 -y`
让水印每秒向右移动20像素，直到消失
- `ffmpeg -i big_buck_bunny.mp4 -ignore_loop 0 -i doggie3.gif -lavfi "overlay=enable=\'mod(t,10)\':x=\'100*mod(t,10)-w\'" -shortest out.mp4 -y`
设置水印每隔10秒从左向右移动直至消失
## gif水印循环播放
```bash
第一种：设置gif的-ignore_loop为0，让gif保持循环播放即可，命令如下：
ffmpeg -hide_banner -i big_buck_bunny.mp4 -ignore_loop 0 -i doggie3.gif -filter_complex  overlay -shortest out.mp4 -y
但是这种方式，只适用于gif格式的图像，如果滤镜是一小段视频就无能为力了。

第二种：使用movie滤镜，同样是让gif循环播放，虽然这种方式复杂点，不过这种解决方案支持视频水印，命令如下：
ffmpeg -hide_banner -i big_buck_bunny.mp4 -vf "movie=doggie3.gif:loop=0,setpts=N/FRAME_RATE/TB[out];[0:v][out]overlay=x=main_w-overlay_w:y=0" -shortest out.mp4 -y
上面的命令有两个地方比较关键：

loop=0,setpts=N/FRAME_RATE/TB ：设置水印gif无限循环
-shortest ：将输出文件的时长设置为第一个视频文件的时长，如果不设置，你会发现命令会一直执行根本不会停下来，因为gif图的循环是无限的
这样gif图/短视频就会一直不停的播放了。


希望水印播放一次就不播放了，那就设置上面的eof_action为pass就可以了，如下：
ffmpeg -hide_banner -i big_buck_bunny.mp4 -i doggie3.gif -filter_complex "overlay=x=0:y=0:eof_action=pass" out.mp4 -y

如果视频一开始就播放且只播放一次，假如水印比较短可能根本就没被注意就过去了，这时可以设置水印出现的延迟时间，使用-itsoffset选项，如下：
ffmpeg -hide_banner -i big_buck_bunny.mp4 -itsoffset 3 -i doggie3.gif -filter_complex "overlay=x=0:y=0:eof_action=pass" out.mp4 -y
这样，视频播放3秒后，水印才会出现。
```
## 水印旋转
```bash
如果想实现旋转的功能，需要使用ffmpeg过滤器的链式功能，即：先把作为水印的图片旋转，再覆盖到视频上。

1. 水印旋转一次
ffmpeg -i buck.mp4 -i s1.jpg -lavfi "[1:v]format=rgba,rotate='PI/6:c=0x00000000:ow=hypot(iw,ih):oh=ow'[out];[0:v][out]overlay=10:10" out.mp4 -y

思路是：
调整水印宽高，根据勾股定律计算图片对角长度(hypot)，将这个值设置为水印的宽高，这样，图片无论如何旋转，都不会超过设定的宽高，也就不会出现图片部分丢失的情况了
将图片显示的像素格式转换为rgba格式，如果做过前端的小伙伴会很熟悉的，最后的a表示透明度，如此一来，c=0x00000000的作用就是将图片旋转后的背景变为白色且完全透明，这样就不会遮挡视频了

2. 让旋转停不下，具体命令如下：
ffmpeg -i buck.mp4 -loop 1 -i s1.jpg -lavfi "[1:v]format=rgba,rotate='PI/2*t:c=0x00000000:ow=hypot(iw,ih):oh=ow'[out];[0:v][out]overlay=10:10" -shortest out.mp4 -y
这次水印图片前面添加了-loop 1，正常情况下水印图片默认在播放一次后就停下来，保留最后一帧，所以要让水印图片保持循环才行。
```
# 录屏及摄像头
- `ffmpeg -devices`:列出音频设备及摄像头
## 录屏命令
`ffmpeg -f x11grab -s 1920x1080 -r 30 -i :0.0 -c:v libx264 output.mp4`
- -f x11grab 指定使用X11grab设备作为输入。
- -s [分辨率] 设置录制视频的分辨率，例如 1920x1080。不指定就是全屏
- -r [帧率] 设置录制视频的帧率，例如 30。
- -i :0.0 指定要捕获的X11屏幕。:0.0 通常表示第一个屏幕。如果你有多个屏幕或显示设置不同，这个值可能需要调整。
- -c:v [编码器] 设置视频编码器，例如 libx264 用于H.264编码。
- output.mp4 是输出文件的名称。
## 录屏加声音
`ffmpeg -f x11grab -i :0.0 -f alsa -ac 2 -i hw:2 -vcodec libx264 -acodec libmp3lame -s 1280x720 -r 15 xx.mp4`
alsa使用于linux alsa声卡，`hw:2`是当前电脑上的设备，可以用`arecord -l`命令来显示设备，card几就hw:几
## 摄像头
`ffmpeg -f v4l2 -framerate 25 -video_size 640x480 -i /dev/video0 xx.mkv`
可以通过`v4l2-ctl --list-devices`来查看v4l2的设备，
## 摄像头加声音
`ffmpeg -f v4l2 -framerate 25 -video_size 640x480 -i /dev/video0 -f alsa -ac 2 -i hw:2 -vcodec libx264 -acodec libmp3lame xx.mkv`
# 文字水印
`ffmpeg -i xx.mp4 -vf "drawtext=fontfile=simhei.ttf: text='hi,Nanbert':x=100:y=1fontsize=24fontcolor=white:shadowy=2" output.mp4`
- ontfile:字体类型
- text:要添加的文字内容
- fontsize:字体大小
- fontcolor：字体颜色
# 各种特效
- 渐入:`ffmpeg -i in.mp4 -vf fade=in:0:90 out.mp4`
- 黑白:`ffmpeg -i in.mp4 -vf lutyuv="u=128:v=128" out.mp4`
- 锐化:`ffmpeg -i in.mp4 -vf unsharp=luma_msize_x=7:luma_msize_y=7:luma_amount=2.5 out.mp4`
- 反锐化:`ffmpeg -i in.mp4 -vf unsharp=7:7:-2:7:7:-2 out.mp4`
- 渐晕:`ffmpeg -i in.mp4 -vf vignette=PI/4 out.mp4`
- 闪烁渐晕:`ffmpeg -i in.mp4 -vf vignette='PI/4+random(1)*PI/50':eval=frame out.mp4`
- 视频颤抖:`ffmpeg -i in.mp4 -vf crop="in_w/2:in_h/2:(in_w-out_w)/2+((in_w-out_w)/2)*sin(n/10):(in_h-out_h)/2+((in_h-out_h)/2)*sin(n/7)" out.mp4`
- 色彩变幻:`ffmpeg -i in.mp4 -vf hue="H=2*PI*t:s=sin(2*PI*t)+1" out.mp4`
- 模糊处理:`ffmpeg -i in.mp4 -vf boxblur=5:1:cr=0:ar=0 out.mp4`
- 镜像翻转:`ffmpeg -i in.mp4 -vf crop=iw/2:ih:0:0,split[left][tmp];[tmp]hflip[right];[left]pad=iw*2[a];[a][right]overlay=w out.mp4`
- 水平翻转:`ffmpeg -i in.mp4 -vf geq=p(W-X\\,Y) out.mp4`
- 垂直翻转:`ffmpeg -i in.mp4 -vf vflip out.mp4`
- 浮雕效果:`ffmpeg -i in.mp4 -vf format=gray,geq=lum_expr='(p(X,Y)+(256-p(X-4,Y-4)))/2' out.mp4`
- 均匀噪声:`ffmpeg -i in.mp4 -vf noise=alls=20:allf=t+u out.mp4`
# 字幕
- `ffmpeg -i xx.mp4 -vf ass=xx.ass -b:v 3000k xx.mp4`
# 图片
- 5秒处截图：`ffmpeg -ss 00:00:05 -i xx.mp4 xx.jpg -r 1 -vframes 1 -an -vodec mjpeg`
- 每一帧输出图片：`ffmpeg -i xx.mp4 image_%d.jpg`
- 图片转为视频：`ffmpeg -f image2 -i image_%d.jpg video.mpg`


