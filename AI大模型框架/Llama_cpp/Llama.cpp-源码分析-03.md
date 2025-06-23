# Llama.cpp 源码分析-03 GGUF 模型文件格式分析

## 1. GGUF 模型文件结构

### 1.1 GGUF 简介

GGUF（GGML Unified Format）是一种用于保存大语言模型（LLM）的 统一模型文件格式，它具备以下特性：
- 自包含（不需要额外的 config.json、tokenizer.model）
- 支持结构信息、模型权重、tokenizer 元数据打包
- 被 llama.cpp 等多个引擎支持，加载速度快，适配性强


### 1.2 GGUF 结构

![](./img/GGUF.png)

**文件开头：Header（文件头）**

| 偏移量位置 | 字段名            | 类型       | 含义                                                            |
| :--------: | :---------------: | :--------: | :-------------------------------------------------------------: |
| 0x00       | Magic Number      | `char[4]`  | 固定为 `GGUF`，用于识别文件格式。ASCII 为 `0x47 0x47 0x55 0x46` |
| 0x04       | GGUF Version      | `uint32_t` | 当前版本号，通常是 `3`（截至 2024）                             |
| 0x08       | Tensor Count      | `uint64_t` | 后面要加载的 tensor 数量                                        |
| 0x10       | Metadata KV Count | `uint64_t` | 后面 metadata 区的键值对数量                                    |


**Metadata 区域：键值对元信息区**

这个区域就是图中绿色框框：
- 存储的是 <key, value> 形式的 模型元数据。
- 每个 key 是 GGUF string（定长编码字符串），value 支持多种类型（参考 gguf_type 枚举）。
- 这些键值对存储的是模型的高层配置信息，比如：
- 这些字段是原来 config.json、tokenizer_config.json 中的内容，都被内嵌到了 GGUF 中，不再需要额外文件。


**Tensor Info 区域：张量目录表**

这个区域是图中右上角的紫色框：
- 每个张量（参数）在 GGUF 中都有一条记录，字段包括：

| 字段名         | 类型         | 含义                                          |
| :------------: | :----------: | :-------------------------------------------: |
| `name`         | GGUF string  | 张量名称，例如 `blk.0.ffn_gate.weight`        |
| `n_dimensions` | uint32\_t    | 维度数量（如 2 表示是矩阵）                   |
| `dimensions[]` | uint64\_t\[] | 各维度的大小（如 `[4096, 32000]`）            |
| `type`         | uint32\_t    | 张量数据的类型，比如 10 表示 `GGML_TYPE_Q4_K` |
| `offset`       | uint64\_t    | 数据段中该张量的起始偏移（单位是字节）        |


**张量数据段（rest of the file）**

图中黑色右侧部分：
- 存放的是所有 tensor 的原始数据块（被量化压缩或原始浮点数）。
- 它的顺序和上面 tensor info 一一对应，可通过 offset 字段来访问。
- 数据可能采用不同量化格式（如 Q4_K, Q6_K, F32, I8），具体由 type 指定。

## 2. gguf_context 类结构分析

包含对 gguf_kv 与 gguf_info 类的示例

```cpp
struct gguf_context {
    uint32_t version = GGUF_VERSION;            // GGUF 文件版本号
    std::vector<struct gguf_kv> kv;             // 元数据 key-value 列表
    std::vector<struct gguf_tensor_info> info;
    size_t alignment = GGUF_DEFAULT_ALIGNMENT;
    size_t offset    = 0; // 整个张量数据块在文件中的起始偏移 - offset of `data` from beginning of file
    size_t size      = 0; // 张量数据块的总大小 - size of `data` in bytes
    void * data = nullptr;  // 指向 mmap 或 malloc 读取的张量数据的首地址
};
```

- kv 与 info 分别对应了GGUF文件中的 Metadata 区域以及 Tensor Info 区域
- gguf_context.data + info[i].offset 可以定位到模型中第 i 个 tensor 的数据块





