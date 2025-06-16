# Linux: 编译安装各类开发工具到本地


## 1. 编译安装 GCC 11.3
下载源码并编译
```bash
cd ~/local/src
wget https://ftp.gnu.org/gnu/gcc/gcc-11.3.0/gcc-11.3.0.tar.gz
tar -xzf gcc-11.3.0.tar.gz
cd gcc-11.3.0
./contrib/download_prerequisites

cd ~/local/src
mkdir gcc-11.3.0-build
cd gcc-11.3.0-build

../gcc-11.3.0/configure --prefix=$HOME/local/gcc-11.3.0 --disable-multilib --enable-languages=c,c++

make -j16
make install
```
更新环境配置
```bash
export CC=$HOME/local/gcc-11.3.0/bin/gcc
export CXX=$HOME/local/gcc-11.3.0/bin/g++
export PATH=$HOME/local/gcc-11.3.0/bin:$PATH
export LD_LIBRARY_PATH=$HOME/local/gcc-11.3.0/lib64:$LD_LIBRARY_PATH
```