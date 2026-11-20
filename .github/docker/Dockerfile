# OpenWrt 25.12 编译环境（固件构建用）
# 目的：把 apt 装依赖那 ~9 分钟预烤进镜像，构建时直接 `container:` 用。
FROM ubuntu:24.04

ENV DEBIAN_FRONTEND=noninteractive LANG=C.UTF-8 \
    FORCE_UNSAFE_CONFIGURE=1

RUN apt-get update && apt-get install -y --no-install-recommends \
       build-essential clang flex bison g++ gawk gcc-multilib g++-multilib \
       gettext git libncurses-dev libssl-dev libelf-dev zlib1g-dev \
       python3 python3-dev python3-setuptools python3-pyelftools python3-ply \
       rsync unzip file wget curl ca-certificates subversion swig cmake \
       ninja-build autoconf automake libtool pkgconf ccache help2man patch \
       time xsltproc zstd liblz4-tool libzstd-dev device-tree-compiler lua5.1 \
       musl-tools sudo xz-utils bzip2 \
       rclone \
    && rm -rf /var/lib/apt/lists/* \
    && git config --system --add safe.directory '*' \
    && echo "--- 自检 ---" \
    && for t in gcc g++ make python3 unzip rsync dtc ccache musl-gcc rclone; do \
         command -v "$t" >/dev/null || { echo "缺 $t"; exit 1; }; done \
    && gcc --version | head -1 && ccache --version | head -1 && rclone version | head -1
