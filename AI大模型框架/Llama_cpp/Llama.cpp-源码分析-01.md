# Llama.cpp 源码分析-01 GGML后端与设备注册流程

## 1. ggml_backend_registry 结构体
**后端注册器（backend registry）**
用于统一管理所有计算后端（如 CPU、GPU、OpenCL、CUDA、BLAS 等）以及它们所包含的设备
一个“调度中心”，在程序启动时注册各种后端，并收集这些后端提供的“计算设备”，供系统后续查询使用（例如查找 GPU、是否支持加速等） 

### 1.1 核心成员：
- backends - 注册的后端列表
- devices - 每个后端对应的设备列表
```cpp
std::vector<ggml_backend_reg_entry> backends;   // 存储已注册的后端，每个后端可能包含多个设备
std::vector<ggml_backend_dev_t>     devices;    // 所有已注册的设备（从后端提取）
```

### 1.2 构造函数 **```ggml_backend_registry()```**
- 注册所有编译支持的后端
- 自动根据编译宏注册所有已启用的后端
- 每次注册一个后端，会调用该后端对应的注册函数，如 ggml_backend_cuda_reg()，返回一个后端注册结构体。

```cpp
ggml_backend_registry() {
#ifdef GGML_USE_CANN
        register_backend(ggml_backend_cann_reg());
#endif
#ifdef GGML_USE_CPU
        register_backend(ggml_backend_cpu_reg());
#endif
}
```

### 1.3 注册后端 ```register_backend()```
后端注册到系统devices列表函数 
1. 将后端注册信息push添加到 backends 列表中
2. 遍历后端注册的所有设备，并将它们注册到 devices 列表中

```cpp
/**
 * @brief 这个函数用于将某个后端注册到系统devices列表中，并将其所有子设备也一并注册。
 * 
 * @param reg 传入一个后端注册结构体（ggml_backend_reg_t），它包含了后端的接口函数和设备列表。
 * @param handle 传入一个动态链接库句柄（dl_handle_ptr），如果后端是从动态库加载的，可以传入该句柄。
 */
void register_backend(ggml_backend_reg_t reg, dl_handle_ptrhandle = nullptr)
```

### 1.4 注册设备 ```register_device()```
后端注册到系统devices列表函数 
将传入的设备push存入devices列表中
```cpp
// @brief 将一个设备注册到系统devices列表中
void register_device(ggml_backend_dev_t device)
```

### 1.5 加载后端 ```load_backend()```
从磁盘加载一个共享库（动态链接库），验证其合法性与兼容性，然后初始化该后端并将其注册进系统。
```cpp
/**
 * @brief 它会从磁盘加载一个共享库（动态链接库），验证其合法性与兼容性，然后初始化该后端并将其注册进系统。
 * 
 * @param path 动态库路径
 * @param silent 是否静默加载（不输出错误信息）
 * @return ggml_backend_reg_t 返回一个从动态库中加载的后端注册结构体（ggml_backend_reg_t）
 */
ggml_backend_reg_t load_backend(const fs::path & path, bool silent)
```

### 1.6 卸载后端 ```unload_backend()```
卸载传入的后端，同时清除对应的设备列表