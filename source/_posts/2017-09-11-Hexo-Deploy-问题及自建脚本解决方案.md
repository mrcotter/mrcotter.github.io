---
title: Hexo Deploy 问题及自建脚本解决方案
categories: 程序猿
tags:
  - Hexo
  - 博客
  - Blog
  - Bug
abbrlink: e7815373
date: 2017-09-11 17:04:57
---

上周在更新 Blog 的时候遇到了奇怪的问题，Hexo 突然就不能正常 deploy 了。在 GitHub 上搜索到别人也有遇到同样的情况，但是解决方法各不相同。有删除 `.deploy_git` 再重新 `deploy`，有重新生成 `ssh 密钥`，也有备份后删除整个 hexo 重来的。这几个办法我都用了，结果还是不管用，执行 `hexo d` 命令后，就一直卡在某个位置，等待一段时间后如果按 `ctrl + c` 强行结束进程，会提示 `Deploy done`，但远程的 repository 并没有任何变化。

手动使用 `git` 命令进行同步一切正常，对于这个莫名其妙的bug，手动建一个脚本就能解决了。

<!--more-->

------

### Clone 博客所在的 repository

在本地创建一个目录，将下列命令中的 `yourname` 替换为你的 Github `ID`，本地路径地址也请替换。

```bash
git clone https://github.com/yourname/yourname.github.io.git /Path/to/Local/Git/yourname.github.io
```

### 创建 deploy.sh 脚本

新建文档，输入：

```bash
#!/bin/bash

# Generate blog
hexo clean
hexo generate
# Copy to repository
cp -R public/* /Path/to/Local/Git/yourname.github.io
cd /Path/to/Local/Git/yourname.github.io
# Deploy
git add .
current_date_time=`date "+%Y-%m-%d %H:%M:%S"`
git commit -m "Site updated: $current_date_time"
git push origin master
```

将文件另存为 `.sh` 格式至 Hexo 博客所在目录，启动 `Terminal`，跳转至该目录下，设置脚本的执行权限。

```bash
chmod +x deploy.sh
```

之后，需要更新网站时，在博客所在目录执行以下命令即可。

```bash
./deploy.sh
```

这个 bug 就暂时这么解决吧，有其它问题欢迎留言。
