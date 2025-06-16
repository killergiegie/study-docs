# <center>LLVM IR 转机器码教程</center>

## 1. LLVM IR 转换为 SelectionDAG

LLVM IR（中间表示）在经过优化后，会被转换为 **SelectionDAG（选择有向无环图）**，用于指令选择和优化。

- **SelectionDAG 的创建**：
  - 由 `SelectionDAGBuilder` 类负责创建。
  - `SelectionDAGISel` 类调用 `SelectionDAGBuilder::visit()` 遍历每条 IR 指令，创建 `SDNode`（SelectionDAG 节点）。
  - 例如，处理 `SDiv`（整数除法）指令时，会使用 `ISD::SDIV` 操作码创建 `SDNode` 节点。

---

## 2. 合法化 SelectionDAG

SelectionDAG 中的某些指令可能 **不受目标架构支持**，因此需要合法化（Legalization），使其适配目标平台。

- **合法化的过程**：
  - **数据类型合法化**：目标平台可能不支持某些数据类型，需要转换。
  - **操作合法化**：目标平台可能不支持某些操作，需要用等效的操作代替。
  - `TargetLowering` 接口负责处理合法化，每个架构实现自己的 `TargetLowering`（例如 `X86TargetLowering`）。
  - `setOperationAction()` 函数用于指定某个 `ISD` 节点是否需要合法化处理。

---

## 3. 转换为目标平台的机器指令

合法化后的 `SDNode` 需要转换为目标平台的 **MachineSDNode**（机器指令）。

- **机器指令定义**：
  - 使用 `.td`（TableGen）文件描述指令集、寄存器等信息。
  - 通过 `tablegen` 工具转换为 `.inc` 头文件，供 C++ 代码使用。

- **指令选择**：
  - 可由 `SelectCode` 自动完成，也可以自定义 `SelectionDAGISel::Select()` 进行手动选择。
  - 生成的 `MachineSDNode` 仍然是 DAG 形式，尚未变成线性指令流。

---

## 4. 指令调度

机器执行指令的顺序需要优化，以提高性能。

- **指令调度的目标**：
  - 解决 **指令依赖**、**寄存器压力** 和 **流水线阻塞** 等问题，减少执行延迟。
  - 通过 **拓扑排序** 将 DAG 转换为 **线性指令序列**。
  - 目标平台提供特定的 **调度算法** 来优化代码执行顺序。

---

## 5. 寄存器分配

机器指令使用的是 **虚拟寄存器（Virtual Register）**，需要映射到真实的硬件寄存器。

- **寄存器分配的关键问题**：
  - **寄存器有限**，无法无限使用虚拟寄存器。
  - **寄存器溢出（Register Spilling）**：当寄存器不够用时，需要把数据存入内存，增加开销。

- **寄存器分配的方法**：
  - **图染色算法**（复杂度较高，不常用）。
  - **贪心算法**（LLVM 默认）：优先分配 **活动周期长** 的变量，减少寄存器溢出。

- **SSA 形式转换**：
  - 在寄存器分配之前，LLVM 代码仍然是 **SSA（静态单赋值）形式**，但真实机器不支持 SSA，需要进行转换。

---

## 6. 代码发射

最终的机器指令需要被 **转换为目标可执行文件** 或 **JIT 直接执行**。

- **代码发射方式**：
  1. **JIT（即时编译）**：将机器码直接发射到内存并执行。
  2. **MC（Machine Code）框架**：生成汇编代码或目标文件。

- **关键实现**：
  - `LLVMTargetMachine::addPassesToEmitFile()` 负责 **发射目标文件**。
  - `AsmPrinter::EmitInstruction()` 负责 **转换 MI（Machine Instruction）到 MCInst（Machine Code Instruction）**。
  - **llc 工具** 可用于生成目标平台的汇编代码：
    ```sh
    llc input.ll -o output.s
    ```

---

## 总结

LLVM 将 C/C++ 代码转换为机器码的流程如下：
1. **LLVM IR 转换为 SelectionDAG**。
2. **对 SelectionDAG 进行合法化**，确保兼容目标架构。
3. **转换为目标平台的 MachineSDNode**，映射到具体指令集。
4. **指令调度**，优化指令顺序以提高性能。
5. **寄存器分配**，将虚拟寄存器映射到物理寄存器。
6. **代码发射**，生成最终的汇编代码或可执行文件。

这个过程涉及 **IR 低级化（Lowering）**、**合法化（Legalization）**、**指令选择（Instruction Selection）**、**调度（Scheduling）**、**寄存器分配（Register Allocation）**，最终 **发射代码（Code Emission）**，形成可执行文件。