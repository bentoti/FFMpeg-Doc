# cuda和ffmpeg安装

* 安装显卡驱动

    [centos7安装nvidia驱动](./Centos7Nvidia.md)

*  下载FFmpeg

    [FFmpeg官网下载页面](http://ffmpeg.org/download.html)

* 下载安装CUDA Toolkit

    [nvidia官网下载](https://developer.nvidia.com/cuda-downloads)

    根据下载页面的提示下载安装

    [nvidia官网安装文档](https://docs.nvidia.com/cuda/cuda-installation-guide-linux/index.html)

* 编译FFmpeg

    ```
    [root@localhost ~]# PATH="$HOME/bin:$PATH" PKG_CONFIG_PATH="$HOME/ffmpeg_build/lib/pkgconfig" ./configure \
    --enable-cuda \
    --enable-cuvid \
    --enable-nvenc \
    --enable-nonfree \
    --enable-libnpp \
    --extra-cflags=-I/usr/local/cuda/include \
    --extra-ldflags=-L/usr/local/cuda/lib64 \
    --prefix="$HOME/ffmpeg_build" \
    --bindir="$HOME/bin"

    [root@localhost ~]# make -j 10

    [root@localhost ~]# make install

    [root@localhost ~]# ffmpeg -version
    ```

    PATH为添加环境变量就是ffmpeg命令的存放路径

    PKG_CONFIG_PATH为pkgconfig路径，如果遇到[nv-codec-headers](#nv-codec-headers)报错此路径和make install时PREFIX指定路径保持一致

    --prefix安装路径

    --bindir命令安装路径，和PATH添加的路径保持一致


* 问题记录

    <div id="nv-codec-headers"></div>

    ```
    ERROR: cuda requested, but not all dependencies are satisfied: ffnvcodec

    [root@localhost ~]# git clone https://git.videolan.org/git/ffmpeg/nv-codec-headers.git

    [root@localhost ~]# cd nv-codec-headers

    [root@localhost ~]# make && make install prefix="/path/to"

    ```

    pkgconfig命名导致的问题，编译FFmpeg时PKG_CONFIG_PATH为PREFIX指定的路径

    ```
    ERROR: libnpp not found
    ```

    --extra-cflags和--extra-ldflags未指定或者指定路径未不存在

    ```
    ffmpeg: error while loading shared libraries: libnppig.so.9.2: cannot open shared object file: No such file or directory

    [root@localhost ~]# export LD_LIBRARY_PATH=/usr/local/cuda/lib64
    ```
