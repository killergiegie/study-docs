# <center>LLVM工具使用方法</center>

## 1. clang基本命令

使用Clang作为编译器驱动，编译得到可执行文件a.out
```bash
clang testp.c
```
使用 -E 参数，可以实现预处理
```bash
clang testp.c -E
```
使用 -ast-dump 参数输出抽象语法树（AST）
```bash
clang -cc1 test.c -ast-dump
```
使用clang将C语言代码转换为LLVM IR表达
>-S 参数告诉 Clang 生成汇编语言文件
-emit-llvm 参数该选项告诉 Clang 编译器生成 LLVM IR 文件
-o 指定生成的文件名
```bash
clang -emit-llvm -S multiply.c -o multiply.ll
```
可以将ll文件编译为指定架构的汇编文件（-S）、目标文件（-c）以及可执行文件
```bash
clang -target x86_64 -S multiply.ll -o multiply_x86.s
clang -target x86_64 -c multiply.ll -o multiply_x86.o
clang -target x86_64 multiply.ll -o multiply_x86
```

---

## 2. llvm命令
可以使用llvm-as工具将ll文件转换成字节码，使用llvm-dis工具可以逆向转换
```bash
llvm-as multiply.ll –o multiply.bc
llvm-dis multiply.bc –o multiply.ll
```
使用llvm-link命令链接两个LLVM bitcode文件,和传统的链接器不同，llvm-link不会链接Object文件生成一个二进制文件，它只链接bitcode文件。
```bash
llvm-link test1.bc test2.bc –o output.bc
```
可以使用lli工具运行bitcode文件
```bash
lli output.bc
```
---

## 3. llc工具
1. llc 是一个常用的工具用于将 LLVM IR (.ll 文件) 编译成汇编代码或目标文件，但它和 clang 的作用有所不同。具体来说，clang 是一个前端编译器，提供了从源代码到目标文件的完整编译流程，而 llc 是 LLVM 的低级编译器，用于将 LLVM 中间表示（IR）转换为机器码的汇编或目标文件。
2. llc 不能直接生成可执行文件。llc 只负责将 LLVM IR 文件转换为汇编代码或目标文件（.o）。它的主要功能是将 LLVM 的中间表示（IR）转换成特定架构的汇编语言或目标文件，而不进行链接操作。如果你需要生成可执行文件，还需要使用链接器来链接目标文件。通常，链接的工作是通过 clang 或 gcc 等编译器来完成的。

```bash
llc -target x86_64 -filetype=s assembly multiply.ll -o multiply_x86.s
llc -target x86_64 -filetype=obj multiply.ll -o multiply_x86.o
```
>clang 会做更多的工作，不仅可以处理 C/C++ 源代码，还可以处理 IR 文件，最终生成可执行文件、目标文件或汇编文件。
llc 专注于将 LLVM IR 文件转换成底层的汇编代码或目标文件。
再使用ld/lld指令链接目标文件为可执行文件
```bash
ld -o multiply_x86 multiply_x86.o
```
---

## 4. opt基本命令
使用opt将ll文件按照指定的pass优化
```bash
opt -S -passes=instcombine test_file.ll -o output1.ll
```

若有多个pass需要使用，则使用引号和逗号隔开
```bash
opt -S -passes="pass1,pass2,pass3" test_file.ll -o output1.ll
```
使用命令查看所有的pass库：
```bash
opt --print-passes
```

```IR
define dso_local i32 @factor(i32 noundef %n) #0 {
entry:
  %n.addr = alloca i32, align 4
  %ret = alloca i32, align 4
  store i32 %n, ptr %n.addr, align 4
  store i32 1, ptr %ret, align 4
  br label %while.cond

while.cond:                                       ; preds = %while.body, %entry
  %0 = load i32, ptr %n.addr, align 4
  %cmp = icmp sgt i32 %0, 1
  br i1 %cmp, label %while.body, label %while.end

while.body:                                       ; preds = %while.cond
  %1 = load i32, ptr %n.addr, align 4
  %2 = load i32, ptr %ret, align 4
  %mul = mul nsw i32 %2, %1
  store i32 %mul, ptr %ret, align 4
  %3 = load i32, ptr %n.addr, align 4
  %dec = add nsw i32 %3, -1
  store i32 %dec, ptr %n.addr, align 4
  br label %while.cond, !llvm.loop !6

while.end:                                        ; preds = %while.cond
  %4 = load i32, ptr %ret, align 4
  ret i32 %4
}


```