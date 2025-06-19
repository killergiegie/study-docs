# Llama.cpp 源码分析

## 1. 启动分析

### 1.1 后端注册流程 ggml_backend_registry 结构体
两个核心成员：
- 注册的后端列表
- 每个后端对应的设备列表
```cpp
std::vector<ggml_backend_reg_entry> backends;   // 存储已注册的后端，每个后端可能包含多个设备
std::vector<ggml_backend_dev_t>     devices;    // 所有已注册的设备（从后端提取）
```

构造函数 ```ggml_backend_registry()```
根据编译宏注册所有已启用的后端，执行对应注册操作```register_backend(ggml_backend_cann_reg());```