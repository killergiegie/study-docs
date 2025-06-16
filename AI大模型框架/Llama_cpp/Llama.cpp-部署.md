# Llama.cpp 部署与使用

**参考网页：**
量化含义：https://zhuanlan.zhihu.com/p/24560784106
Qwen部署方法：https://qwen.readthedocs.io/zh-cn/latest/run_locally/llama.cpp.html

**模型库：**
https://www.modelscope.cn/models/bartowski/google_gemma-3-12b-it-GGUF/files
https://www.modelscope.cn/models/ggml-org/Qwen2.5-Omni-7B-GGUF/files

## 1. 环境配置
### 1.1 环境需求
- cmake ≥ 13.4
- GCC 最好大于 gcc-10 或 gcc-11
- 如果是GPU推理的话，CUDA 尽量最新，目前使用CUDA12.0，需要gcc-11

### 1.2 编译Llama.cpp源码（纯CPU版本）
```bash
cmake -B build -S . \
    -DCMAKE_BUILD_TYPE=Release \
    -DLLAMA_CURL=OFF \
    -DGGML_NATIVE=OFF \
    -DGGML_USE_VNNI=OFF \
    -DGGML_CUDA=OFF

cd build
make -j16
# 没有指定安装路径，不要 make install
```

### 1.3 编译Llama.cpp源码（GPU + CUDA12.0版本）
```bash
cmake -B build -S . \
    -DCMAKE_BUILD_TYPE=Release \
    -DLLAMA_CURL=OFF \
    -DGGML_NATIVE=OFF \
    -DGGML_USE_VNNI=OFF \
    -DGGML_CUDA=ON \
    -DGGML_CUDA_ENABLE_UNIFIED_MEMORY=1 

cd build
make -j16
# 没有指定安装路径，不要 make install
```

### 1.4 参数配置明细
| 参数项                                | 含义说明                                                                |
| ------------------------------------- | ----------------------------------------------------------------------- |
| `-DCMAKE_BUILD_TYPE=Release`          | 启用 Release 模式编译，开启优化（如 `-O3`），用于部署环境               |
| `-DLLAMA_CURL=OFF`                    | 禁用 curl 支持，不启用在线模型下载功能，减少依赖                        |
| `-DGGML_NATIVE=OFF`                   | 禁用本地 CPU 优化（如 AVX2/AVX512），提高跨平台兼容性                   |
| `-DGGML_USE_VNNI=OFF`                 | 禁用 VNNI（Vector Neural Network Instructions），仅部分 Intel CPU 支持  |
| `-DGGML_CUDA=ON`                      | 启用 CUDA 后端，使用 NVIDIA GPU 进行推理加速                            |
| `-DGGML_CUDA_ENABLE_UNIFIED_MEMORY=1` | 启用 CUDA 统一内存（Unified Memory），降低显存 OOM 风险，提升内存灵活性 |



## 2. 纯文本推理（命令行工具 llama-cli）
从 ModelScope 上下载 GGUF 格式的模型文件

### 2.1 Qwen3 + GPU1 推理
仅使用 GPU1 进行推理
```bash
CUDA_VISIBLE_DEVICES=1 nice -n 10 ~/LLM/llama.cpp-master/build/bin/llama-cli \
  -m Qwen3-8B-Q4_K_M.gguf \
  --jinja --color \
  -ngl 36 -fa -sm row \
  --temp 0.8 --top-k 40 --top-p 0.95 --presence-penalty 1.2 \
  -t 8 \
  -c 4096 \
  -n 512 
```

### 2.2 gemma-3 + CPU 推理
使用 CPU + 单线程进行推理
```bash
~/LLM/llama.cpp-cpu/build/bin/llama-cli \
  -m google_gemma-3-12b-it-qat-bf16.gguf \
  --jinja --color \
  --temp 0.8 --top-k 40 --top-p 0.95 --presence-penalty 1.2 \
  -t 1 \
  -c 2048 \
  -n 2048 
```

## 3. 多模态推理（命令行工具 llama-mtmd-cli）

### 3.1 Qwen2.5-VL-7B-Q4_K_M 模型 + GPU 推理
仅使用 GPU1 进行推理
```bash
CUDA_VISIBLE_DEVICES=1 nice -n 10 ~/LLM/llama.cpp-master/build/bin/llama-mtmd-cli \
  -m ./Qwen2.5-VL-7B-Instruct-Q4_K_M.gguf \
  --mmproj ./Qwen2.5_mmproj-F16.gguf \
  -p "回答这题。首先复述出题目，请详细推理并输出尽量多内容；遇到数学题时请逐步推理；所有数学表达式必须使用标准 LaTeX 语法，采用 \( \) 或 \[ \] 包裹；输出内容采用三段式结构，依次为：“解题步骤”、“结论”与“知识点总结”，每段以固定标题开头，便于分段解析和结构化展示" \
  --image ./math_test/3.png \
  -ngl 99 -fa -sm row \
  --temp 0.8 --top-k 40 --top-p 0.95 --presence-penalty 1.2 \
  -t 24 \
  -c 8192 \
  -n 8192 

~/LLM/llama.cpp-master/build/bin/llama-mtmd-cli \
  -m ./Qwen2.5-VL-7B-Instruct-Q4_K_M.gguf \
  --mmproj ./Qwen2.5_mmproj-F16.gguf \
  -p "回答这题。首先复述出题目，请详细推理并输出尽量多内容；遇到数学题时请逐步推理；所有数学表达式必须使用标准 LaTeX 语法，采用 \( \) 或 \[ \] 包裹；输出内容采用三段式结构，依次为：“解题步骤”、“结论”与“知识点总结”，每段以固定标题开头，便于分段解析和结构化展示" \
  --image ./math_test/3.png \
  --temp 0.8 --top-k 40 --top-p 0.95 \
  -t 24 \
  -c 8192 \
  -n 8192 
```

### 3.2 Gemma-3-12b-it 模型

#### 3.2.1 Gemma-3-12b-it-Q4_K_M 模型 + GPU 推理
仅使用 GPU1 进行推理
```bash
CUDA_VISIBLE_DEVICES=1 nice -n 10 ~/LLM/llama.cpp-master/build/bin/llama-mtmd-cli \
  -m ./gemma-3-12b-it-Q4_K_M.gguf \
  --mmproj ./gemma-3-mmproj-model-f16.gguf \
  -p "回答这道题，首先复述出题目，请详细推理并输出尽量多内容，部分题目可能不止一个解，有需要分类讨论的可能；遇到数学题时请逐步推理；所有数学表达式必须使用LaTeX语法，必须采用 \(\) 或 \[\] 包裹起来；输出内容采用三段式结构，依次为：“解题步骤”、“结论”与“知识点总结”，每段以固定标题开头，便于分段解析和结构化展示" \
  --image ./math_test/geometry.png \
  -ngl 99 -fa -sm row \
  --temp 1.0 --top-k 40 --top-p 0.95 --presence-penalty 1.2 \
  -t 8 \
  -c 4096 \
  -n 4096 
```

#### 3.2.2 Gemma-3-12b-it-Q4_K_M 模型 + CPU 推理
使用8线程进行推理

```bash
/bin/time ~/LLM/llama.cpp-cpu/build/bin/llama-mtmd-cli \
  -m ./gemma-3-12b-it-Q4_K_M.gguf \
  --mmproj ./gemma-3-mmproj-model-f16.gguf \
  -p "回答这道题，首先复述出题目，请详细推理并输出尽量多内容，部分题目可能不止一个解，有需要分类讨论的可能；遇到数学题时请逐步推理；所有数学表达式必须使用LaTeX语法，必须采用 \(\) 或 \[\] 包裹起来；输出内容采用三段式结构，依次为：“解题步骤”、“结论”与“知识点总结”，每段以固定标题开头，便于分段解析和结构化展示" \
  --image ./math_test/6.png \
  --temp 1.0 --top-k 40 --top-p 0.95 --presence-penalty 1.2 \
  -t 8 \
  -c 2048 \
  -n 2048 

/bin/time ~/LLM/llama.cpp-cpu/build/bin/llama-mtmd-cli \
  -m ./gemma-3-12b-it-Q4_K_M.gguf \
  --mmproj ./gemma-3-mmproj-model-f16.gguf \
  -p "回答这道题，首先复述出题目，部分题目可能不止一个解，有需要分类讨论的可能；遇到数学题时请逐步推理；所有数学表达式必须使用LaTeX语法，必须采用 \(\) 或 \[\] 包裹起来；输出内容采用三段式结构，依次为：“解题步骤”、“结论”与“知识点总结”，每段以固定标题开头，便于分段解析和结构化展示" \
  --image ./math_test/6.png \
  --temp 1.0 --top-k 40 --top-p 0.95 --presence-penalty 1.2 \
  -t 8 \
  --parallel 1 \
  --repeat-last-n 16 \
  --rope-scale 1 \
  --no-mmproj-offload \
  --cache-type-k iq4_nl \
  --cache-type-v f16 \
  -c 2048 \
  -n 2048 
```

#### 3.2.2 Gemma-3-12b-it-IQ3 模型 + CPU 推理

Q4_K_M  性能：4.46 tokens/s + Mem 14.26 GB
IQ4_XS  性能：4.10 tokens/s + Mem  8.77 GB
Q3_K_S  性能：4.38 tokens/s + Mem  7.75 GB
IQ3_M   性能：2.69 tokens/s + Mem  8.70 GB

```bash
/bin/time ~/LLM/llama.cpp-cpu/build/bin/llama-mtmd-cli \
  -m ./gemma-3-GGUF/google_gemma-3-12b-it-Q3_K_XL.gguf \
  --mmproj ./gemma-3-mmproj-model-f16.gguf \
  -p "回答这道题，首先复述出题目，部分题目可能不止一个解，有需要分类讨论的可能；遇到数学题时请逐步推理；所有数学表达式必须使用LaTeX语法，必须采用 \(\) 或 \[\] 包裹起来；输出内容采用三段式结构，依次为：“解题步骤”、“结论”与“知识点总结”，每段以固定标题开头，便于分段解析和结构化展示" \
  --image ./math_test/6.png \
  --temp 1.0 --top-k 40 --top-p 0.95 --presence-penalty 1.2 \
  -t 8 \
  -c 2048 \
  -n 2048 
```


#### 3.2.3 Gemma-3-12b-it-bf16 模型 + CPU 推理
使用8线程进行推理
```bash
~/LLM/llama.cpp-cpu/build/bin/llama-mtmd-cli \
  -m ./gemma-3-12b-it-Q4_K_M.gguf \
  --mmproj ./mmproj-google_gemma-3-12b-it-bf16.gguf \
  -p "回答这道题，首先复述出题目，部分题目可能不止一个解，有需要分类讨论的可能；遇到数学题时请逐步推理；所有数学表达式必须使用LaTeX语法，必须采用 \(\) 或 \[\] 包裹起来；输出内容采用三段式结构，依次为：“解题步骤”、“结论”与“知识点总结”，每段以固定标题开头，便于分段解析和结构化展示" \
  --image ./math_test/8.png \
  --temp 1.0 --top-k 40 --top-p 0.95 \
  -t 24 \
  -c 4096 \
  -n 4096 
```


### 3.3 InternVL3-8B-Q4_K_M 模型 + CPU 推理
使用8线程进行推理
```bash
/bin/time ~/LLM/llama.cpp-cpu/build/bin/llama-mtmd-cli \
  -m ./intern/InternVL3-8B-Instruct-Q4_K_M.gguf \
  --mmproj ./intern/mmproj-InternVL3-8B-Instruct-f16.gguf \
  -p "回答这道题，首先复述出题目，部分题目可能不止一个解，有需要分类讨论的可能；遇到数学题时请逐步推理；所有数学表达式必须使用LaTeX语法，必须采用 \(\) 或 \[\] 包裹起来；输出内容采用三段式结构，依次为：“解题步骤”、“结论”与“知识点总结”，每段以固定标题开头，便于分段解析和结构化展示" \
  --image ./math_test/1.png \
  --temp 0.6 --top-k 40 --top-p 0.95 \
  -t 8 \
  -c 4096 \
  -n 4096 
```

性能：8.6 tokens/s + Mem 8.3 GB

### 3.4 MiMo-VL-7B-RL-Q4_K_M 模型 + CPU 推理
使用8线程进行推理
```bash
/bin/time ~/LLM/llama.cpp-cpu/build/bin/llama-mtmd-cli \
  -m ./mimo/MiMo-VL-7B-RL-Q4_K_M.gguf \
  --mmproj ./mimo/mmproj-F16.gguf \
  -p "回答这道题，首先复述出题目，部分题目可能不止一个解，有需要分类讨论的可能；遇到数学题时请逐步推理；所有数学表达式必须使用LaTeX语法，必须采用 \(\) 或 \[\] 包裹起来；输出内容采用三段式结构，依次为：“解题步骤”、“结论”与“知识点总结”，每段以固定标题开头，便于分段解析和结构化展示" \
  --image ./math_test/3.png \
  --temp 0.6 --top-k 40 --top-p 0.95 \
  --repeat-last-n 1024 --repeat-penalty 1.2 \
  -t 24 \
  -c 8192 \
  -n 8192 
```


## 4. 运行参数总览表
| 参数                       | 适用平台 | 类型        | 含义说明                                                                |
| -------------------------- | -------- | ----------- | ----------------------------------------------------------------------- |
| `-t 24` / `--threads 24`   | CPU      | 通用        | **指定使用的线程数**，通常设为 CPU 核心数，提升并行性能                 |
| `--mlock`                  | CPU      | 通用        | **锁定模型到内存**，避免 swap，提升稳定性（需 root 权限或系统支持）     |
| `--numa`                   | CPU      | 通用        | 启用 **NUMA-aware 线程绑定**，适用于多 CPU NUMA 架构                    |
| `--no-mmap`                | CPU      | 通用        | 禁用 `mmap`，使用传统读取模型方式，某些系统可能更快                     |
| `--no-prompt-cache`        | CPU      | 通用        | 禁用 prompt 缓存，可减少内存使用，但会重复计算 prompt                   |
| `-c 2048`                  | CPU/GPU  | 通用        | 设置 **上下文长度**（token 数），受模型限制，建议不要超过 GGUF 中定义值 |
| `-n 1024`                  | CPU/GPU  | 通用        | 生成 **最大 token 数量**                                                |
| `--temp 0.8`               | CPU/GPU  | 通用        | **采样温度**（越高越随机，0.7\~1.0 通常较自然）                         |
| `--top-k 40`               | CPU/GPU  | 通用        | 每步生成时在 **概率前 k 个 token 中采样**                               |
| `--top-p 0.95`             | CPU/GPU  | 通用        | **Top-p 采样（nucleus sampling）**，在累计概率前 95% 范围中采样         |
| `--presence-penalty 1.2`   | CPU/GPU  | 通用        | 对 **重复 token 添加惩罚**，提高多样性                                  |
| `-m model.gguf`            | CPU/GPU  | 通用        | 指定加载的 **GGUF 模型文件**                                            |
| `--jinja`                  | CPU/GPU  | 通用        | 使用模型内置的 **对话模板**（适合 Chat 类模型）                         |
| `--color`                  | CPU/GPU  | 通用        | 启用终端 **彩色输出**，增强可读性                                       |
| `--mmproj mmproj.gguf`     | CPU/GPU  | 多模态      | 加载 **多模态投影模型**（如图文匹配的映射网络），仅适用于多模态模型     |
| `--image image.png`        | CPU/GPU  | 多模态      | 指定输入图像（支持 PNG、JPEG 等），用于 **图文联合推理**                |
| `-p "<prompt>"`            | CPU/GPU  | 文本/多模态 | 提示词（prompt），如果配合 `--image`，则为图文联合输入                  |


GPU专用（需 -DGGML_CUDA=ON 构建）

| 参数         | 适用平台 | 含义说明                                                       |
| ------------ | -------- | -------------------------------------------------------------- |
| `-ngl 99`    | GPU      | 将 99 个 transformer 层卸载到 GPU（**最大化加速**）            |
| `-dev cuda1` | GPU      | 指定使用哪张 GPU（如 `cuda0`、`cuda1`）                        |
| `-fa`        | GPU      | 启用 **Flash Attention**，适合大模型和长上下文（CUDA >= 11.7） |
| `-sm row`    | GPU      | 多 GPU 分片方式（`row` = 按层切分，`auto` 自动）               |
