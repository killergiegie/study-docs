# <center>安装LLVM + Clang</center>
## 1. 下载LLVM源码
在https://github.com/llvm/llvm-project/releases 选择LLVM版本
或者可以直接使用最新版llvm
```bash
git clone https://github.com/llvm/llvm-project
```
可以选择使用Ninja工具构建LLVM，似乎速度会快一些（可能）
Ninja安装指令：
```bash
sudo apt install ninja-build
```

## 2. 编译LLVM
配置CMake，同时支持本地x86端和RISC-V端
```bash
# 创建build目录
mkdir build && cd build

# 执行配置选项
cmake -G Ninja ../llvm  -DCMAKE_INSTALL_PREFIX=../install -DLLVM_ENABLE_PROJECTS="clang" -DLLVM_TARGETS_TO_BUILD="X86;RISCV" -DCMAKE_BUILD_TYPE=Release -DLLVM_ENABLE_ASSERTIONS=On
# cmake -G Ninja ../llvm \
#   -DCMAKE_INSTALL_PREFIX=../install \
#   -DLLVM_ENABLE_PROJECTS="clang" \
#   -DLLVM_TARGETS_TO_BUILD="X86;RISCV" \  
#   -DCMAKE_BUILD_TYPE=Release \
#   -DLLVM_ENABLE_ASSERTIONS=On

# 执行编译和安装
ninja -j8
ninja install
```
编译完成后，build/bin目录下的工具（如clang、llc）将支持生成X86和RISC-V代码

## 3. 安装RISC-V交叉编译工具链

## 4. 将llvm工具加入环境变量
```bash
nano ~/.bashrc
```
在文件末尾添加：（记得修改路径）
```bash
export PATH="/home/killer/LLVM/llvm-project-main/install/bin:$PATH"
export LD_LIBRARY_PATH="/home/killer/LLVM/llvm-project-main/install/lib:$LD_LIBRARY_PATH"
```
保存后加载配置（或重新打开终端）：
```bash
source ~/.bashrc
```