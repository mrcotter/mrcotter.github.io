---
title: Homebrew 编译 mpv --HEAD 报错的解决方法（macOS）
categories: 程序猿
tags:
  - mpv
  - macOS
  - FFmpeg
  - Homebrew
abbrlink: 2912b351
date: 2017-12-05 15:53:41
---

对于 macOS High Sierra 的用户，在 mpv 正式发布下一个正式版本之前，为了解决播放视频文件的一个`致命 bug`（即在播放视频时，如果尝试双击播放另一个视频文件，mpv 无法正常切换，而是启动一个新的实例，但该实例图标无响应也无法强制关闭），我之前一直建议用户在编译 mpv 时加入 `--HEAD` 来使用 bug 修复后的版本。不过，在近一段时间，使用 Homebrew 编译 `--HEAD` 版本必然出错。原因是 mpv 的开发者对最近一次 `FFmpeg API` 的修改导致程序功能不可用的情况感到忍无可忍，因此开发者决定使用自己 fork 并有针对性打补丁的版本 `ffmpeg-mpv`，并且编译 git master 版本的 mpv 会强制要求使用这一特殊版本 FFmpeg 的支持，也造成了现有的 brew formula 失效。

<!--more-->

------

### 怎么办

关于 mpv 和 FFmpeg 之间的矛盾如何解决目前仍然没有定论，存在着不少争议。为了能够使用 Homebrew 正确编译最新版本的 mpv，我对比并且试验了一些方法，最后使用的是来自网友 [dreness](https://github.com/dreness) 提供的办法（见 [issue#5108](https://github.com/mpv-player/mpv/issues/5108)），通过打补丁的方式修改 FFmpeg 和 mpv 的编译代码：

1. 将 FFmpeg HEAD Git Repo Url 替换为 ffmpeg-mpv 所在地址；
2. macOS 下编译 mpv 需对 `stream/stream_libarchive.h` 打补丁。

### 代码变更内容

具体到代码，需要修改的文件包括 Homebrew 下的 `/Formula/ffmpeg.rb`、`/Formula/mpv.rb` 以及 mpv 下的 `/stream/stream_libarchive.h`，变动如下：

```bash
diff --git a/Formula/ffmpeg.rb b/Formula/ffmpeg.rb
index 5ee917f0..13fad092 100644
--- a/Formula/ffmpeg.rb
+++ b/Formula/ffmpeg.rb
@@ -3,7 +3,7 @@ class Ffmpeg < Formula
   homepage "https://ffmpeg.org/"
   url "https://ffmpeg.org/releases/ffmpeg-3.4.tar.bz2"
   sha256 "5d8911fe6017d00c98a359d7c8e7818e48f2c0cc2c9086a986ea8cb4d478c85e"
-  head "https://github.com/FFmpeg/FFmpeg.git"
+  head "https://github.com/mpv-player/ffmpeg-mpv.git"

   bottle do
     sha256 "1807e00fdfc308feaa78924e0b6991b26e6f37ea379ac82b78dca038a0f67a7b" => :high_sierra
diff --git a/Formula/mpv.rb b/Formula/mpv.rb
index c557288a..af34525b 100644
--- a/Formula/mpv.rb
+++ b/Formula/mpv.rb
@@ -12,6 +12,14 @@ class Mpv < Formula
     sha256 "ffbd72e37a328a5c6d5779746c9e6462b3f6d260f365bd922716a16ad3f3cd8a" => :el_capitan
   end

+  head do
+    patch do
+      # patch for https://github.com/mpv-player/mpv/issues/5108
+      url "https://youbeill.in/scrap/note-VSiLBZQxn2.txt"
+      sha256 "972cd7301eb7783abb632a904168af5cf891dca5b42016cf96241b86a5231d92"
+    end
+  end
+
   option "with-bundle", "Enable compilation of the .app bundle."

   depends_on "pkg-config" => :build
```

```bash
diff --git a/stream/stream_libarchive.h b/stream/stream_libarchive.h
index 56e86a6..8884834 100644
--- a/stream/stream_libarchive.h
+++ b/stream/stream_libarchive.h
@@ -1,6 +1,11 @@
 #include <locale.h>
 #include "osdep/io.h"

+#ifdef __APPLE__
+# include <string.h>
+# include <xlocale.h>
+#endif
+
 struct mp_log;

 struct mp_archive {
```

### 解决方法

现在，我们可以通过简单的命令来对 Homebrew Formula 进行修改。

```bash
cd $(brew --prefix)/Homebrew/Library/taps/homebrew/homebrew-core
curl -o mpv-ffmpeg.patch https://youbeill.in/scrap/note-PnAcVrJqbc.txt
patch -p1 < mpv-ffmpeg.patch
```

之后就可以正常编译 `ffmpeg --HEAD` 和 `mpv --HEAD`，例如：

```bash
brew install ffmpeg --HEAD --with-fdk-aac --with-sdl2 --with-freetype --with-libass --with-libbluray --with-libvorbis --with-libvpx --with-opus --with-webp --with-x265
brew install mpv --HEAD --with-bundle --with-libbluray --with-libdvdnav --with-libdvdread --with-uchardet --with-libaacs --with-libcaca --with-rubberband --with-libarchive --with-vapoursynth
```

声明：以上只是一个暂时的解决方案，直到开发者最终决定如何解决 ffmpeg-mpv 这一 fork 的问题。后续如果有其它变化，我会及时更新补充。补丁是**基于现有 FFmpeg 发行版本 3.4 进行修改的，如果后续有版本更新，该补丁会失效**，需要做相应的修改。另外，补丁文件是以在线文件的形式提供，**需要联网**。

如果有其它问题，欢迎在评论区给我留言。Peace！
