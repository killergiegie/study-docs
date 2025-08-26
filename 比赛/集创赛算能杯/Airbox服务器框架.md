# Airbox 服务器框架设计

## 1. Airbox 常用指令

```bash
sudo chown -R ghy:ghy ghy/

sudo chown -R lyh:lyh lyh/
```

## 2. Airbox 项目结构

your_project/
├── main.py            # 主控 + 状态机 + 启动线程
├── base_worker.py     # Worker 基类：提供 start/stop/通用接口
├── stt_worker.py      # STTWorker 类（Python）
├── tts_worker.py      # TTSWorker 类（Python）
└── llm_worker.py      # 可替换为 llama.cpp 或独立 Py 模块



主进程(python实现)<绑定1个CPU>：
- main线程：维护各个子进程
- tcp线程：tcp通信状态机
- rtsp/rtp线程：rtp音频传输状态机
- tcp心跳监控线程：断线重连
- 调度线程，负责各个进程之间的数据接受与分发

STT 子进程(python 实现)<绑定1个CPU>：
- 与主进程使用multiprocessing.Queue通信
- 输入 目标音频的地址字符串
- 输出 文本字符串

LLM 子进程(Qwen2.5+python实现/llama.cpp实现)<绑定4~5个CPU>：
- 与主进程使用stdin/out实现通信 subprocess.Popen + stdin/stdout
- 构造数据包，标明来自哪个会话（并发支持）
- 输入 提问文本字符串
- 输出 LLM回答字符串

TTS 子进程(python实现)<绑定1~2个CPU>:
- 与主进程使用multiprocessing.Queue通信
- 输入 待处理的文本字符串
- 输出 合成的音频字节流


## 3. Airbox 启动 Qwen2.5 测试

进入 python 环境：

```bash
source /data/home/lyh/test_llm/python_3_10/bin/activate
cd airbox_server/

```

进入工程目录：
```bash
cd /data/home/lyh/test_llm/LLM-TPU_0625/models/Qwen2_5_VL/python_demo
```

运行：
```bash
python3 tcp_llm_html_tts_rtsp_v4.py -m ~/Qwen2_5-VL-7B-Instruct-AWQ/qwen2.5-vl-7b-instruct-awq_w4bf16_seq8192_bm1684x_1dev_20250430_115515.bmodel -c /data/home/lyh/test_llm/LLM-TPU_0625/models/Qwen2_5_VL/config


/bin/time python3 llm_worker.py -m ~/Qwen2_5-VL-7B-Instruct-AWQ/qwen2.5-vl-7b-instruct-awq_w4bf16_seq8192_bm1684x_1dev_20250430_115515.bmodel -c /data/home/lyh/test_llm/LLM-TPU_0625/models/Qwen2_5_VL/config

sudo /data/home/gzc/kernel_dev/tpu_hw_reset.sh

```

等服务器加载完成后再运行客户端
```
bmwhisper ./audio/record_out.wav --model small --bmodel_dir ./bmodel/ --chip_mode soc --language {'Chinese'}
```


## qwen2.5-3b

```bash
/bin/time python qwen2_5_vl.py \
    -m /data/home/lyh/Qwen2_5-VL-7B-Instruct-AWQ/qwen2.5-vl-7b-instruct-awq_w4bf16_seq4096_bm1684x_1dev_20250805_163620.bmodel \
    -c ./configs/config.json \
    -t ./configs \
    -p ./configs \
    -d 0 \
    -vi "[{\"type\":\"image_url\",\"image_url\":{\"url\": \"./math_test/10.png\"}, \"max_side\":420}]" \
    -ll INFO

```
Qwen2.5-vl-3B-bmodel
TPU 内存：4025 MB(加载)
CPU 内存： 506 MB(加载) ->  593 MB(推理)

提示词：
请你使用文本描述清楚这张图片的所有内容，若有数学公式则使用Latex格式输出，尽量严谨，无需进一步解答。

## qwen2.5-7b

```bash
/bin/time ./qwen2.5-server \
  -m /data/home/lyh/Qwen2_5-VL-7B-Instruct-AWQ/qwen2.5-vl-3b-instruct-awq_w4bf16_seq1024_bm1684x_1dev_20250812_002505.bmodel \
  -c /data/home/lyh/airbox_server/models/qwen2_5-vl/qwen2.5-7b-vl/config \
  -d 0 
```

TPU 内存： 7822 MB (加载)

现在有一片神奇的草地，里面的草每天都在匀速生长。如果放8头牛可以6天把草吃完，如果放6头牛可以10天吃完，那么，如果放四头牛的话，可以几天吃完呢？如果是3头牛，又会是几天能吃完呢？

## qwen3-4b

```bash
/bin/time ./qwen3-server \
  -m /data/home/lyh/Qwen2_5-VL-7B-Instruct-AWQ/qwen3-4b_w4bf16_seq2048_bm1684x_1dev_20250811_234428.bmodel \
  -c /data/home/gzc/models/bmodel/qwen3/config \
  -e \
  -d 0 
  
```



TPU 内存： 2917 MB (加载)

现在有一片神奇的草地，里面的草每天都在匀速生长。如果放8头牛可以6天把草吃完，如果放6头牛可以10天吃完，那么，如果放四头牛的话，可以几天吃完呢？如果是3头牛，又会是几天能吃完呢？


## gemma-3-12b

```bash
/bin/time python3 pipeline.py \
  -m /data/home/gzc/models/gemma3/gemma-3-12b-it_hf_w4bf16_seq2048_bm1684x_1dev_20250804_141717.bmodel \
  -c ./config \
  -d 0 

$media_path: /data/home/lyh/airbox_server/models/math_test/6.png

```

请回答图片中的这道题，要求：首先复述出题目，请详细推理并输出尽量多内容；所有数学表达式必须使用标准 LaTeX 语法。输出内容采用两段式结构，依次为：“解题步骤”与“结论”，每段以固定标题开头。

Gemma-3-12b bmodel
TPU 内存：10684 MB (模型加载)
CPU 内存：767 MB (模型加载) -> (推理)
推理速度：

## 151服务器 TPU-MLIR Docker

```bash
# 1、 进入docker
docker exec -u ghy -it sophon_tpu_llm bash
# 2、 激活编译环境
source /workspace/LLM-TPU/models/InternVL3/compile/internvl3/bin/activate
source /workspace/tpu-mlir/envsetup.sh
# 3、 编译
cd /model_test/bmodel  
#（model_test目录为host的共享目录，放在里面可以省去拷贝步骤）
llm_convert.py \
  -m /workspace/models/Qwen2.5-VL-7B-Instruct-AWQ \ 
  -s 4096 \ 
  --quantize w4bf16 \ 
  -c bm1684x \
  --out_dir qwen2.5-vl-7b \
  --max_pixels 640,480 \
  --use_block_with_kv \
  --max_input_length 512 \
  --dynamic    

llm_convert.py -m /workspace/models/Qwen2.5-VL-7B-Instruct-AWQ -s 4096 --quantize w4bf16 -c bm1684x --out_dir qwen2.5-vl-7b --max_pixels 644,504 --use_block_with_kv --max_input_length 512 --dynamic  

```

其中，/workspace/models/Qwen2.5-VL-7B-Instruct-AWQ是safetensor模型的路径，  qwen2.5-vl-7b是输出目录的路径


模块化服务架构

四个大模型并行运行

多模态协同（识图 + 推理 + STT + TTS）

后台进程管理（FIFO + JSON 指令分发）