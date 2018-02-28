---
title: macOS 下 FFmpeg 视频转码入门及进阶使用教程
categories: 程序猿
tags:
  - FFmpeg
  - 视频转码
  - HEVC
  - x264
  - x265
  - Homebrew
abbrlink: a7465832
date: 2018-02-24 18:26:35
---

如今较为常见的视频封装格式有 mp4 和 mkv 等， 内部的视频编码格式从前几年盛行的 `H.264/x264` 逐渐开始向新一代的 `HEVC/x265`（High Efficiency Video Coding 高效视频编码）过渡，而常见的音频编码格式无非 AC3、DTS 或者 AAC 等。无论是借助带有 GUI 的编码软件，还是使用命令行，[FFmpeg](http://ffmpeg.org/) 是最为广泛使用的工具，理论上 FFmpeg 支持各个平台，包括 Windows、macOS、iOS 以及 Android 等，这里只介绍在 macOS 下的使用。通过简单的命令，你可以大致了解 FFmpeg 在视频转换上的强大之处，视频编码部分也集中在 x264、x265，以及如何压制 macOS High Sierra 和 iOS 11 可以正确识别并生成缩略图的 HEVC 10bit 视频。文章最后，也会用一个较为复杂的例子，应用 `-filter_complex` 进行视频帧率的插值运算、嵌入 pgs 图形字幕，以及最后输出 HEVC 编码进行说明。

<!--more-->

------

### 安装

如果有看过我以前文章的朋友，可能会注意到使用 [Homebrew](https://brew.sh/) 编译 [mpv](https://mpv.io/) 的一个重要依赖就是 FFmpeg。不过，如果将其用作视频转码，默认编译的 FFmpeg 会缺少一部分组件，因此这里可能需要重新安装 FFmpeg。以我个人编译版本为例，使用 `--HEAD` 来配合最新的 mpv，在 Terminal 中输入如下命令：

```bash
brew install ffmpeg --HEAD --with-fdk-aac --with-sdl2 --with-freetype --with-libass --with-libbluray --with-libvorbis --with-libvpx --with-opus --with-webp --with-x265
```

等待安装结束即可。

### 基础篇

#### 压制 x264 编码视频文件

```bash
ffmpeg -i input.mp4 -c:a libfdk_aac -c:v libx264 -crf 20 -preset slow output.mp4
```

使用 FFmpeg 编码的基本规则， `-i` 之后的文件为输入的视频文件，即 input.mp4，支持的格式众多，例如 mkv、flv、vob 等等，文件可以包含目录，使用 macOS 的文件拖拽功能很方便。output.mp4 即为输出文件，文件名可自定义，视频封装格式建议对应编码格式，不应将 mpeg-2 或者 vp8 编码的视频也封装为 mp4。`-c:a` 之后表示输出文件的音频编码器，一般 mp4 常用的音频编码为 AAC-LC，按照官方 Wiki 指南，建议使用编码器 `libfdk_aac` 而不是 `aac`，`libfdk_aac` 音质更好，这也是为什么在前文中编译 FFmpeg 增加 `--with-fdk-aac` 的原因。`-c:v` 之后代表输出文件的视频编码器，使用 `libx264` 即可压制 x264 编码的视频流。`-crf 20` 代表视频编码的码率系数，数字越大，压制的效果越差，建议选择范围在 `16 - 28`，压制高质量的视频建议取值 20 以下。`-preset slow` 代表一组控制压缩时间和文件大小的参数选择，一般常选 `fast`、`medium` 和 `slow`。

以上都是基于 one-pass 压制，如果需要严格控制码率则需要使用 two-pass，更详细的介绍，可以参考 [Encode / H.264](https://trac.ffmpeg.org/wiki/Encode/H.264)。

#### 压制 HEVC 10bit 编码视频文件

其实 FFmpeg 很早就开始支持 HEVC (x265) 的视频转码，只是一直改动较大，而最近的版本也终于支持编码 macOS High Sierra 下 Quicktime 可以播放，并且在系统中能够正确预览并生成缩略图的视频文件。编码命令的改动很小，添加一个 `format tag` 参数即可，如下：

```bash
ffmpeg -i input.mp4 \
       -c:v libx265 -preset medium -crf 18 -pix_fmt yuv420p10le \
       -c:a libfdk_aac -b:a 256k \
       -tag:v hvc1 \
       output_10bit.mp4
```

和压制 x264 视频非常类似，主要的不同点在于 `-c:v` 视频编码器需换为 `libx265`，并且压制 10bit 需要指定色彩空间，添加 `-pix_fmt yuv420p10le`。在音频编码参数中，如果增加 `-b:a`，可以控制音频文件的码率，按需使用。最后，非常重要的一点，必须添加参数 `tag:v hvc1`，这样输出的 Video Stream 会被标记为 `hvc1`，可以被 macOS 以及 iOS 11 原生支持播放，否则默认会被标记为 `hev1`，不被原生支持，第三方播放器播放倒没什么问题。

### 进阶篇

#### 修改视频分辨率

假如原视频的分辨率为 1920x1080，为了降低文件大小，最简单的办法是将其转压成一个分辨率较低的版本，例如 720p，即 1280x720，那么我们可以使用 `scale` 视频滤镜来缩放视频：

```bash
ffmpeg -i input.mp4 -vf scale=-2:720 -c:v libx264 -crf 20 -preset slow -c:a copy output.mp4
```

`-vf scale=-2:720` 会自动计算对应的横向分辨率（需为 2 的倍数，因此为 -2），源文件音频编码保持不变，因此设为 `copy` 即可。特殊情况下，遇到源文件视频比例错误，除了修改分辨率数值，还需要设置 `dar` 参数，例如：

```bash
ffmpeg -i input.avi -vf scale=722x406,setdar=16/9 -c:v libx264 -c:a libfdk_aac -preset slow -crf 20 output.mp4
```

另外，绝对不建议增大分辨率，因为毫无意义，受限于原视频的视频质量，增大分辨率除了体积增大，画质只会更差。

#### 反交错（Deinterlace）

偶尔我会遇到一些早期使用 VCD/DVD 时代编码的视频，其中一个重要的特点就是隔行扫描，而直接转码的结果就是视频中快速运动的物体都能看到非常明显的扫描线。解决办法同样需要应用 `vf` 视频滤镜中的 `yadif` 来进行反交错，如下：

```bash
ffmpeg -i input.vob -vf yadif -c:v libx264 -preset slow -crf 20 -c:a libfdk_aac -b:a 256k output.mp4
```

如果压制出来的效果不佳（还是有扫描线），可以尝试将 `vf` 的部分改为 `-vf yadif=1:-1:0,mcdeint=2:1:10`。

#### 旋转视频

需要将原视频进行旋转，同样可以应用视频滤镜来达到目的，如下：

```bash
ffmpeg -i input.mov -vf "transpose=1" -c:a copy output.mov
```

其中，

```
0 = 90 Counter Clockwise and Vertical Flip  (default) 
1 = 90 Clockwise 
2 = 90 Counter Clockwise 
3 = 90 Clockwise and Vertical Flip
```

如果想要 180 度翻转视频，则需要改为 `-vf "transpose=2,transpose=2"`。值得注意的是，旋转视频意味着对视频进行重编码，输出质量会稍微受到影响，可以添加 `crf` 参数控制视频输出质量，音频部分可以使用 `copy`。

### 一个复杂的“栗子”

最后的这个例子，是我最近遇到的一个视频，简要的编码信息如下：

```
Input #0, matroska,webm, from 'Input.mkv':
    Duration: 00:23:55.97, start: 0.000000, bitrate: 16372 kb/s
    Stream #0:0: Video: hevc (Main 10), yuv420p10le(tv, bt709), 1920x1080, SAR 1:1 DAR 16:9, 59.94 fps, 59.94 tbr, 1k tbn, 59.94 tbc (default)
    Stream #0:1(jpn): Audio: flac, 48000 Hz, stereo, s32 (24 bit) (default)
    Stream #0:2(jpn): Audio: flac, 48000 Hz, stereo, s32 (24 bit)
    Stream #0:3(chi): Subtitle: hdmv_pgs_subtitle (default)
    Stream #0:4(chi): Subtitle: hdmv_pgs_subtitle
```

可以看到，这是一个 HEVC 10bit 编码，分辨率 1080p，帧率 59.94 fps 的视频文件，带有两条 flac 编码的音轨，另有两条是 pgs 格式的图形字幕。我自己的 Macbook Pro 已经无法完全流畅地播放这个视频了，除了 HEVC 带来的巨大计算量，高帧率也是一个麻烦，可惜网上没有其它好的片源，因此，我只有自己尝试压缩。目标：维持分辨率但帧率减半，即降为 29. 97 fps，音轨只需要第一条，并且重编码为 AAC-LC，原片为日语，因此必须带有字幕，图形字幕直接嵌入视频，最后以 HEVC 10bit 重编码，少许降低码率。

改变帧率普遍会使用 `-vf fps=fps=29.97` 这类的参数，但自己尝试后发现一个问题，视频观看的感觉有跳跃性，不流畅，很像是丢帧的感觉。因为将帧率减半，意味着有一半的信息都丢弃了，而普通降低帧率的算法只有简单的插值运算甚至完全没有，造成了视频不连贯的效果。因此，改变帧率正确的做法是进行运动插值运算（Motion Interpolation），此法既可用在提高帧率上，也可以用于降低帧率，最终的结果都是提高视频播放的流畅度。这里会使用 `-filter_complex` 代替 `vf`，联合应用 `minterpolate`、`overlay` 以及 `map` 来解决帧率、嵌入视频，和保留一条音轨的问题。压制命令如下：

```bash
ffmpeg -i input.mkv \
-filter_complex "[0:v]minterpolate='fps=29.97:mi_mode=mci:me_mode=bidir:mc_mode=aobmc:vsbmc=1'[bg],[bg][0:s:0]overlay[v]" -map "[v]" -map 0:a:0 \
-c:v libx265 -preset medium -crf 18 -pix_fmt yuv420p10le \
-c:a libfdk_aac -b:a 256k \
-tag:v hvc1 \
output_10bit.mp4
```

这里看起来会很复杂，实际上 `-filter_complex` 的工作模式就像是 pipe，`[0:v]` 表示输入文件的视频流，对应 Stream #0:0。从 `minterpolate` 到 `vsbmc=1` 都是插补滤镜的设置参数，具体的作用可以查看[官方文档](http://ffmpeg.org/ffmpeg-filters.html#toc-minterpolate)。`[bg]`代表该滤镜输出后的视频流，并传递给下一个滤镜 `overlay`。`[0:s:0]`表示输入文件的第一个字幕通道，对应 Stream #0:3，所以如果是 `[0:s:1]` 则对应 Stream #0:4。 `overlay` 就会将该图形字幕嵌入到视频中，然后输出为 `[v]`，进行 mapping。视频取处理后的 `[v]`，音频取原输入文件的第一个音频通道，`[0:a:0]` 即代表 Stream #0:1。最后和此前压制视频的参数就一模一样了，压制为 HEVC 10bit 编码的视频文件。

需要注意的是，运动插值运算非常耗时，CPU 占用确不高，应该是 `minterpolate` 滤镜只能调用单核的缘故。在我的电脑上， 此 23 分钟左右的视频压制一次耗时约 20 小时，请谨慎使用。

好了，关于 FFmpeg 视频转码和压缩的话题就聊到这里，下次见。Peace！
