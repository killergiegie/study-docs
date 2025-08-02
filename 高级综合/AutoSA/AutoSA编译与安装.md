# AutoSA 编译与安装

配置 libtool 工具环境变量
```bash
export ACLOCAL_PATH=$HOME/local/libtool/share/aclocal:$ACLOCAL_PATH
```

## 0. 编译安装 GF2X

```bash
export ACLOCAL_PATH=$HOME/local/libtool/share/aclocal:$ACLOCAL_PATH

cd ~/local/src
wget https://gitlab.inria.fr/gf2x/gf2x/-/archive/gf2x-1.3.0/gf2x-gf2x-1.3.0.tar.gz
tar -xvzf gf2x-gf2x-1.3.0.tar.gz
cd gf2x-gf2x-1.3.0
# 1. 生成 configure 脚本
autoreconf -i

./configure --prefix=$HOME/local/gf2x
make -j8
make install
```

## 1. 编译安装 NTL

```bash

./configure NTL_GMP_LIP=on GMP_PREFIX=$HOME/local/gmp PREFIX=$HOME/local/ntl GF2X_PREFIX=$HOME/local/gf2x  NTL_GF2X_LIB=on

make
make check
make install

```


## 2. 编译安装 AutoSA

**注意！！：要使用 LLVM-10.0.0 左右的版本，llvm-12及以上不兼容**

首先要修改源码：
```bash
cd src
vi autosa_common.cpp
    # 大约在第1231行
    if (index < 0)  // ❌
    # 改为
    if (!index)  // ✅
```

记得更新 ~/.bashrc 使用 llvm-10

```bash
export ACLOCAL_PATH=$HOME/local/libtool/share/aclocal:$ACLOCAL_PATH
export CPPFLAGS="-I$HOME/local/ntl/include -I$HOME/local/gmp/include"
export LDFLAGS="-L$HOME/local/ntl/lib -L$HOME/local/gmp/lib"
export LIBS="-lpthread"
export CXXFLAGS="-O2 -g0"
export CFLAGS="-O2 -g0"

git clone https://github.com/UCLA-VAST/AutoSA.git
cd AutoSA
conda activate autoSA

. ./install

```
