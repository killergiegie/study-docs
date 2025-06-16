# <center>LLVM 创建自己的pass</center>

## 1. 创建pass测试文件
默认当前目录为~/LLVM/，创建测试代码testcode.c
```bash
vi testcode.c
    int func(int a, int b) 
    {
        int sum = 0;
        int iter;
        for (iter = 0; iter < a; iter++) {
            int iter1;
            for (iter1 = 0; iter1 < b; iter1++) {
                sum += iter > iter1 ? 1 : 0;
            }
        }
        return sum;
    }
```
使用clang工具将C语言转换成字节码，注意要带上 **-O1** 或者更高的优化等级，否则会默认不开启优化，无法使用pass
```bash
clang -emit-llvm -c -O1 testcode.c -o testcode.bc
```
---

## 2. 创建新pass：opcodeCounter
进入llvm源码相关目录下，创建新的目录opcodeCounter
```bash
cd ~/LLVM/llvm-project-main/llvm/lib/Transforms/
mkdir opcodeCounter
cd opcodeCounter
```
创建pass源码 OpcodeCounter.cpp
```bash
# cd ~/LLVM/llvm-project-main/llvm/lib/Transforms/opcodeCounter/
vi OpcodeCounter.cpp
```
```cpp
#include "llvm/Pass.h"
#include "llvm/IR/Function.h"
#include "llvm/IR/Instructions.h"
#include "llvm/IR/InstIterator.h"
#include "llvm/Passes/PassBuilder.h"
#include "llvm/Passes/PassPlugin.h"
#include "llvm/Support/raw_ostream.h"
#include <map>     // 包含 std::map
#include <string>  // 包含 std::string

using namespace llvm;

namespace {
    /// 定义Pass类
    class OpcodeCounter : public PassInfoMixin<OpcodeCounter> {
    public:
    /// 主函数，用于处理模块
    PreservedAnalyses run(Function &F, FunctionAnalysisManager &) {
            errs() << "OpcodeCounter Pass is running on function: " << F.getName() << "\n";
            // 创建一个映射，用于统计每种操作码的数量
            std::map<std::string, int> opcodeMap;

            // 遍历函数中的每一条指令
            for (auto &I : instructions(F)) {
            // 获取指令的操作码名称
            std::string opcodeName = I.getOpcodeName();
            // 在映射中增加对应操作码的计数
            opcodeMap[opcodeName]++;
            }

            // 打印函数名称
            errs() << "Function: " << F.getName() << "\n";

            // 打印每种操作码的数量
            for (auto &entry : opcodeMap) {
            errs() << "  " << entry.first << ": " << entry.second << "\n";
            }

            // 返回表明分析结果未修改IR
            return PreservedAnalyses::all();
        }
    };
} // namespace

/// 注册Pass插件
extern "C" LLVM_ATTRIBUTE_WEAK ::llvm::PassPluginLibraryInfo
llvmGetPassPluginInfo() {
  return {
    LLVM_PLUGIN_API_VERSION, "OpcodeCounter", "v1.0",
    [](PassBuilder &PB) {
      PB.registerPipelineParsingCallback(
        [](StringRef Name, FunctionPassManager &FPM,
           ArrayRef<PassBuilder::PipelineElement>) {
          // 如果Pass名称匹配，将Pass添加到Pass管理器中
          if (Name == "opcodeCounter") {
            FPM.addPass(OpcodeCounter());
            return true;
          }
          return false;
        });
    }};
}
```
---
## 3. 修改CMake，重新编译
在当前目录下，创建CMakeLists.txt
```bash
vi CMakeLists.txt
```
写入以下内容：
```CMakeLists
add_llvm_library(OpcodeCounter MODULE
    OpcodeCounter.cpp
)
```
修改上一级目录的CMakeLists.txt，在文件末尾加入一行：add_subdirectory(opcodeCounter)
```bash
cd ..
vi CMakeLists.txt
    add_subdirectory(opcodeCounter)
```
回到llvm的编译目录build下，重新执行cmake构建，并编译，无需install
```bash
cd ~/LLVM/llvm-project-main/build/

cmake -G Ninja ../llvm  -DCMAKE_INSTALL_PREFIX=../install -DLLVM_ENABLE_PROJECTS="clang" -DLLVM_TARGETS_TO_BUILD="X86;RISCV" -DCMAKE_BUILD_TYPE=Release -DLLVM_ENABLE_ASSERTIONS=On

ninja -j8
```
编译完成后即可在 ~/LLVM/llvm-project-main/build/lib 目录下，看到生成的动态库文件 OpcodeCounter.so
若此处编译报错，重新修改pass源码，无需重新cmake，可以直接编译，直到编译通过。

---

## 4. 执行pass优化
在生成了新的pass的库文件后，使用opt工具对测试代码进行优化
```bash
opt -debug-pass-manager -load-pass-plugin ../llvm-project-main/build/lib/OpcodeCounter.so -passes="opcodeCounter" -disable-output testcode.bc
```
此时可以正常输出pass分析后的内容：
```bash
Running analysis: InnerAnalysisManagerProxy<AnalysisManager<Function>, Module> on [module]
Running pass: {anonymous}::OpcodeCounter on func (21 instructions)
OpcodeCounter Pass is running on function: func
Function: func
  add: 3
  br: 5
  icmp: 5
  phi: 6
  ret: 1
  zext: 1
Running pass: VerifierPass on [module]
Running analysis: VerifierAnalysis on [module]
```
注意此处使用的是新版的pass管理工具，旧版的在指令上略有不同