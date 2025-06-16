# vLLM 部署与使用

## 1. 配置环境

### 1.1 vLLM 0.9.0.1 版本从源码构建 CPU 版

1. 安装 Pytorch 2.6 + cpu
```bash
# CPU only
pip install torch==2.6.0 torchvision==0.21.0 torchaudio==2.6.0 --index-url https://download.pytorch.org/whl/cpu
# 检查安装是否成功
python -c "import torch; print(torch.__version__, torch.version.cuda, torch.cuda.is_available())"
```

2. 下载 vLLM 0.9.0.1 版本源码（最好是 git clone，需要```.git```目录），设置使用本地的pytorch
```bash
python use_existing_torch.py        
```

3. 下载依赖库
```bash
pip install -r requirements/build.txt requirements/common.txt requirements/cpu.txt
```

4. 修改文件

修改 ```pyproject.toml``` 文件中的
license = "Apache-2.0"
改为
license = { text = "Apache-2.0" }
删除 ```pyproject.tom```l 文件中的这一行
license-files = ["LICENSE"]

5. 下载安装依赖库 numa，并加入路径
```bash
export NUMACTL_DIR=$HOME/local/numactl
export C_INCLUDE_PATH=$NUMACTL_DIR/include:$C_INCLUDE_PATH
export LIBRARY_PATH=$NUMACTL_DIR/lib:$LIBRARY_PATH
export LD_LIBRARY_PATH=$NUMACTL_DIR/lib:$LD_LIBRARY_PATH
export CPATH=$NUMACTL_DIR/include:$CPATH
```

6. 编译vLLM
```bash
VLLM_TARGET_DEVICE=cpu python setup.py install
```

7. 使用vLLM
```bash
export VLLM_CPU_OMP_THREADS_BIND=0-15
export OMP_NUM_THREADS=16
export VLLM_CPU_KVCACHE_SPACE=40  # 单位是 MB
```

编写python脚本
```python
import os
from transformers import AutoTokenizer
from vllm import LLM, SamplingParams

def main():
    os.environ['VLLM_TARGET_DEVICE'] = 'cpu'
    os.environ['VLLM_USE_LEGACY_ENGINE'] = 'true'

    model_dir = "/home2/ghy/.cache/modelscope/hub/models/LLM-Research/gemma-3-12b-it"

    tokenizer = AutoTokenizer.from_pretrained(
        model_dir,
        local_files_only=True,
        trust_remote_code=True,
    )

    messages = [
        {"role": "system", "content": "You are a helpful assistant."},
        {"role": "user", "content": "天空为什么是蓝色的？"}
    ]

    prompt = tokenizer.apply_chat_template(
        messages,
        tokenize=False,
        add_generation_prompt=True
    )

    llm = LLM(
        model=model_dir,
        tensor_parallel_size=1,
        dtype="bfloat16",  # CPU 必须用 float32
        trust_remote_code=True,
        # model_impl='transformers',
        device="cpu",
        max_model_len=2048,
    )

    sampling_params = SamplingParams(
        temperature=0.7,
        top_p=0.8,
        repetition_penalty=1.05,
        max_tokens=512,
    )

    outputs = llm.generate([prompt], sampling_params, stream=True)

    print("\n=== Prompt ===")
    print(prompt)
    print("\n=== Output ===")

    for request_output in outputs:
        # 每次追加的是新的 tokens（增量）
        print(request_output.outputs[0].text, end='', flush=True)


# 👇 关键：必须加这句
if __name__ == "__main__":
    main()

```


### 1.2 vLLM 0.9.0.1 版本从源码构建 GPU + CUDA-11.8 版

1. 先安装 CUDA 11.8
```bash
cd ~/local/src
wget https://developer.download.nvidia.com/compute/cuda/11.8.0/local_installers/cuda_11.8.0_520.61.05_linux.run
sh cuda_11.8.0_520.61.05_linux.run --toolkit --silent --toolkitpath=$HOME/local/cuda-11.8
# 检查安装是否成功
nvcc --version
```
修改环境配置文件，加入cuda的路径

2. 安装 Pytorch 2.6 + cu118
```bash
pip3 install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu118
# 检查安装是否成功
python -c "import torch; print(torch.__version__, torch.version.cuda, torch.cuda.is_available())"
```

3. 下载 vLLM 0.9.0.1 版本源码（最好是 git clone，需要```.git```目录），设置使用本地的pytorch
```bash
python use_existing_torch.py        
```

4. 下载依赖库
```bash
pip install -r requirements/build.txt requirements/common.txt
```

5. 修改文件
修改 ```pyproject.toml``` 文件中的
license = "Apache-2.0"
改为
license = { text = "Apache-2.0" }
删除 ```pyproject.tom```l 文件中的这一行
license-files = ["LICENSE"]

6. 手动下载两个github包，并修改cmake文件
下载cutlass与FlashMLA，解压至 ```vllm-0.9.0.1/vllm/third_party/``` 目录下
手动修改文件 ```vllm-0.9.0.1/cmake/external_projects/flashmla.cmake```:
```GIT_REPOSITORY https://github.com/vllm-project/FlashMLA.git```
改为
```SOURCE_DIR ${CMAKE_SOURCE_DIR}/vllm/third_party/FlashMLA```
同理修改 ```vllm-0.9.0.1/CMakeLists.txt``` 文件

7. 编译vLLM
```bash
# VLLM_TARGET_DEVICE=cpu python setup.py install
pip install --no-build-isolation -e .
```

8. 使用vLLM
```bash
pip install xformers==0.0.25
```

编写python脚本

```python
from vllm import LLM, SamplingParams

llm = LLM(model="../gemma-3-12b-it-Q4_K_M.gguf")  # 替换为你的模型路径

prompt = "请简要介绍一下相对论的基本思想。"
sampling_params = SamplingParams(max_tokens=128, temperature=0.7)

outputs = llm.generate(prompt, sampling_params)
print(outputs[0].outputs[0].text)
```

> 由于测试的模型不支持 FP16 机械能推理，且V100不支持 BF16，所以没输出

## 2. vLLM 部署模型(GPU)

首先需要从 ModelScope 下载模型文件
```bash
modelscope download --model Qwen/Qwen2.5-VL-7B-Instruct --local_dir ./qwen2_5
modelscope download --model LLM-Research/gemma-3-12b-it --local_dir ./gemma-3
```

### 2.1 vLLM-Server端
```bash
# 强制只使用 GPU1
export CUDA_VISIBLE_DEVICES=0,1
# 使用 ModelScope 的模型
export VLLM_USE_MODELSCOPE=True
# 启用 “可扩展段式” 显存分配策略
export PYTORCH_CUDA_ALLOC_CONF=expandable_segments:True

python -m vllm.entrypoints.openai.api_server \
  --model /home2/ghy/.cache/modelscope/hub/models/LLM-Research/gemma-3-12b-it-fp32 \
  --tokenizer /home2/ghy/.cache/modelscope/hub/models/LLM-Research/gemma-3-12b-it-fp32 \
  --dtype float32 \
  --port 8000 \
  --trust-remote-code \
  --served-model-name gemma-3-12b-it \
  --gpu-memory-utilization 0.95 \
  --max-model-len 8192 \
  --max-num-seqs 1 \
  --enforce-eager

```
--chat-template /home2/ghy/.cache/modelscope/hub/models/LLM-Research/gemma-3-12b-it/chat_template_fixed.json \
```bash
--chat-template # 指定对话模板
--gpu-memory-utilization # 指定 GPU 显存的最大占用率
--max-num-seqs # 指定并发数量
--mm-embed-counts image:256 # 指定图像enbedding后的大小
--enforce-eager # 确保模型加载后立即执行初始化
```

### 2.2 vLLM-Client端
```bash
curl http://localhost:8000/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gemma-3-12b-it",
    "messages": [
      {
        "role": "system",
        "content": [{"type": "text", "text": "你是一个乐于助人的数学助手"}]
      },
      {
        "role": "user",
        "content": [{"type": "text", "text": "请简单介绍一下三角函数"}]
      }
    ],
    "temperature": 0.7,
    "max_tokens": 512
  }'


```

### 2.3 图片访问端
```bash
python3 -m http.server 8008
```

## 3. vLLM 部署量化模型（CPU）

```bash
export VLLM_CPU_OMP_THREADS_BIND=0-23
export OMP_NUM_THREADS=24
export VLLM_CPU_KVCACHE_SPACE=40  # 单位是 MB

export VLLM_USE_LEGACY_ENGINE=true
export VLLM_USE_V1=0

LD_PRELOAD=/usr/local/gcc-13.2.0/lib64/libstdc++.so.6 VLLM_USE_LEGACY_ENGINE=true python gemma_server.py

# bf16
LD_PRELOAD=/usr/local/gcc-13.2.0/lib64/libstdc++.so.6 VLLM_USE_LEGACY_ENGINE=true python -m vllm.entrypoints.openai.api_server \
  --model /home2/ghy/.cache/modelscope/hub/models/LLM-Research/gemma-3-12b-it \
  --tokenizer /home2/ghy/.cache/modelscope/hub/models/LLM-Research/gemma-3-12b-it \
  --dtype bfloat16 \
  --device cpu \
  --port 8000 \
  --trust-remote-code \
  --served-model-name gemma-3-12b-it \
  --max-model-len 2048 \
  --max-num-seqs 1 \
  --enforce-eager

# w4a16
LD_PRELOAD=/usr/local/gcc-13.2.0/lib64/libstdc++.so.6 VLLM_USE_LEGACY_ENGINE=true python -m vllm.entrypoints.openai.api_server \
  --model /data/home2/ghy/.cache/modelscope/hub/models/nm-testing/gemma-3-12b-it-quantized___w4a16 \
  --tokenizer /data/home2/ghy/.cache/modelscope/hub/models/nm-testing/gemma-3-12b-it-quantized___w4a16 \
  --dtype bfloat16 \
  --device cpu \
  --port 8000 \
  --trust-remote-code \
  --served-model-name gemma-3-12b-it \
  --max-model-len 2048 \
  --max-num-seqs 1 \
  --enforce-eager

# w8a8
LD_PRELOAD=/usr/local/gcc-13.2.0/lib64/libstdc++.so.6 VLLM_USE_LEGACY_ENGINE=true python -m vllm.entrypoints.openai.api_server \
  --model /data/home2/ghy/.cache/modelscope/hub/models/nm-testing/gemma-3-12b-it-quantized___w8a8 \
  --tokenizer /data/home2/ghy/.cache/modelscope/hub/models/nm-testing/gemma-3-12b-it-quantized___w8a8 \
  --dtype bfloat16 \
  --device cpu \
  --port 8000 \
  --trust-remote-code \
  --served-model-name gemma-3-12b-it \
  --max-model-len 2048 \
  --max-num-seqs 1 \
  --load-format bitsandbytes \
  --quantization bitsandbytes \
  --enforce-eager

# bnb
LD_PRELOAD=/usr/local/gcc-13.2.0/lib64/libstdc++.so.6 VLLM_USE_LEGACY_ENGINE=true python -m vllm.entrypoints.openai.api_server \
  --model /data/home2/ghy/.cache/modelscope/hub/models/AIDC-AI/Ovis1___6-Gemma2-9B-GPTQ-Int4/ \
  --tokenizer /data/home2/ghy/.cache/modelscope/hub/models/AIDC-AI/Ovis1___6-Gemma2-9B-GPTQ-Int4/ \
  --dtype bfloat16 \
  --device cpu \
  --port 8000 \
  --trust-remote-code \
  --served-model-name gemma-3-12b-it \
  --max-model-len 2048 \
  --max-num-seqs 1 \
  --quantization gptq \
  --add_bos_token=True \
  --enforce-eager
```

## 3. 备份 conda 环境
```conda
conda env export > ~/vllm_cpu_env_full.yml
conda env create -n vllm_cpu_copy -f vllm_cpu_env_full.yml
```

