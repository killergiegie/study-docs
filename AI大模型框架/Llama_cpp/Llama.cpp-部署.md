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

### 3.1 Gemma-3-12b-it 模型

#### 3.1.0 Gemma-3-12b-it 量化模型 + CPU 推理

```bash
/bin/time ~/LLM/llama.cpp-cpu-new/build/bin/llama-mtmd-cli \
  -m ./gemma-3-GGUF/google_gemma-3-12b-it-Q3_K_L.gguf \
  --mmproj ./gemma-3-GGUF/mmproj-google_gemma-3-12b-it-f16.gguf \
  -p "回答22题，首先复述出题目，部分题目可能不止一个解，有需要分类讨论的可能；遇到数学题时请逐步推理；所有数学表达式必须使用LaTeX语法，必须采用 \(\) 或 \[\] 包裹起来；输出内容采用三段式结构，依次为：“解题步骤”、“结论”与“知识点总结”，每段以固定标题开头，便于分段解析和结构化展示" \
  --image ./math_test/math.jpg \
  --temp 1.0 --top-k 40 --top-p 0.95 --presence-penalty 1.2 \
  -t 8 \
  -c 2048 \
  -n 2048 
```


#### 3.1.1 Gemma-3-12b-it-Q4_K_M 模型 + GPU 推理
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

#### 3.1.2 Gemma-3-12b-it-Q4_K_M 模型 + CPU 推理
使用8线程进行推理

```bash
/bin/time ~/LLM/llama.cpp-cpu/build/bin/llama-mtmd-cli \
  -m ./gemma-3-12b-it-Q4_K_M.gguf \
  --mmproj ./gemma-3-mmproj-model-f16.gguf \
  -p "回答二十二题，首先复述出题目，请详细推理并输出尽量多内容，部分题目可能不止一个解，有需要分类讨论的可能；遇到数学题时请逐步推理；所有数学表达式必须使用LaTeX语法，必须采用 \(\) 或 \[\] 包裹起来；输出内容采用三段式结构，依次为：“解题步骤”、“结论”与“知识点总结”，每段以固定标题开头，便于分段解析和结构化展示" \
  --image ./math_test/math.jpg \
  --temp 1.0 --top-k 40 --top-p 0.95 --presence-penalty 1.2 \
  -t 8 \
  -c 2048 \
  -n 2048 

/bin/time ~/LLM/llama.cpp-cpu/build/bin/llama-mtmd-cli \
  -m ./gemma-3-12b-it-Q4_K_M.gguf \
  --mmproj ./gemma-3-mmproj-model-f16.gguf \
  -p "回答二十二题，首先复述出题目，部分题目可能不止一个解，有需要分类讨论的可能；遇到数学题时请逐步推理；所有数学表达式必须使用LaTeX语法，必须采用 \(\) 或 \[\] 包裹起来；输出内容采用三段式结构，依次为：“解题步骤”、“结论”与“知识点总结”，每段以固定标题开头，便于分段解析和结构化展示" \
  --image ./math_test/math.jpg \
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



#### 3.1.3 Gemma-3-12b-it-bf16 模型 + CPU 推理
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

### 3.5 Qwen2.5-VL-7B-Q4_K_M 模型 + GPU 推理
仅使用 GPU1 进行推理
```bash
CUDA_VISIBLE_DEVICES=1 nice -n 10 ~/LLM/llama.cpp-master/build/bin/llama-mtmd-cli \
  -m ./Qwen2.5-VL-7B-Instruct-Q4_K_M.gguf \
  --mmproj ./Qwen2.5_mmproj-F16.gguf \
  -p "回答22题。首先复述出题目，请详细推理并输出尽量多内容；遇到数学题时请逐步推理；所有数学表达式必须使用标准 LaTeX 语法，采用 \( \) 或 \[ \] 包裹；输出内容采用三段式结构，依次为：“解题步骤”、“结论”与“知识点总结”，每段以固定标题开头，便于分段解析和结构化展示" \
  --image ./math_test/math.jpg \
  -ngl 99 -fa -sm row \
  --temp 0.8 --top-k 40 --top-p 0.95 --presence-penalty 1.2 \
  -t 24 \
  -c 8192 \
  -n 8192 

/bin/time ~/LLM/llama.cpp-cpu/build/bin/llama-mtmd-cli \
  -m ./Qwen2.5-VL-7B-Instruct-Q4_K_M.gguf \
  --mmproj ./Qwen2.5_mmproj-F16.gguf \
  -p "回答二十二题。首先复述出题目，请详细推理并输出尽量多内容；遇到数学题时请逐步推理；所有数学表达式必须使用标准 LaTeX 语法，采用 \( \) 或 \[ \] 包裹；输出内容采用三段式结构，依次为：“解题步骤”、“结论”与“知识点总结”，每段以固定标题开头，便于分段解析和结构化展示" \
  --image ./math_test/math.jpg \
  --temp 0.8 --top-k 40 --top-p 0.95 --presence-penalty 1.2 \
  -t 8 \
  -c 4096 \
  -n 4096 
```


### 3.6 bagel-ema.gguf 模型 + CPU 推理\

#### 3.6.1 bagel-ema-q3_k_m.gguf
使用8线程进行推理
```bash
/bin/time ~/LLM/llama.cpp-cpu/build-update/bin/llama-mtmd-cli \
  -m ./bagel/ema-q4_k_m.gguf \
  --mmproj ./bagel/pig_ae_fp32-f16.gguf \
  -p "回答这道题，首先复述出题目，部分题目可能不止一个解，有需要分类讨论的可能；遇到数学题时请逐步推理；所有数学表达式必须使用LaTeX语法，必须采用 \(\) 或 \[\] 包裹起来；输出内容采用三段式结构，依次为：“解题步骤”、“结论”与“知识点总结”，每段以固定标题开头，便于分段解析和结构化展示" \
  --image ./math_test/6.png \
  --temp 0.6 --top-k 40 --top-p 0.95 \
  -t 8 \
  -c 2048 \
  -n 2048 
```

### 3.7 nvidia_OpenReasoning-Nemotron-7B 模型

使用8线程进行推理
```bash
/bin/time ~/LLM/llama.cpp-cpu/build-update/bin/llama-cli \
  -m ./nemotron/nvidia_OpenReasoning-Nemotron-7B-Q4_K_M.gguf \
  --temp 0.8 --top-k 40 --top-p 0.95 \
  -t 8 \
  -c 4096 \
  -n 4096 
```
$ \text{已知 } \sin^2 \theta + \sin \theta = 1，\text{ 求 } \cos^2 \theta + \cos^6 \theta \text{ 之值。} $

### 3.8 gemma-3-R1984 模型

使用8线程进行推理
```bash
/bin/time ~/LLM/llama.cpp-cpu/build/bin/llama-mtmd-cli \
  -m ./gemma-3-GGUF/google_gemma-3-12b-it-Q4_K_M.gguf \
  --mmproj ./mmproj-google_gemma-3-12b-it-f16.gguf \
  -p "回答这道题，首先复述出题目，部分题目可能不止一个解，有需要分类讨论的可能；遇到数学题时请逐步推理；所有数学表达式必须使用LaTeX语法，必须采用 \(\) 或 \[\] 包裹起来；输出内容采用三段式结构，依次为：“解题步骤”、“结论”与“知识点总结”，每段以固定标题开头，便于分段解析和结构化展示" \
  --image ./math_test/6.png \
  --temp 1.0 --top-k 40 --top-p 0.95 --presence-penalty 1.2 \
  -t 8 \
  -c 2048 \
  -n 2048 
```

### 3.9 erax 模型
使用8线程进行推理
```bash
/bin/time ~/LLM/llama.cpp-cpu/build/bin/llama-mtmd-cli \
  -m ./erax/EraX-VL-2B-V1.5.Q4_K_M.gguf \
  --mmproj ./erax/EraX-VL-2B-V1.5.mmproj-fp16.gguf \
  -p "使用中文汉字复述题目，数学公式使用latex格式输出，不要其他任何内容。" \
  --image ./math_test/6.png \
  --temp 1.0 --top-k 40 --top-p 0.95 --presence-penalty 1.2 \
  -t 8 \
  -c 2048 \
  -n 2048 
```


### 3.10 llama-tts

```bash
/bin/time ~/LLM/llama.cpp-cpu/build/bin/llama-tts \
  -m ./outetts/OuteTTS-1.0-0.6B-FP16.gguf \
  -mv ./outetts/WavTokenizer-Large-75-F16.gguf \
  -p "I have an apple" \
  -t 16 \
  --tts-use-guide-tokens \
  --temp 0.4 --repeat-last-n 64 --repeat-penalty 1.1 \
  --top-k 40 --top-p 0.9 --min-p 0.05 \
  -o output_audio.wav

/bin/time ~/LLM/llama.cpp-cpu-new/build/bin/llama-tts \
  -m ./outetts/OuteTTS-0.2-500M-FP16.gguf \
  -mv ./outetts/mywavtokenizer.gguf \
  -p "hello hello hello hello hello hello" \
  -t 16 \
  --tts-use-guide-tokens \
  --temp 0.4 --repeat-last-n 64 --repeat-penalty 1.1 \
  --top-k 40 --top-p 0.9 --min-p 0.05 \
  -o output_audio.wav

/bin/time ~/LLM/llama.cpp-cpu-new/build/bin/llama-tts \
  -m ./outetts/OuteTTS-1.0-0.6B-FP16.gguf \
  -mv ./outetts/mywavtokenizer.gguf \
  -p "I have an apple" \
  -t 16 \
  --temp 0.4 --repeat-last-n 64 --repeat-penalty 1.1 \
  --top-k 40 --top-p 0.9  \
  -o output_audio.wav

```

### 3.11 ChatTTS

性能方面：
CPU 内存占用：602MB
TPU 内存占用：303MB(常驻) 371MB(合成时)
TPU Util最高：30% - 52%
Real-Time Factor(RTF):  2.26 （合成1s音频需要2.26s）

### 3.12 Qwen2.5-omni

```bash
/bin/time ~/LLM/llama.cpp-cpu-new/build/bin/llama-mtmd-cli \
 -m ./qwen-omni/Qwen2.5-Omni-3B-Q4_K_M.gguf \
 --mmproj ./qwen-omni/mmproj-Qwen2.5-Omni-3B-f16.gguf \
 -p "请你使用文本描述清楚这张图片的所有内容，无需进一步解答。" \
 --image ./math_test/gk-03.png \
 --temp 0.8 --top-k 40 --top-p 0.95 --presence-penalty 1.2 \
 -t 8 \
 -c 1024 \
 -n 1024 
```

高考-01
若z=(m^2+m-6)+(m-2)i为纯虚数,则实数m的值为()

高考-02
x+y-1>=0,x-y+3>=0,x<=2;z=x+2y+1的最大值为?

高考-03
设函数$f(x)=xe^x+a(1-e^x)+1.$
(1) 求函数$f(x)$ 的单调区间；
(2)若函数$f(x)$ 在$(0,+\infty)$零点，证明：$a>2.$




高中题5
函数 \( y = \lg (a^{2x} - (ab)^{x} - 2b^{2x} + 1) \)，\( a > 0, b > 0 \)，在 \( x \) 取何值时，\( y > 0 \) ?。

高中题6
已知：sin²θ + sin θ = 1，求cos²θ + cos⁶θ的值。


### 3.13 Gemma-3n

#### Gemma-3n-E4B-Q4_K_M

```bash
/bin/time ~/LLM/llama.cpp-cpu-new/build/bin/llama-cli \
 -m ./gemma-3n-GGUF/google_gemma-3n-E4B-it-Q4_K_M.gguf \
 -p "回答：已知：sin²θ + sin θ = 1，求cos²θ + cos⁶θ的值。要求：首先复述出题目，请详细推理并输出尽量多内容；所有数学表达式必须使用标准 LaTeX 语法。输出内容采用三段式结构，依次为：“解题步骤”、“结论”与“知识点总结”，每段以固定标题开头，便于分段解析和结构化展示" \
 --temp 1.0 --top-k 64 --top-p 0.95 \
 -t 8 \
 -c 2048 \
 -n 2048 
```

#### Gemma-3n-E2B-Q8_0

```bash
/bin/time ~/LLM/llama.cpp-cpu-new/build/bin/llama-cli \
 -m ./gemma-3n-GGUF/google_gemma-3n-E2B-it-Q8_0.gguf \
 -p "回答：现在有一片神奇的草地，里面的草每天都在匀速生长。如果放8头牛可以6天把草吃完，如果放6头牛可以10天吃完，那么，如果放四头牛的话，可以几天吃完呢？如果是3头牛，又会是几天能吃完呢？" \
 --temp 1.0 --top-k 64 --top-p 0.95 \
 -t 8 \
 -c 2048 \
 -n 2048 

/bin/time ~/LLM/llama.cpp-cpu-new/build/bin/llama-cli \
  -m ./gemma-3-GGUF/google_gemma-3-12b-it-Q3_K_L.gguf \
  -p "回答：已知 $f(x) = (\frac{x - 1}{x + 1})^2 (x \geq 1)$，$f^{-1}(x)$ 为 f(x) 的反函数，又 $g(x) = \frac{1}{f^{-1}(x)} + \sqrt{x} + 2$。求 $f^{-1}(x)$ 的定义域、单调区间和 g(x) 的最小值。要求：首先复述出题目，请详细推理并输出尽量多内容，注意可能会需要你分类讨论；所有数学表达式必须使用标准 LaTeX 语法。输出内容采用两段式结构，依次为：“解题步骤”与“结论”，每段以固定标题开头" \
  --temp 1.0 --top-k 40 --top-p 0.95 --presence-penalty 1.2 \
  -t 8 \
  -c 2048 \
  -n 2048 


```

#### Gemma-3n 模型效果测试

|  题目  |  E4B-Q4K-M  |  E2B-Q8-0  |  qwen 识图 |
|:------:|:-----------:|:----------:| :----: |
|    1    |    正确     |    错误    |  准确  |
|    2    |   证明题    |   证明题   |  准确  |
|    3    |    错误     |    错误    |  准确  |
|    4    |    错误     |    错误    |  准确  |
|    5    |    错误     |    错误    | 小错误 |
|    6    |    正确     |    正确    |  准确  |
|    7    |  答对一半   |    错误    |  准确  |
|    8    |  答对一半   |    错误    |  准确  |
|    9    |    错误     |    错误    |  准确  |
|   10    |    错误     |    错误    |  准确  |


500MB(CPU)+2236MB(TPU)

TTS+STT 3709 MB


$x \in (a, a + \frac{1}{a}) \cup (2a, \infty)$.  即 $a < x < a + \frac{1}{a}$ 或 $x > 2a$.

### 3.14 Qwen3
```bash

/bin/time ~/LLM/llama.cpp-cpu-new/build/bin/llama-cli \
 -m ./qwen3/Qwen3-4B-Q4_K_M.gguf \
 -p "回答：现在有一片神奇的草地，里面的草每天都在匀速生长。如果放8头牛可以6天把草吃完，如果放6头牛可以10天吃完，那么，如果放四头牛的话，可以几天吃完呢？如果是3头牛，又会是几天能吃完呢？/no_think" \
 --temp 1.0 --top-k 64 --top-p 0.95 \
 -t 8 \
 -c 2048 \
 -n 2048 
 
 ```