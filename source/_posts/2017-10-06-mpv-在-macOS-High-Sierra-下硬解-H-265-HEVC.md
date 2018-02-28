---
title: mpv 在 macOS High Sierra 下硬解 H.265/HEVC
categories: 程序猿
tags:
  - mpv
  - macOS
  - HEVC
  - H.265
  - Hardware Decoding
  - 硬解
abbrlink: 497173ab
date: 2017-10-06 14:20:59
---

macOS High Sierra 新增的一个特性，即支持新一代视频压缩技术 **HEVC（High Efficiency Video Coding 高效视频编码，也称为 H.265）**，这项技术也內建在 iOS 11 中。对比现行常用的 H.264/x264 标准，HEVC 编码的视频能在保证相同画质的前提下，将文件的体积最少降低 40%，优势可以说相当明显。而想要播放 HEVC 视频，mac 上常见的播放器都可以做到，但要实现硬件加速却需要满足一些条件和设置。这篇文章会介绍使用 mpv 这一优秀的开源播放器在 macOS High Sierra 下实现硬解 H.265/HEVC，具体的操作会涉及到使用 Terminal 以及基于 Homebrew 来安装和管理部分程序。

<!--more-->

------

### 支持硬解 HEVC 的 Mac 仅限于新机型

1. "We're building in software encoder support into High Sierra for all Macs."
2. "Hardware acceleration of HEVC in the newest Macs."

苹果在 macOS High Sierra 发布会上的表述，即所有的 Mac 都支持 HEVC 的软件编码，而硬件解码的支持仅限于最新的 Mac 几款机型。具体来说，**需要 2016 或 2017 款 Mac**。2016 款 Mac，CPU 型号为 Skylake，可以支持硬解 HEVC 8bit；2017 款，CPU 使用的是 Kaby Lake，可以硬解 8bit 和 10bit（即 HDR ）。2016 款 Mac 如果配有独显，虽然独显能硬解 10bit，但基于苹果系统不行，需要换用 Windows。

个人还在使用 2013 年末的 15寸 Macbook Pro Retina，当然不在支持之列，过段时间考虑换新。

### 编译 x265 和 FFmpeg 

默认使用 [Homebrew](https://brew.sh/) 管理和安装 macOS 中的命令行程序，例如本篇的 FFmpeg 和 mpv 等，而安装 Homebrew 的前提是安装 Xcode 及其 Command Line Tools，这里就不详细说明了，请自行 Google。

最新版本的 x265 在默认安装时已提供对 16bit 的支持，无需添加 option `--with-16-bit` ，使用 `install` 或者 `reinstall`（如果你的系统已安装 x265）重新编译支持 16bit 的 x265，否则解码时会有类似 `libx265` 缺少 `pix_fmt 'yuv420p10le'` 这样的错误提示：

```bash
brew install x265
```

之后，你需要重新编译 FFmpeg。FFmpeg 于近期引入了一条针对 HEVC 的补丁，在编译时加入 `--HEAD` 即可使用该补丁版本。同上，使用 `install` 或者 `reinstall`（如果你的系统已安装 FFmpeg）重新进行编译，其它 option，除 x265 外可根据自己的需求增减：

```bash
brew install ffmpeg --HEAD --with-fdk-aac --with-sdl2 --with-freetype --with-libass --with-libbluray --with-libvorbis --with-libvpx --with-opus --with-webp --with-x265
```

### 编译 mpv

mpv 解码时依赖于 FFmpeg，而 macOS High Sierra 更新后由于应用打包机制的变化导致 mpv 在切换视频时产生了一个严重的 bug。在播放视频时，如果尝试双击播放另一个视频文件，mpv 无法正常切换，而是启动一个新的实例，但该实例图标无响应也无法强制关闭。该 bug 已经修复，暂时未发布到 release 通道，因此也需要在编译时加入 `--HEAD` 使用 bug 修复后的版本。关于 mpv 的设置，可以参考我的 config 配置文件，请访问 [mrcotter_dotfiles](https://github.com/mrcotter/mrcotter_dotfiles)。

```bash
brew install mpv --HEAD --with-bundle --with-libbluray --with-libdvdnav --with-libdvdread --with-uchardet --with-libaacs --with-libcaca --with-rubberband --with-libarchive --with-vapoursynth
brew linkapps mpv
```

需要注意的是，该 `master` 版本的 mpv 改变了部分设置参数，需要对 `~/.config/mpv/mpv.conf` 进行修改，如下：

```bash
vo=opengl
```

修改为

```bash
vo=gpu
```

如果原设置使用的是：

```bash
profile=opengl-hq
```

需修改为

```bash
profile=gpu-hq
```

另外，确保 `hwdec` 设置为 `auto / auto-copy / videotoolbox / videotoolbox-copy` 其中一项。至此，如果你正好拥有一台 2017 款的 Mac，那么你已经可以使用 mpv 硬解 H.265/HEVC 10bit 视频了，去看看 CPU 占用率吧。最后，附上一小段脚本，帮你快速绑定 mpv 作为系统默认的视频播放器（需要 Homebrew 的支持）。

```bash
#!/usr/bin/env bash

# Set default application using mpv for video play
APPFILE=/Applications/mpv.app

if [ ! -e "$APPFILE" ]; then
   	exit
fi

BUNDLEID=$(mdls -name kMDItemCFBundleIdentifier -r $APPFILE)

EXTS=( 3GP ASF AVI FLV M4V MKV MOV MP4 MPEG MPG MPG2 MPG4 OGM RMVB WMV )

if test ! $(which duti); then
	echo "Installing duti"
	brew install duti
fi

for ext in ${EXTS[@]}
do
	lower=$(echo $ext | awk '{print tolower($0)}')
	duti -s $BUNDLEID $ext all
	duti -s $BUNDLEID $lower all
done
```

保存为 `.sh` 文件后不要忘了使用 `chmod` 设置执行权限。Peace！






