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


```

等服务器加载完成后再运行客户端
```
bmwhisper ./audio/record_out.wav --model small --bmodel_dir ./bmodel/ --chip_mode soc --language {'Chinese'}
```