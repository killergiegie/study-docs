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

## 2. 编译安装 miniconda

首先下载 miniconda 安装脚本并运行
```bash
cd ~/local/src
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
bash Miniconda3-latest-Linux-x86_64.sh
```

接下来按照终端提示依次操作：
1. 确认 license
2. 输入安装路径：```~/local/miniconda3```
3. 配置是否 conda init: no

最后更新环境变量
```bash
vi ~/.bashrc
    # 最后一行加入
    export PATH=~/local/miniconda3/bin:$PATH
source ~/.bashrc
# 检查conda是否可用
conda --version
```

## 3. 编译安装 LLVM 21

首先在https://github.com/llvm/llvm-project/releases 选择LLVM版本
或者可以直接使用最新版llvm
```bash
git clone https://github.com/llvm/llvm-project
```

然后安装所需的依赖库：```cmake```, ```ninja```, ```ncurses```
若是手动编译，还需更新环境变量：
```bash
# CMake
export PATH=/usr/local/cmake-3.28.0-rc4-linux-x86_64/bin:$PATH
# Ninja
export PATH=$HOME/local/ninja/bin:$PATH
# LLVM Dependency
export LD_LIBRARY_PATH=$HOME/local/ncurses/lib:$LD_LIBRARY_PATH
```


接下来构建并编译LLVM

```bash
mkdir build
cd build

cmake -G Ninja ../llvm-project-llvmorg-21.1.0-rc1/llvm \
 -DCMAKE_INSTALL_PREFIX=$HOME/local/llvm-21.1.0 \
 -DLLVM_ENABLE_PROJECTS="clang" \
 -DLLVM_TARGETS_TO_BUILD="X86" \
 -DCMAKE_BUILD_TYPE=Release \
 -DLLVM_ENABLE_ASSERTIONS=On \
 -DCMAKE_C_COMPILER=/usr/local/gcc-13.2.0/bin/gcc \
 -DCMAKE_CXX_COMPILER=/usr/local/gcc-13.2.0/bin/g++


ninja -j32
ninja install
```

编译并安装完成后，编写环境变量配置
首先编辑 ```~/.bashrc``` 文件

```bash
vi ~/.bashrc
# 加入如下内容：
    # Choose Compiler
    source $HOME/local/envs/compiler_llvm21.sh
    # source $HOME/local/envs/compiler_gcc13.sh
    # source $HOME/local/envs/compiler_gcc7.sh

    echo "Using C compiler: $CC"
    echo "Using C++ compiler: $CXX"
```

接下来编写编译器配置 ```$HOME/local/envs/compiler_llvm21.sh```

```bash
# Set LLVM environment variables
export LLVM_DIR=$HOME/local/llvm-21.1.0        # Set the base path for LLVM installation
export PATH=$LLVM_DIR/bin:$PATH        # Add LLVM bin directory to PATH (for clang, llvm-tblgen, etc.)
export LD_LIBRARY_PATH=/usr/local/gcc-13.2.0/lib64:$LLVM_DIR/lib:$LD_LIBRARY_PATH
export CMAKE_PREFIX_PATH=$LLVM_DIR:$CMAKE_PREFIX_PATH  # For CMake to find LLVM
export LLVM_CONFIG=$LLVM_DIR/bin/llvm-config  # Set llvm-config for scripts/tools that use it

# LIBRARY_PATH 影响编译时库的搜索顺序
export LIBRARY_PATH=/usr/local/gcc-13.2.0/lib64:$LIBRARY_PATH
# C_INCLUDE_PATH/CPLUS_INCLUDE_PATH 影响头文件搜索顺序
export CPLUS_INCLUDE_PATH=/usr/local/gcc-13.2.0/include/c++/13.2.0:$CPLUS_INCLUDE_PATH

# 在 Bash 或 zsh 中导出变量
export CCC_OVERRIDE_OPTIONS="^--gcc-install-dir=/usr/local/gcc-13.2.0/lib/gcc/x86_64-pc-linux-gnu/13.2.0"

# Set Clang as the default compiler (optional but recommended)
export CC=/usr/local/gcc-13.2.0/bin/gcc
export CXX=/usr/local/gcc-13.2.0/bin/g++

```

随后检查clang是否安装成功
```bash
clang -v
clang++ -v
```








