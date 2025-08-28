# 编译优化算法 - LLVM Pass 源码分析


## 0. LLVM Pass 源码目录结构

此目录是 LLVM 的核心优化 pass 源码，建议按照下方顺序学习
 
```bash
llvm/llvm/lib/Transforms/
    InstCombine/            # 1. 指令级优化（基础）：InstCombine
    Scalar/                 # 2. 标量优化（核心）：DCE、GVN、LICM、LoopSimplify
    Utils/                  # 3. 通用数据流分析框架（工具库）
    IPO/                    # 4. 跨过程优化，对整个程序函数之间做优化。
    Vectorize/              # 5. 向量化 Pass
    AggressiveInstCombine/  # 6. 比普通 InstCombine 更“激进”的指令合并/化简。
   
    ObjCARC/                # 无需学习 只在 Apple 平台上有用。
    HipStdPar/              # 无需学习 这个目录和 GPU offloading 有关。
    Coroutines/             # 无需学习 把高级语言里的 协程 (coroutine) 特性转化成 LLVM IR 可以执行的状态机。
    CFGuard/                # 无需学习 微软的安全机制，Windows 下的防护特性。
```

## 1. InstCombine Pass 合集

InstCombine 是 LLVM 最常见的**指令级别模式匹配优化** (Instruction Combine) 的实现集合。这些文件基本上每个都对应一类 IR 指令（算术 / 位运算 / PHI / Select …）的优化规则。


- `InstructionCombining.cpp`        :  核心入口文件，InstCombine Pass 的主调度逻辑。
- `InstCombineAddSub.cpp`           :  针对 加法/减法 的优化规则。
- `InstCombineAndOrXor.cpp`         :  针对 按位与 (and)、或 (or)、异或 (xor) 的优化。
- `InstCombineAtomicRMW.cpp`        :  针对 原子读-改-写 (atomicrmw) 指令的优化。
- `InstCombineCalls.cpp`            :  针对 函数调用 (call) 的优化。
- `InstCombineCasts.cpp`            :  针对 类型转换 (cast) 指令的优化。
- `InstCombineCompares.cpp`         :  针对 比较指令 (icmp, fcmp) 的优化。
- `InstCombineLoadStoreAlloca.cpp`  :  针对 内存相关指令 (load, store, alloca) 的优化。
- `InstCombineMulDivRem.cpp`        :  针对 乘法 (mul)、除法 (div)、取余 (rem) 的优化。
- `InstCombineNegator.cpp`          :  针对 取负数 (negation) 的优化。
- `InstCombinePHI.cpp`              :  针对 PHI 节点（SSA 里的 φ 函数）的优化。
- `InstCombineSelect.cpp`           :  针对 select 指令（类似三元运算符 cond ? a : b）的优化。
- `InstCombineShifts.cpp`           :  针对 移位 (shl, lshr, ashr) 指令的优化。
- `InstCombineSimplifyDemanded.cpp` :  按需简化 如果某些比特/向量元素 永远不会被用到，就可以简化掉计算。
- `InstCombineVectorOps.cpp`        :  针对 向量运算的优化。



## 2. Scalar Pass 合集

Scalar 是 LLVM 里“标量优化”的核心实现。这个目录很大，因为几乎所有编译原理书上讲过的经典优化都在这里。

### 2.1 基础死代码/常量传播类

这些是“减负型”Pass，直接减少 IR 体积和计算。

- `DCE.cpp`             : 删除无用指令（无副作用且结果没人用）。
- `ADCE.cpp`            : 更激进，基于控制流条件，能删掉更多。
- `BDCE.cpp`            : 按位跟踪无用位，删除更细粒度的死计算。
- `SCCP.cpp`            : 稀疏条件常量传播 + 死代码删除
- `EarlyCSE.cpp`        : 早期公共子表达式消除（比 GVN 简单快速）


### 2.2 公共子表达式/全局优化

提升效率、减少重复求值。

- `GVN.cpp`                             :  全局值编号（公共子表达式消除）
- `NewGVN.cpp`                          :  新版 GVN 实现。
- `GVNHoist.cpp`                        :  把公共子表达式“上提”到 dominator block，减少重复计算
- `GVNSink.cpp`                         :  把公共子表达式“下沉”到 merge block，避免过早计算
- `CorrelatedValuePropagation.cpp`      :  利用分支相关性传播值。
- `ConstraintElimination.cpp`           :  基于条件约束（x>0 ⇒ x≠0）消除冗余比较。


### 2.3 控制流简化 / CFG 优化

控制流更规整，利于后续分析；消分支 → 提高预测准确率。

- `SimplifyCFGPass.cpp`                         : 控制流图简化，合并块、删除无用跳转。
- `FlattenCFGPass.cpp`                          : 把深层次 CFG 扁平化（降低分支深度）。
- `StructurizeCFG.cpp`                          : 把 CFG 转成更规整的结构（GPU backend 常用）。
- `JumpThreading.cpp / DFAJumpThreading.cpp`    : 跳转穿线，把确定分支提前。
- `CallSiteSplitting.cpp`                       : 根据参数常量把调用点拆开。
- `MergeICmps.cpp`                              : 把多个比较合并为一个。
- `Reassociate.cpp`                             : 重新结合算术表达式 (a+b)+c → a+(b+c)，利于发现优化机会。
- `NaryReassociate.cpp`                         : 更一般化的表达式重结合。



### 2.4 循环优化（大头）

循环优化是性能关键：减少迭代次数、改善 cache、消除冗余。


- `LICM.cpp`                  ：Loop Invariant Code Motion，把循环不变计算搬到外面。
- `LoopSimplifyCFG.cpp`       ：循环 CFG 规整化，方便其他优化。
- `LoopUnrollPass.cpp`        ：循环展开（unroll）。
- `LoopUnrollAndJamPass.cpp`  ：展开 + 融合嵌套循环。
- `LoopRotation.cpp`          ：循环旋转（改变循环入口位置，便于优化）。
- `LoopStrengthReduce.cpp`    ：循环强度削弱（用加法替代乘法）。
- `IndVarSimplify.cpp`        ：归纳变量简化（让 induction variable 更规整）。
- `LoopIdiomRecognize.cpp`    ：识别循环模式，替换为库函数（如 memset, memcpy）。
- `LoopPredication.cpp`       ：把循环条件转换为更好预测的形式。
- `LoopDeletion.cpp`          ：删除无用循环。
- `LoopFuse.cpp`              ：循环融合（减少内存访问）。
- `LoopInterchange.cpp`       ：交换嵌套循环的层次（改善局部性）。
- `LoopFlatten.cpp`           ：展开嵌套循环成单层。
- `LoopDistribute.cpp`        ：循环分裂（提高并行度）。
- `LoopSink.cpp`              ：把某些指令 sink 到循环内部末尾，减少寄存器压力。
- `LoopInstSimplify.cpp`      ：循环内部简化。
- `LoopLoadElimination.cpp`   ：删除循环内冗余 load。
- `LoopTermFold.cpp`          ：折叠循环终止条件。
- `LoopAccessAnalysisPrinter.cpp`：循环访问模式分析。
- `LoopBoundSplit.cpp`        ：根据边界条件拆循环。
- `LoopVersioningLICM.cpp`    ：给 LICM 增加版本化以处理别名不确定性。
- `IVUsersPrinter.cpp`        ：打印/分析 induction variable 使用情况。


### 2.5 内存与数据优化

减少内存流量，把复杂结构打散成 SSA，更好优化。

- `SROA.cpp`                  ：Scalar Replacement of Aggregates，把结构体/数组切分为标量。
- `Reg2Mem.cpp`               ：相反的，把 SSA 形式拆回内存（调试/降低优化时用）。
- `MemCpyOptimizer.cpp`       ：优化 memcpy → 直接赋值 / 消冗余。
- `DeadStoreElimination.cpp`  ：删除不会被用到的 store。
- `MergedLoadStoreMotion.cpp` ：合并相邻的 load/store。
- `SeparateConstOffsetFromGEP.cpp`    ：优化 GEP（地址计算），分离常量偏移。
- `Scalarizer.cpp / ScalarizeMaskedMemIntrin.cpp` ：把向量内存操作分解为标量操作。


### 2.6 其他特殊/平台相关优化

- `TailRecursionElimination.cpp`  ：尾递归消除 → 转换为循环。
- `SpeculativeExecution.cpp`      ：推测执行安全的指令，提前计算。
- `Float2Int.cpp`                 ：尝试用整数计算替代浮点。
- `DivRemPairs.cpp`               ：合并除法和余数运算。
- `InductiveRangeCheckElimination.cpp`    ：消除循环范围检查。
- `LowerXXXIntrinsic.cpp 系列`    ：把高级 intrinsic 降级成普通 IR。
- `GuardWidening.cpp / MakeGuardsExplicit.cpp`    ：和异常/溢出检查有关。
- `PartiallyInlineLibCalls.cpp`   ：把库调用部分内联。
- `ConstantHoisting.cpp`          ：把常量上提到循环外或更高层，避免重复生成。
- `WarnMissedTransforms.cpp`      ：发出警告，提示哪些优化没做成。


### 2.7 辅助/调试类

- `Scalar.cpp`            ：注册所有标量 Pass 的入口文件。
- `AnnotationRemarks.cpp` ：输出 remark 注释，方便分析优化效果。
- `PlaceSafepoints.cpp / RewriteStatepointsForGC.cpp` ：和 GC 相关（插入 safepoint）。


## 3. Utils 合集

Utils 是 LLVM 优化的“工具箱”。这里的文件大部分不是直接的优化 Pass，而是优化 Pass 反复调用的通用基础设施。

### 3.1 SSA / 寄存器 / 内存相关工具

这些是 SSA 相关的基石，几乎所有优化 Pass 都依赖它们。

- `Mem2Reg.cpp`                         ：Promote Memory to Register，把 alloca 的内存变量提升成 SSA 寄存器（构造 SSA 的关键 Pass）。
- `DemoteRegToStack.cpp`                ：反向操作，把寄存器值放回内存（调试/降低优化时有用）。
- `SSAUpdater.cpp / SSAUpdaterBulk.cpp` ：辅助类，快速更新 SSA 形式，合并多个定义点。
- `LCSSA.cpp`                           ：构造 Loop-Closed SSA（循环出口的 SSA 形式）。
- `PromoteMemoryToRegister.cpp`         ：Mem2Reg 的实现细节。
- `SimplifyIndVar.cpp`                  ：简化循环中的 induction variable（归纳变量）。


### 3.2 控制流图 (CFG) 操作

这些是 CFG 改造工具，Scalar/Loop/IPO Pass 会大量用到。

- `BreakCriticalEdges.cpp`      ：拆分关键边（CFG 上必须断开才能插入指令的地方）。
- `FlattenCFG.cpp`              ：扁平化 CFG（跟 Scalar/FlattenCFGPass 有关系）。
- `SimplifyCFG.cpp`             ：控制流简化（合并块、删除冗余分支）。
- `ControlFlowUtils.cpp`        ：各种 CFG 修改辅助函数。
- `FixIrreducible.cpp`          ：把不可规约 CFG（irreducible CFG）修复为规约的。
- `UnifyFunctionExitNodes.cpp`  ：把函数多个出口统一成一个。
- `UnifyLoopExits.cpp`          ：把循环多个出口统一。
- `LoopSimplify.cpp`            ：规整化循环结构（为 LoopPass 准备）。


### 3.3 循环优化工具

所有循环优化 Pass（LICM、LoopUnroll、LoopVectorize）都会依赖这些。

- `LoopUtils.cpp`                   ：循环相关的公共函数（检查 loop 是否 canonical 等）。
- `LoopRotationUtils.cpp`           ：循环旋转的辅助实现。
- `LoopUnroll.cpp / LoopUnrollRuntime.cpp / LoopUnrollAndJam.cpp / LoopVersioning.cpp / LoopPeel.cpp` ：各种循环展开/拆分/版本化工具。
- `CanonicalizeFreezeInLoops.cpp`   ：在循环里规范化 freeze（避免未定义行为传播）。
- `LoopConstrainer.cpp`             ：对循环条件的工具类。


### 3.4 函数 / 模块操作


