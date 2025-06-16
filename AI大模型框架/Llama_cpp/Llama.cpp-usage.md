# Llama.cpp 命令行参数列表（仅 llama-mtmd-cli）

## 1. 基础参数
```-h, --help, --usage```
- 作用：打印帮助信息，显示所有可用参数。
- 使用场景：查询参数文档。

```--version```
- 作用：打印当前 llama.cpp 的版本和构建信息。
- 使用场景：调试或报告时确认所用版本。

```--completion-bash```
- 作用：输出一段可用于 bash 的自动补全脚本。
- 使用场景：让你在命令行中更方便地 tab 补全参数（可通过 ```source <(llama-mtmd-cli --completion-bash)``` 启用）。

```--verbose-prompt```
- 默认：false
- 作用：生成之前打印 prompt 的详细信息（包括 tokens）。
- 使用场景：调试 prompt 分词或 token embedding 时非常有用。

## 2. 多线程参数

```-t, --threads N```
- 默认：-1（自动，通常为 CPU 核心数）
- 作用：设置用于生成阶段（推理）的线程数。
- 环境变量：LLAMA_ARG_THREADS
- 建议：设置为你机器的物理核心数，例如 24 线程机器可设 ```-t 24```。

```-tb, --threads-batch N```
- 默认：同 --threads
- 作用：控制 prompt embedding 和 batch preprocessing 阶段的线程数。
- 区分：有些模型加载阶段耗时，独立控制有助于优化 pipeline。

## 3. CPU Affinity / 核心绑定参数
这些参数用于绑定进程/线程到特定的 CPU 核心，以获得更稳定的性能（主要在多核服务器/嵌入式系统中使用）。

```-C, --cpu-mask M```
- 作用：指定线程运行在哪些 CPU 上，例如 0xf 表示绑定到 CPU 0~3。
- 高级用法：通过位掩码来绑定线程。

```-Cr, --cpu-range lo-hi```
- 作用：用数字范围来指定 CPU 核心，比如 ```--cpu-range 0-11```。
- 便于理解：比掩码更直观。

```--cpu-strict <0|1>```
- 默认：0
- 作用：是否强制使用指定的 CPU，设为 1 表示“严格绑定”。

```--prio N```
- 作用：设置主推理线程的优先级。
- -1: low（低）
- 0: normal（默认）
- 1: medium
- 2: high
- 3: realtime（实时）

```--poll <0~100>```
- 默认：50
- 作用：控制空闲线程的 polling 行为（轮询 vs 睡眠）。
- 0 表示完全不轮询（最省电，但响应慢）
- 100 表示全轮询（最及时，但耗 CPU）
- 通常保持默认即可。

```-Cb, --cpu-mask-batch M / -Crb, --cpu-range-batch lo-hi```
- 作用：用于绑定 batch 处理线程的 CPU 核心（如加载 prompt）。

```--cpu-strict-batch、--prio-batch、--poll-batch```
- 含义：分别对应 batch 阶段的：
  - 严格绑定 CPU；
  - 设置线程优先级；
  - 控制轮询等级。
- 通常用途：调优性能或在多模型部署时避免资源冲突。


## 4. 上下文与生成设置

```-c, --ctx-size N```
- 默认：4096（或模型加载的默认值）
- 作用：设置上下文长度（token 数量）。
- 重要性：决定 prompt + response 的最大 token 数；越大越耗内存。

```-n, --predict, --n-predict N```
- 默认：-1（无限）
- 作用：指定要生成的 token 数量。
- 推荐：设为 1024、2048 等限制响应长度；否则可能输出过长。

```-p, --prompt PROMPT```
- 作用：直接从命令行传入 prompt 文本进行生成。
- 适用：快速测试；不建议用于长 prompt。

```-f, --file FNAME```
- 作用：从文本文件中读取 prompt。
- 推荐：处理较长 prompt、更方便复用。

```-bf, --binary-file FNAME```
- 作用：从二进制文件中读取 token 序列作为 prompt。
- 用途：更底层控制，通常用于程序自动生成 prompt。

```-e, --escape 和 --no-escape```
- 作用：
  - 默认会将 \n, \t, \\, \', \" 等转义字符解析为控制字符；
  - 加上 --no-escape 会完全禁用这一特性。
- 使用建议：
  - 如果 prompt 来源是手写文本，保留默认；
  - 如果 prompt 是机器拼接的字符串，可能需要加 --no-escape。

```-b, --batch-size N```
- 默认：2048
- 作用：逻辑 batch 大小（对 prompt 的 token 分段输入）。
- 含义：大 batch 会加速推理，但可能导致内存溢出。

```-ub, --ubatch-size N```
- 默认：512
- 作用：物理 batch 大小（通常不超过 --batch-size）。
- 含义：微调 prompt 处理的并发度，帮助内存控制。

```--keep N```
- 默认：0
- 作用：对续写任务，保留 prompt 的前 N 个 token，不清空。
- 使用场景：在多轮对话或流式输入中，可以保留先前上下文。

```--swa-full```
- 含义：使用完整大小的 SWA（Sliding Window Attention）缓存。
- 默认：关闭
- 作用：更大的 KV 缓存区（可能支持更长上下文或更快检索）。
- 平台：CPU 专用（目前 Flash Attention 不支持此项）。
- 建议：大模型长上下文下开启，代价是内存占用增加。

```-fa, --flash-attn```
- 作用：开启 Flash Attention 加速。
- 平台：GPU 专属（目前基于 CUDA/Metal 的后端才支持）。
- 效果：大幅提升多头注意力计算速度，尤其是大上下文情况下。
- 注意：必须编译时开启 -DLLAMA_CUBLAS=ON 或 Metal 支持。

## 5. RoPE 与上下文扩展（Context Scaling）

RoPE（旋转位置编码）是 Llama 模型中的位置编码方式：

```--rope-scaling {none,linear,yarn}```
- 含义：控制位置编码在长上下文时的扩展策略：
  - none：不扩展；
  - linear：线性拉伸频率；
  - yarn：使用 YaRN 高阶策略（效果更好但更复杂）。
- 建议：默认已较合理，若你使用了 > 4k context 的模型（如支持 32k token），可配合 --rope-scale 使用。

```--rope-scale N```
- 作用：RoPE 上下文长度扩展因子，N 越大，支持上下文越长。
- 如：N=4 表示支持原模型 4 倍的 context 长度。

```--rope-freq-base N / --rope-freq-scale N```
- 高阶参数，一般只在手动调整上下文频率/周期时使用。
- 通常建议保留默认或由模型 config 加载。

用于更高质量的上下文 extrapolation（外推）：
```--yarn-orig-ctx N```：原始模型的训练 context size。
```--yarn-ext-factor N```：插值/外推的混合因子（-1 为自动）。
```--yarn-attn-factor N```：调节注意力缩放。
```--yarn-beta-slow N / --yarn-beta-fast N```：用于高阶修正项控制。

- 这些参数只对使用 YaRN 扩展的模型有效（如 yaRN-8K, yaRN-32K），否则不会生效。

## 6. KV Cache 缓存相关参数

```-nkvo, --no-kv-offload```
- 含义：禁用 KV 缓存的 offload（不从内存或 GPU 显存交换）。
- 平台：CPU & GPU 通用
- 建议：禁用可减少延迟但占用更多内存；默认开启 KV offload 更适合超长 context 或低内存设备。

```-ctk, --cache-type-k TYPE / -ctv, --cache-type-v TYPE```
- 作用：设置 KV 缓存中的 K/V 的数据类型。
- 支持的类型：
  - 浮点：f32, f16, bf16
  - 量化：q8_0, q4_0, q4_1, iq4_nl, q5_0, q5_1
- 推荐配置（内存 vs 性能权衡）：
  - CPU 上常用：q4_0, q5_1
  - 高精度：f16, bf16
  - 默认：f16

```-dt, --defrag-thold N```
- 含义：设置 KV 缓存碎片整理的阈值。
  - 小于该比例将触发 KV cache 的重新整理（会有延迟）。
  - 设置为负值（如 -1）可禁用 defrag。
- 建议：默认即可，必要时手动控制长时间运行时的内存碎片。

## 7. 并行解码与 NUMA 设置

```-np, --parallel N```
- 作用：设置一次并行生成多少条序列（multi-stream 解码）。
- 默认：1
- 推荐：
  - 推理多个请求/用户时设置 > 1；
  - 单请求交互不建议开启，会浪费上下文。

```--mlock```
- 作用：强制把整个模型锁进物理内存，防止被 swap（置换）。
- 平台：仅限 CPU（或 GPU 管理时）
- 建议：
  - 长时间推理任务或性能敏感时使用；
  - 需要操作系统允许 mlock 权限。

```--no-mmap```
- 作用：禁用内存映射（mmap）方式加载模型。
- 默认：使用 mmap 加快启动、节省虚拟内存页。
- 建议：
  - 某些平台不支持 mmap 或页面管理不佳时可以关闭。

``--numa TYPE``
- 作用：优化 NUMA（非一致内存访问）架构的性能。
- 选项：
  - distribute：线程均匀分布在多个 NUMA 节点；
  - isolate：将所有线程绑定到一个节点；
  - numactl：遵循 numactl 提供的 CPU 拓扑。
- 平台：多 CPU Socket 的服务器上使用
- 推荐：若未设置，最好在首次运行前执行 ```sudo bash -c 'echo 3 > /proc/sys/vm/drop_caches'``` 清缓存。

## 8. 设备配置（Device Management）

```--device <dev1,dev2,..> / -dev```
- 含义：显式指定用于 offload 的设备列表，例如：cuda:0,cuda:1。
- 平台：**GPU 专用**
- 用途：控制模型分配到哪个 GPU 上，适用于多 GPU 推理或定向加速。
  - none 表示完全不用设备，走纯 CPU 路径。

```--list-devices```
- 作用：列出所有可用的加速设备及编号。
- 用途：确认设备 ID 和名称，例如 CUDA GPU 还是 Metal 设备。

```--override-tensor, -ot <tensor name pattern>=<buffer type>```
- 作用：覆盖模型中某些张量的存储类型，例如将 q_proj 改为 f32。
- 用途：用于微调某些算子精度（例如提升注意力精度）。
- 适合：进阶用户做混合精度实验（如部分层保精度，其余低精度）。

```-ngl, --gpu-layers N```
- 作用：指定将模型前多少层加载到 GPU（用于 offload 加速）。
- 平台：**GPU 专用**
- 典型用法：
  - --gpu-layers 0：纯 CPU 推理；
  - --gpu-layers 30：Llama2-7B 全部 30 层丢到 GPU；
- 建议：根据 GPU 显存来决定层数，低端卡建议保守设定。

```-sm, --split-mode {none,layer,row}```
- 作用：指定多 GPU 分配方式：
  - none：只用一个 GPU；
  - layer（默认）：按层划分，每块卡负责一部分；
  - row：按矩阵行划分，适用于推理优化（如 Flash-Attn）。
- 用途：大模型部署在多 GPU 上。
- 平台：**GPU 专用**

```-ts, --tensor-split N0,N1,...```
- 作用：精确控制每块 GPU 的 tensor 分配比例（如 3,1 表示前 75% 分给 GPU0，后 25% 分给 GPU1）。
- 依赖于：--split-mode 为 layer 时生效。
- 平台：**GPU 专用**

```-mg, --main-gpu INDEX```
- 作用：
  - 在 split-mode=none 时，指定唯一使用的 GPU；
  - 在 row 模式时指定用于中间结果和 KV 的主卡。
- 默认：0
- 平台：**GPU 专用**

```--check-tensors```
- 作用：在加载模型时检查张量中是否有无效值（如 NaN）。
- 用途：模型调试或损坏模型排查。

```--override-kv KEY=TYPE:VALUE```
- 高级：允许覆盖模型元信息，比如修改 ```tokenizer.ggml.add_bos_token=bool:false```。
- 用途：热修复模型元信息、兼容老模型、不改源码修改 tokenizer 行为等。

```--no-op-offload```
- 作用：禁止将操作（op）offload 到 GPU，仅张量 offload。
- 用途：在 GPU 支持不完全或测试 CPU 算力时禁用 ```offload operator```。

## 9. LoRA 微调模块加载

```--lora FNAME```
- 作用：加载一个 LoRA adapter 文件，动态注入到基础模型中。
- 用途：低成本微调的模型集成（适合少量 GPU / CPU 微调部署）。
- 可重复：可以加载多个 LoRA 叠加。

```--lora-scaled FNAME SCALE```
- 作用：加载 LoRA，同时指定其缩放因子 SCALE（如 0.8）。
- 用途：更细致地控制 adapter 对输出影响的权重。

## 10. 控制向量（Control Vectors）
控制向量是近期新增的“方向性控制”技术，可视为 soft prompt 或指导向量。

```--control-vector FNAME```
- 作用：添加控制向量（路径为 .bin 控制向量文件）。
- 用途：如语气控制、意图控制、领域适配等。

```--control-vector-scaled FNAME SCALE```
- 作用：加载控制向量并指定缩放系数 SCALE。

```--control-vector-layer-range START END```
- 作用：仅对模型的某个层范围应用控制向量。
- 用途：实现层级局部调节，避免全局干扰。

## 11. 模型加载与来源设置

```-m, --model FNAME```
- 作用：指定本地模型文件路径。
- 默认路径规则：
  - 若未设置，则按 models/7B/ggml-model-f16.gguf；
  - 若使用 --hf-repo，则自动缓存下载到该路径。

```-mu, --model-url URL```
- 作用：通过 URL 下载模型。
- 用途：直接从远程地址拉取 GGUF 文件，而不经过 Hugging Face。
- 适合：自建模型分发站点。

## 12. 日志控制参数（Logging Options）

```--log-disable```
- 作用：完全关闭日志输出（包括 info / warn / debug）。
- 用途：静默运行，适合嵌入式部署或日志敏感场景。

```--log-file FNAME```
- 作用：将日志保存到指定文件路径。
- 用途：调试记录、错误分析或性能日志持久化。

```--log-colors```
- 作用：开启彩色日志输出。
- 建议：在支持彩色终端（如 xterm, VSCode）中开启，更易于阅读。

```-v, --verbose, --log-verbose```
- 作用：开启“无限”级别的日志（log everything）。
- 用途：调试低级问题、查看每一步行为。

```--offline```
- 作用：强制离线运行，禁止任何网络访问。
- 用途：Hugging Face 模型缓存加载或部署在无网环境时使用。

```-lv, --verbosity N```
- 作用：设置日志详细程度，N 值越大输出越多。
- 用途：比 --verbose 更细粒度，可控输出信息级别。

```--log-prefix```
- 作用：在每条日志前增加前缀（模块名等）。
- 建议：配合日志分析工具使用。

```--log-timestamps```
- 作用：在日志中添加时间戳。
- 用途：调试响应延迟、分析性能瓶颈。

## 13. 采样策略参数（Sampling Parameters）

采样参数决定模型在生成时如何选择下一个 token。控制生成的多样性、可信度、长度、重复性等特性。

```--samplers SAMPLERS```
- 作用：设置采样器组合，多个采样器按顺序应用（用分号 ; 分隔）。
- 默认值：
  - penalties;dry;top_n_sigma;top_k;typ_p;top_p;min_p;xtc;temperature
- 用法：可以删减或调整顺序以改变采样风格。

```--sampling-seq, --sampler-seq SEQUENCE```
- 简写版本：用单字符序列简化采样器设定。
- 默认值：edskypmxt（代表具体的采样器组合）。

```-s, --seed SEED```
- 默认：-1（表示使用系统随机种子）
- 用途：固定随机种子可实现输出可复现性。

```--ignore-eos```
- 作用：忽略模型输出的 end-of-sequence token，强制继续生成。
- 隐含效果：相当于设置 --logit-bias <eos-token-id>-inf，使 <eos> 不可能被选中。

## 14. 多样性控制参数

```--temp N```
- 作用：温度采样，控制随机性。
- 默认：0.2（偏保守）
- 建议：
  - 0.1 ~ 0.3: 高确定性（用于推理、摘要）
  - 0.7 ~ 1.0: 高多样性（用于创作）

```--top-k N```
- 作用：只从概率前 K 个 token 中采样。
- 默认：40
- 推荐：
  - 小值（如 10）输出保守；
  - 大值（如 100）更自由；
  - 0 表示不限制（即关闭 top-k）。

```--top-p N```
- 作用：从累积概率 >= N 的 token 中随机采样（核采样）。
- 默认：0.9
- 推荐组合：通常与 temperature 联合使用。

```--min-p N```
- 作用：强制每个 token 至少要有 P 的概率才能被考虑（filter extremely low prob）。
- 默认：0.1

```--typical N```
- 作用：Locally typical sampling，按 token 熵局部最“典型”来筛选。
- 默认：1.0（1.0 表示关闭）
- 推荐：0.8 可以提升合理性。

## 15. 重复惩罚参数

```--repeat-last-n N```
- 作用：从最后 N 个 token 中统计重复惩罚。
- 默认：64（-1 表示全上下文）
- 推荐：防止模型一直生成重复句式或 Token。

```--repeat-penalty N```
- 作用：对重复 Token 添加概率惩罚。
- 默认：1.0（无惩罚）
- 建议值：
  - 1.1：轻微惩罚；
  - 1.3+：强惩罚（可能导致语义断裂）。

```--presence-penalty N```
作用：如果 token 已出现，则整体惩罚其再次出现（适合内容生成去重）。

```--frequency-penalty N```
作用：按 token 出现频率惩罚（出现越多惩罚越重）。

## 16. 高级设置参数

DRY Sampling 系列（稳定性控制）：
```--dry-multiplier```：概率缩放因子；
```--dry-base```：最小 base 值；
```--dry-allowed-length```：允许干预长度；
```--dry-penalty-last-n```：对多少 token 应用 DRY；
```--dry-sequence-breaker STRING```：设置序列中断符（如换行、冒号等）。
- DRY 采样用于增强输出结构的稳定性与连贯性，适合对话、表格、代码生成等场景。
- XTC（Extremely Typical Completion）参数：
```--xtc-probability, --xtc-threshold```：低频但有意义 token 的选取策略。

Mirostat 采样（自适应熵调控）
```--mirostat N```
- 启用：1 为 v1，2 为 v2（建议）
- 特点：根据期望输出熵动态调整温度采样。
- 适合：需要自适应平衡输出多样性与稳定性的场景。

```--mirostat-lr N / --mirostat-ent N```
- lr (η)：学习率，控制调整步长；
- ent (τ)：目标熵（比如 5.0 表示内容既丰富又不随机）。

Logit 级别干预
```-l, --logit-bias TOKEN_ID(+/-)BIAS```
- 作用：手动调整某个 token 的输出概率。
  - +1 增加出现概率；
  - -1 降低；
  - -inf 禁止（如 --ignore-eos 内部实现就是设置 <eos>-inf）。
- 用途：定制输出方向，比如强制开头用 Hello。

语法与结构约束
```--grammar GRAMMAR / --grammar-file FNAME```
- 作用：提供 BNF 风格的生成约束语法。
- 用途：如限制输出为 Email 格式、SQL 语句、HTML 等结构。

```--json-schema SCHEMA / --json-schema-file FILE```
- 作用：限制输出必须符合 JSON Schema。
- 用途：结构化数据生成，如 JSON API 响应、配置项生成。

## 17. 多模态投影器相关参数

```--mmproj FILE```
- 作用：指定多模态投影器（Multimodal Projector）模型文件的路径，通常是 .gguf 格式，例如 ```gemma-3-mmproj-model-f16.gguf```。
- 用途：将输入图像 / 音频编码成 transformer 能识别的嵌入向量。
- 是否必选：如果你不使用 --hf 模型下载，则这是**必选**参数。
- 平台支持：CPU 和 GPU 都可用，可以配合 --no-mmproj-offload 控制。

```--mmproj-url URL```
- 作用：从远程 URL 下载 projector 文件（功能类似于 --model-url）。
- 用途：不手动下载时自动获取 Projector 文件。

```--no-mmproj```
- 作用：禁用 projector，即使模型 metadata 中标记了 mmproj 也不加载。
- 用途：适用于需要纯文本测试或调试模型行为时排除图像处理逻辑。

```--no-mmproj-offload```
- 作用：强制 projector 在 CPU 上运行，不 offload 到 GPU。
- 用途：当 GPU 资源紧张或不稳定时可使用此选项避免多模态模块使用 GPU 显存。

**多模态输入参数**
```--image FILE / --audio FILE```
- 作用：指定要输入的图像或音频文件路径（可以重复使用此参数传入多个文件）。
- 用途：
  - 图像输入支持多模态视觉模型（如 MiniCPM-V、Gemma3-Vision）；
  - 音频输入支持语音识别模型（如 Whisper、VALL-E）。
- 可重复：支持多个 --image 和 --audio 同时输入。
- 使用建议：
  - 单图使用：--image ./math.jpg
  - 多图组合：--image img1.png --image img2.jpg

**自定义对话模板（Chat Template）**
```--chat-template TEMPLATE_NAME```
- 作用：指定要使用的 Jinja 风格对话模板（影响系统 prompt 格式）。
- 默认行为：使用模型 metadata 中自带模板。
- 常用内置模板（涵盖常见模型）：
  - gemma, llama2, llama3, chatml, mistral-v3, openchat, vicuna, zephyr, phi4, etc.
- 用途：
  - 保证多轮对话格式正确（如 <|im_start|>, <|user|> 等 tag）；
  - 与具体模型训练对齐，提高输出一致性。

**❗说明：Chat 模板禁用机制**
- 如果你使用了 --suffix 或 --prefix，则模板将被禁用，提示以裸文本格式输入。
