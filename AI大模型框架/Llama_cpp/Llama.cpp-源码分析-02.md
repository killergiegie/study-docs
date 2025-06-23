# Llama.cpp 源码分析-02 Tensor 量化格式分析

## 1. ggml_type_traits 结构体

```cpp
// 用于描述每种数据类型的元信息
    struct ggml_type_traits {
        const char *             type_name;            // 类型名，比如 "Q4_0"
        int64_t                  blck_size;            // 每个 block 里含有多少逻辑元素（tokens）
        int64_t                  blck_size_interleave; // interleave elements in blocks
        size_t                   type_size;            // 每个 block 在内存中占用的字节数
        bool                     is_quantized;         // 是否是量化类型
        ggml_to_float_t          to_float;             // 把量化数据 → 转换为float32
        ggml_from_float_t        from_float_ref;       // 把 float32 → 量化数据
    };
```

数组 type_traits：
```cpp
static const struct ggml_type_traits type_traits[GGML_TYPE_COUNT] 
```
每个元素表示 一个 ggml 支持的“数据类型” 的属性，比如：
- 是不是量化的？
- 一个 block 包含多少元素？
- 一个 block 占多少字节？
- 怎么从 float 转换过来，怎么还原成 float？

### 1.1 Q4_0 量化 block 结构

#### 1.1.1 Q4_0 block 结构分析

**Q4 量化本质上是：**
用低精度整数（如 4-bit）+ 缩放系数 来近似表示一组高精度 float 数据。
- 4-bit 本身没有浮点数表示能力，只能表示 [0 ~ 15] 这样的整数 → 必须乘以缩放系数 d。
- 逐个 float 拿一个缩放因子太浪费空间 → 每个 float 都带一个 d 得不偿失。
- 所以：多个 float 共用一个 d，就产生了 “block” 的概念！
```cpp
#define QK4_0 32
// Q4_0量化的一个block结构，包含：2字节的缩放因子d以及16字节(32个4-bits)的量化值qs
typedef struct { 
    ggml_half d;           // delta
    uint8_t qs[QK4_0 / 2]; // nibbles / quants
} block_q4_0; 
```
**表示的 float 数量：**
- qs[16] ⇒ 每字节两个 4-bit ⇒ 共 32 个 4-bit 量化值
- 所以：一个 block_q4_0 对应 32 个 float 值

#### 1.1.2 Q4_0 量化与解量化流程

**量化（float → Q4）**

- 从 32 个 float 中取出 最大绝对值（max）。
- 计算缩放因子 d = max / -8，用于线性压缩。
- 每个值按公式 q = round(x/d) + 8 映射到 [0, 15]，压成 4-bit。
- 将 32 个 4-bit 值合并为 16 字节（qs[16]），另存 d 为 ggml_half

**解量化（Q4 → float）**

- 每个 4-bit 解成 q ∈ [0, 15]，再转为 q - 8 ∈ [-8, 7]，最后乘上 d 还原为近似 float。



### 1.2 Q3_K 量化 block 结构

#### 1.2.1 Q3_K block 结构分析

**Q3_K 量化本质上是：**
- 把 256 个 float32 的权重压缩成 远远少于 1024 字节（因为每个 float 是 4 字节，256 × 4 = 1024 字节）。
- 而 block_q3_K 只用 110 字节 来表示这 256 个数
- 这是通过 量化为 3-bit 整数 + 缩放 实现的：
>用更小的整数代替 float，再乘个缩放值，来“近似还原”原始 float。

```cpp
#define QK_K 256
typedef struct {
    uint8_t hmask[32];     // 每个元素的高 1 bit（256 个元素 → 每个 1 bit → 32 字节）
    uint8_t qs[64];        // 每个元素的低 2 bit（256 × 2 bit → packed 成 64 字节）
    uint8_t scales[12];    // 把 256 个元素分成 16 组，每组一个缩放值（每个缩放值用 6bit 表示 → 共 12 字节）
    ggml_half d;           // 全局缩放值 d（float 的半精度，2 字节）
} block_q3_K;
```

其权重表达为：\(x = a * q\)
- \(q\): 为存储的 3-bit 量化值
- \(a\): 为缩放系数d


#### 1.2.2 Q3_K 量化与解量化流程

**量化（float → Q3_K）**

- 将 256 个 float 拆成 16 组（每组 16 个），逐组拟合出最优缩放因子 s。
- 选出所有缩放因子中绝对值最大的作为全局缩放 d，并保存其反值为 ggml_half 存入 d。
- 每组缩放因子 s 会被编码为 6-bit 精度，拆分为低 4 bit 和高 2 bit，16 组共压缩为 scales[12]。
- 每个 float 用 (x / (d * s)) 计算得到近似量化值 q ∈ [-4, 3]。
- 将 q 拆成：
  - 低 2 bit 存入 qs[64]，4 个 q 值拼成 1 字节（共 256 个 → 64 字节）
  - 高 1 bit 存入 hmask[32]，每 bit 表示一个 q 值是否在 [-4,-1]（高位为 0）还是 [0,3]（高位为 1）

**解量化（Q3_K → float）**

- 解出 d 和每组 6-bit 精度的 scale，还原每组实际缩放因子为 s = (scale - 32) × d
- 对每个 q：
  - 从 qs 中提取低 2 bit，得到值 ∈ [0,3]
  - 从 hmask 中提取高 1 bit，判断是否加上 -4，得到最终 q ∈ [-4,3]
- 每个 q × s × d，还原为近似 float 值

### 1.3 Q4_K 量化 block 结构

#### 1.3.1 Q4_K block 结构分析

**Q4_K 量化本质上是：**

是 GGML 的另一种量化格式 block_q4_K，可以将它与之前的 block_q4_0 和 block_q3_K 一起理解为越来越精细的量化策略演化

| 格式   | 每权重比特  | 缩放方式         | 是否支持偏移 b（zero-point） | 是否分组 scale | 是否非线性编码 |
| ------ | ----------- | ---------------- | ---------------------------- | -------------- | -------------- |
| `q4_0` | 4.0 bits    | 每 block 一个 d  | ❌ 否                         | ❌ 否           | ❌ 否           |
| `q3_K` | \~3.44 bits | 每组一个 scale   | ❌ 否                         | ✅ 16 组        | ✅ 非线性编码   |
| `q4_K` | 4.5 bits    | 每组 scale + min | ✅ 是，x ≈ d·q + b            | ✅ 8 组         | ✅ 非线性编码   |


```cpp
typedef struct {
    GGML_EXTENSION union {
        struct {
            ggml_half d;     // 缩放 scale 系数（乘a）- super-block scale for quantized scales
            ggml_half dmin;  // 偏移 min 值（加b）- super-block scale for quantized mins
        } GGML_COMMON_AGGR_S;
        ggml_half2 dm;
    } GGML_COMMON_AGGR_U;
    uint8_t scales[K_SCALE_SIZE]; // 存储 8 个 scale 和 8 个 min，共 16 个 6-bit 编码值，占用12字节
    uint8_t qs[QK_K / 2];         // 存储 256 个权重的 4-bit 量化值，每个元素占 4 bits，共128字节
} block_q4_K;
```

其权重表达为：\(x = a * q + b\)
- \(q\) : 为存储的 4-bit 量化值
- \(a\) : 为缩放系数 scale（用 d 和 scales 解码）
- \(b\) : 为 min（用 dmin 和 scales 解码），更接近线性映射，还原更精确

#### 1.3.2 Q4_K 量化与解量化流程

**量化（float → Q4_K）**
- 将一行数据按 32 个 float 为一组，共 8 组（256 float） 分段处理
- 对每组计算：
  - scale_j：用于量化该组值的缩放系数
  - min_j：该组 float 的最小值，用作非对称偏移
  - 每个值映射为 q = round((x + min_j) / scale_j)，并裁剪至 [0, 15]
- 所有量化值 q ∈ [0, 15] 用 4-bit 表示，组合为 qs[128]（每两值打包为 1 字节）
- 每组的 scale_j 和 min_j 被编码进 scales[16]，使用 packed 的 6-bit 格式：
  - 低 6 位 + 高 2 位 分别打包，精度允许范围内压缩表示（参考 get_scale_min_k4() 的压缩逻辑）
- 整个 block 中所有 scale 使用最大值 max_scale，存储其倒数为 d = max_scale / 63
- 所有 min 同理，用 max_min 得到 dmin = max_min / 63
- 解码时通过 d × sc_raw 与 dmin × m_raw 恢复 scale 和 min

**解量化（Q4_K → float）**

- 对每 64 个量化值：
  - 拆出对应两个 6-bit scale（sc）与 min（m）索引
  - 将实际缩放值还原为 scale_real = d × sc，min_real = dmin × m
  - 每个量化值（4-bit，范围 0~15）：
  - \( x = q \cdot \text{scale_real} - \text{min_real} \)
- 所有量化值存储在 qs[128] 中，每字节两值（低 4-bit + 高 4-bit）


**与 Q4_0 的主要区别**

| 特性        | Q4\_0                         | Q4\_K                                  |
| :---------: | :---------------------------: | :------------------------------------: |
| 量化方式    | 对称：x ≈ d × q               | **非对称**：x ≈ d × q − m              |
| 缩放粒度    | 全 block 一个 d               | 每 32 个值一个 `scale` 与 `min`        |
| 偏移（min） | 无偏移，中心在 0              | 有偏移，每组独立存储 min               |
| 精度        | 4-bit × 32，精度较低          | 4-bit × 256，分组缩放 + 偏移，精度更高 |



### 1.4 IQ4_NL 与 IQ4_XS 量化 block 结构

**什么是 IQ 量化？**

IQ（Interleaved Quantization）是一种非线性、块级量化（block-wise quantization）方法，目的是：
- 在尽量不损失精度的情况下进一步压缩模型大小
- 改进 Q4（4-bit）格式的表达能力
- 适配更复杂的分布，比如激活值或权重不是线性分布的情况

#### 1.4.1 IQ4_NL 非线性量化（Non-linear Quantization）

IQ4_NL的block结构上与Q4_0类似，将32个float作为一个block，量化为一个4-bit

一个block存储32个Q4，一组block共用一个scale，共占用 18 字节

```cpp
// Non-linear quants
#define QK4_NL 32
typedef struct {
    ggml_half d;            // delta 缩放因子，2字节
    uint8_t qs[QK4_NL/2];   // 量化值，32个4-bit的量化值，共16字节
} block_iq4_nl;
```


#### 1.4.2 IQ4_NL 量化与解量化流程

**量化（float → IQ4_NL）**

- 将输入向量按 32 个 float 为一块 super-block 处理，每块量化为一个 block_iq4_nl
- 采用 非线性量化表 kvalues_iq4nl[16]，而非均匀间隔值
  - 这些是手动挑选的 16 个整数，覆盖 [-127, +113]，分布更密集地覆盖常用区间，提高小值精度
- 步骤如下：
  - 计算 sigma²（平均能量）：为每个元素构造权重（用于最小化量化误差）
  - 对每 16/32 个 float：
    - 初始化权重（可选加权）
    - 初始估算缩放因子 d = max / kvalue_max（大致拟合）
    - 使用 best_index_int8 在 kvalues_iq4nl 中找到最合适的量化值索引（非线性查找）
    - 拟合最优 scale d = Σ(qx·x) / Σ(q²) 最小化平方误差（WLS 形式）
    - 若 ntry > 0：尝试更优的 d（即尝试多个变种 id 看是否更优）
  - 得到最终缩放值 d，将其保存为 ggml_half（16-bit）存入 block_iq4_nl.d
  - 所有索引值（0~15）打包为 16 字节的 qs[16]，每字节两个 4-bit 编码值

**解量化（IQ4_NL → float）**

- 从 qs[16] 中取出每个 4-bit 编码值，查表获取对应 kvalues_iq4nl 值（整型）
- 乘上缩放因子 d 得到近似还原的 float 值：
  - \( x_j = d * kvalues [索引] \)

#### 1.4.3 IQ4_XS 非线性量化（Non-linear Quantization）

IQ4_XS 是一种 **混合精度+超块重标定** 的非线性量化格式，其设计目标是：
- 在保持较低比特（4bit）前提下，引入更加灵活的 scale 编码，以提高重构精度。

```cpp
typedef struct {
    ggml_half d;                // 超块缩放因子基值（super-block base scale）
    uint16_t scales_h;          // 8 个子块的 scale 高 2 bit，每个子块 2 bit，共 16 bit
    uint8_t  scales_l[QK_K/64]; // 每个子块一个 4 bit scale 低位，共 QK_K/64 = 8 字节
    uint8_t  qs[QK_K/2];        // 4-bit 编码数据，每两个值占 1 字节，共 128 字节
} block_iq4_xs;
```

其中，QK_K == 256，也就是说：
- 一个 block 表示 256 个 float
- 被划分为 8 个子块（每个 32 个元素）


#### 1.4.4 IQ4_XS 量化与解量化流程


**量化（float → IQ4_XS）**

- 输入张量被分成 super-block（每 256 个 float），即每个 block 包含 256 个元素。
- 每个 super-block 再划分为 8 个子块（每块 32 个 float）。
- 量化过程如下：
  - 对每个子块（32个 float）进行非线性拟合：
    - 使用 kvalues_iq4nl 表中的 16 个非线性 int8 值（如 -127,...,113）。
    - 通过最小均方误差搜索，确定最优拟合缩放因子 scale，并保存为 scales[8]。
  - 全局 scale 对齐（压缩 8 个 scale）：
    - 找出 scales 中绝对值最大的值 max_scale，并设为 d。
    - 所有子块的 scale 都转换为整数因子 ls ∈ [-32, 31]，表示为 scale_i ≈ d × ls。
    - 将这 8 个整数 ls 编码为：
      - 低 4 位存在 scales_l[8] 中（每个占 4 bit，共 8 字节）。
      - 高 2 位存在 scales_h (16 bit) 中（8 × 2 = 16 位）。
  - 将每个 float 映射到非线性查表值的 index：
    - 对每个 float 值 x，按比例缩放并寻找最接近 kvalues_iq4nl[] 表的项。
    - 将对应 index 存入 qs[128]，每两个值打包为一个 byte（4-bit 编码）。


**解量化（IQ4_XS → float）**

对每个 block（256 元素）进行解压：
- 提取全局缩放因子：
  - d ← x[i].d，为 float16 → float32 的全局基值。
- 提取每个子块的局部 scale：
  - 从 scales_l[] 和 scales_h 组合出每个子块的整数缩放因子 ls ∈ [0, 63]。
  - 实际缩放值为 dl = d × (ls - 32)。
- 解码量化值：
  - 每 2 个编码值占 1 字节，拆为两个 4-bit 索引。
  - 查表 kvalues_iq4nl[index] 得到原始数值，再乘上 dl 还原为近似 float。



### 1.x ggml数据类型属性表（部分

| ggml\_type        | blck\_size | type\_size        | 对应数据结构 | 是否量化 |
| :-------------:   | :--------: | :---------------: | :----------: | :------: |
| `GGML_TYPE_I64`   | 1          | 8 (int64\_t)      | long int     | ❌ 否     |
| `GGML_TYPE_I32`   | 1          | 4 (int32\_t)      | int          | ❌ 否     |
| `GGML_TYPE_I16`   | 1          | 2 (int16\_t)      | short        | ❌ 否     |
| `GGML_TYPE_I8`    | 1          | 1 (int8\_t)       | int8         | ❌ 否     |
| `GGML_TYPE_F64`   | 1          | 8 (double)        | double       | ❌ 否     |
| `GGML_TYPE_F32`   | 1          | 4 (float)         | float        | ❌ 否     |
| `GGML_TYPE_F16`   | 1          | 2 (`ggml_fp16_t`) | uint16       | ❌ 否     |
| `GGML_TYPE_Q4_0`  | 32         | 18                | block_q4_0   | ✅ 是     |
| `GGML_TYPE_Q4_1`  | 32         | 20                | block_q4_1   | ✅ 是     |
| `GGML_TYPE_Q3_K`  | 256        | 110               | block_q3_K   | ✅ 是     |
| `GGML_TYPE_Q4_K`  | 256        | 144               | block_q4_K   | ✅ 是     |
| `GGML_TYPE_IQ4_NL`| 32         | 18                | block_iq4_nl | ✅ 是     |
| `GGML_TYPE_IQ4_XS`| 256        | 136               | block_iq4_xs | ✅ 是     |




## 2. ggml_tensor 结构体 