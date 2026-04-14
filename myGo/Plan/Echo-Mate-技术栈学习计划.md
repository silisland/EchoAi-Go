# Echo-Mate AI Desktop Robot - 技术栈学习计划

**项目**: Echo-Mate AI桌面机器人  
**目标平台**: RV1106 (Cortex A7 + 1TOPS NPU)  
**框架**: LVGL v9.2 + CMake + Buildroot Linux  
**文档生成时间**: 2026-04-14  

---

## 📋 学习路径概览

本学习计划按照 **8个阶段** 组织，从基础到高级逐步推进，预计总学习周期 **16周**。

```
Phase 1 (1-2周) → C/C++基础 + LVGL图形框架
Phase 2 (3-4周) → CMake交叉编译 + 嵌入式Linux
Phase 3 (5-6周) → 音频处理 (PortAudio/Opus/ALSA)
Phase 4 (7-8周) → WebSocket通信协议
Phase 5 (9-10周) → AI服务端 (Python + FunASR + CosyVoice)
Phase 6 (11-12周) → YOLOv5 + RKNN目标检测
Phase 7 (13-14周) → 云服务集成 (阿里云/高德地图)
Phase 8 (15-16周) → 系统集成与测试
```

---

## 🔧 1. 硬件平台

### RV1106 (Rockchip 视觉处理器)

| 规格 | 参数 |
|------|------|
| **CPU** | 单核 ARM Cortex-A7 32位，支持NEON和FPU |
| **NPU** | 0.5-1 TOPS 算力 (INT4/INT8/INT16) |
| **ISP** | 第三代图像信号处理器，支持最高5M@30fps |
| **视频编码** | H.264/H.265 5M@30fps |
| **内存** | 128-256MB DDR3L 嵌入式 |
| **以太网** | 内置100M PHY |

**文档资源**:
- [Rockchip RV1106 数据手册](https://datasheet4u.com/datasheet-pdf/Rockchip/RV1106/pdf.php?id=1543288)

**难度**: ⭐⭐⭐⭐ 高级 (需要嵌入式系统知识)  
**前置要求**: ARM架构、Linux内核、嵌入式开发基础

**学习重点**:
- ARM Cortex-A7 架构特性
- NPU 推理加速原理
- 内存受限环境下的优化
- 外设接口配置 (GPIO/I2C/SPI/UART)

---

## 💻 2. 软件栈

### 2.1 LVGL v9.2 (GUI框架)

| 特性 | 说明 |
|------|------|
| **类型** | 嵌入式图形库 |
| **语言** | C (兼容C++) |
| **许可证** | MIT |
| **内存占用** | 低功耗 (~64KB RAM) |
| **特性** | 组件、样式、动画、主题 |

**官方文档**:
- [LVGL v9.2 文档](https://docs.lvgl.io/9.2/intro/index.html)
- [LVGL GitHub](https://github.com/lvgl/lvgl)

**难度**: ⭐⭐⭐ 中级  
**前置要求**: C编程基础、显示驱动原理

**核心概念**:
1. **显示驱动** - 帧缓冲区、刷新回调
2. **输入设备** - 触摸屏、按键、编码器
3. **对象生命周期** - 创建、删除、父子关系
4. **样式系统** - 状态、过渡、继承
5. **事件处理** - 点击、滑动、长按

**学习资源**:
- [LVGL官方教程](https://docs.lvgl.io/9.2/overview/index.html)
- [LVGL示例代码](https://github.com/lvgl/lvgl/tree/master/examples)

---

### 2.2 CMake (构建系统)

| 用途 | 说明 |
|------|------|
| **目标** | ARM交叉编译 |
| **工具链** | arm-gnueabihf |
| **关键文件** | toolchain.cmake, CMakeLists.txt |

**官方文档**:
- [CMake交叉编译指南](https://cmake.org/cmake/help/book/mastering-cmake/chapter/Cross%20Compiling%20With%20CMake.html)

**难度**: ⭐⭐⭐ 中级  
**前置要求**: Makefile/构建系统基础

**核心概念**:
- 工具链文件配置
- `CMAKE_SYSTEM_NAME` 设置
- 交叉编译变量
- 第三方库链接

**示例配置**:
```cmake
set(CMAKE_SYSTEM_NAME Linux)
set(CMAKE_SYSTEM_PROCESSOR arm)
set(CMAKE_C_COMPILER arm-linux-gnueabihf-gcc)
set(CMAKE_CXX_COMPILER arm-linux-gnueabihf-g++)
```

---

### 2.3 Buildroot Linux

| 特性 | 说明 |
|------|------|
| **用途** | 嵌入式Linux根文件系统生成 |
| **版本** | 2026.02 |
| **功能** | 交叉编译工具链、内核、引导程序 |

**官方文档**:
- [Buildroot手册](https://buildroot.org/downloads/manual/manual.html)

**难度**: ⭐⭐⭐⭐ 高级  
**前置要求**: Linux系统、嵌入式构建经验

**学习重点**:
- Board Support Package (BSP)
- 根文件系统配置
- 内核裁剪
- 引导程序配置

---

## 📝 3. 编程语言

### 3.1 C/C++ (嵌入式客户端)

| 特性 | 说明 |
|------|------|
| **标准** | C++11/14 |
| **应用** | LVGL UI、WebSocket、音频I/O |
| **关键库** | WebSocket++, PortAudio, ALSA |

**难度**: ⭐⭐⭐⭐ 中高级  
**前置要求**: C编程、数据结构、嵌入式系统

**推荐学习路径**:
1. C语言基础 (指针、结构体、内存管理)
2. C++面向对象 (类、继承、多态)
3. C++11新特性 (智能指针、lambda、auto)
4. 嵌入式C++ (避免异常、RTTI)

**关键技能**:
- 裸机编程 vs RTOS
- 内存对齐与优化
- 编译器优化选项
- GDB调试技巧

---

### 3.2 Python 3.10+ (AI服务端)

| 特性 | 说明 |
|------|------|
| **用途** | AI模型管道 |
| **关键框架** | FunASR、PyTorch、FastAPI |

**官方文档**:
- [Python官方文档](https://docs.python.org/3.10/)

**难度**: ⭐⭐ 初级到中级  
**前置要求**: Python语法、异步编程

**推荐学习路径**:
1. Python基础语法
2. 异步编程 (async/await)
3. WebSocket服务器
4. AI模型推理 (PyTorch)

---

## 🎵 4. 音频处理

### 4.1 PortAudio (音频I/O)

| 特性 | 说明 |
|------|------|
| **用途** | 跨平台音频采集/播放 |
| **语言** | C/C++ |
| **许可证** | MIT |

**官方文档**:
- [PortAudio官网](https://www.portaudio.com/)
- [API参考](http://portaudio.com/docs/v19-doxydocs/index.html)

**难度**: ⭐⭐⭐ 中级  
**前置要求**: 音频基础概念

**核心概念**:
- 音频流处理
- 回调函数机制
- 延迟控制
- 采样率匹配

---

### 4.2 Opus 编解码器 (音频压缩)

| 特性 | 说明 |
|------|------|
| **用途** | 语音音频压缩 |
| **码率** | 6-510 kb/s |
| **采样率** | 8-48 kHz |
| **标准** | RFC 6716 |

**官方文档**:
- [Opus文档](https://opus-codec.org/docs/)
- [API参考](https://opus-codec.org/docs/opus_api-1.6.html)

**难度**: ⭐⭐⭐ 中级  
**前置要求**: 音频编解码基础

**核心概念**:
- 编码/解码API
- 码率控制
- 复杂度设置
- 语音优化模式

**代码示例**:
```c
// Opus编码器创建
OpusEncoder *enc = opus_encoder_create(
    16000,  // 采样率
    1,      // 单声道
    OPUS_APPLICATION_VOIP,  // 语音优化
    &error
);
```

---

### 4.3 ALSA (Linux音频)

| 特性 | 说明 |
|------|------|
| **用途** | Linux声音系统 |
| **组件** | alsa-lib, 内核驱动 |
| **版本** | 1.2.15.3 |

**官方文档**:
- [ALSA API文档](https://www.alsa-project.org/alsa-doc/alsa-lib/index.html)

**难度**: ⭐⭐⭐⭐ 高级  
**前置要求**: Linux内核、音频驱动

**核心概念**:
- PCM接口
- 混音器控制
- 插件系统
- 设备配置

---

## 🤖 5. AI/ML 技术栈

### 5.1 SenseVoice (语音识别 ASR)

| 特性 | 说明 |
|------|------|
| **类型** | 多语言语音识别 |
| **支持语言** | 50+种语言 |
| **训练数据** | 400,000+ 小时 |
| **功能** | ASR、语种识别、情感识别、音频事件检测 |
| **仓库** | [FunAudioLLM/SenseVoice](https://github.com/FunAudioLLM/SenseVoice) |

**难度**: ⭐⭐⭐ 中级  
**前置要求**: 语音处理、FunASR框架

**核心概念**:
- 非自回归推理
- 流式识别
- 多语言模型

**学习资源**:
- [SenseVoice论文](https://arxiv.org/abs/2407.04051)
- [FunASR文档](https://github.com/alibaba-damo-academy/funasr)

---

### 5.2 FSMN-VAD (语音活动检测)

| 特性 | 说明 |
|------|------|
| **类型** | 语音活动检测 |
| **框架** | FunASR |
| **训练数据** | 5,000小时中英文 |
| **模型大小** | 0.4M 参数 |

**GitHub**:
- [modelscope/FunASR](https://github.com/alibaba-damo-academy/funasr)

**难度**: ⭐⭐⭐ 中级  
**前置要求**: 音频信号处理

**核心概念**:
- 语音片段检测
- 噪声抑制
- 端点检测算法

---

### 5.3 CosyVoice (语音合成 TTS)

| 特性 | 说明 |
|------|------|
| **类型** | 文本转语音 |
| **支持语言** | 9种语言 |
| **特性** | 声音克隆、音色控制 |
| **仓库** | [FunAudioLLM/CosyVoice](https://github.com/FunAudioLLM/CosyVoice) |

**官方文档**:
- [CosyVoice 3.0](https://funaudiollm.github.io/cosyvoice3)
- [阿里云CosyVoice API](https://www.alibabacloud.com/help/en/model-studio/text-to-speech)

**难度**: ⭐⭐⭐⭐ 高级  
**前置要求**: 语音合成基础

**核心概念**:
- 零样本TTS
- 声音克隆
- 音色迁移

---

### 5.4 通义千问 (大语言模型)

| 特性 | 说明 |
|------|------|
| **提供商** | 阿里云模型服务平台 |
| **API** | DashScope |
| **模型** | Qwen系列 (qwen-plus, qwen-flash等) |

**官方文档**:
- [DashScope API文档](https://www.alibabacloud.com/help/en/model-studio/qwen-api-via-dashscope)

**难度**: ⭐⭐ 初级  
**前置要求**: API集成基础

**核心概念**:
- Chat Completions API
- 流式响应
- Function Calling
- 系统提示词

---

### 5.5 YOLOv5 + RKNN (目标检测)

| 特性 | 说明 |
|------|------|
| **类型** | 目标检测 |
| **NPU加速** | Rockchip RKNN |
| **框架** | rknn-toolkit2 |

**资源**:
- [Rockchip YOLOv5](https://github.com/airockchip/yolov5)
- [RKNN Model Zoo](https://github.com/airockchip/rknn_model_zoo)

**难度**: ⭐⭐⭐⭐⭐ 专家级  
**前置要求**: 深度学习、嵌入式优化

**核心概念**:
- 模型转换 (ONNX→RKNN)
- 量化 (FP16/INT8)
- NPU推理优化
- 后处理 (NMS)

**学习路径**:
1. YOLOv5基础
2. 模型导出ONNX
3. RKNN模型转换
4. NPU推理部署

---

## 🌐 6. 通信协议

### 6.1 WebSocket 协议

| 特性 | 说明 |
|------|------|
| **标准** | RFC6455 |
| **用途** | 实时双向通信 |
| **数据格式** | JSON + 二进制 (Opus音频) |

**难度**: ⭐⭐ 初级  
**前置要求**: HTTP基础

**核心概念**:
- 握手过程
- 帧格式
- 掩码机制
- 心跳保活

---

### 6.2 WebSocket++ (C++库)

| 特性 | 说明 |
|------|------|
| **类型** | 头文件-only C++库 |
| **版本** | 0.8.2 |
| **依赖** | C++11 或 Boost |
| **传输层** | Asio/iostream/raw |

**官方文档**:
- [WebSocket++文档](https://docs.websocketpp.org/)
- [GitHub仓库](https://github.com/zaphoyd/websocketpp)

**难度**: ⭐⭐⭐ 中级  
**前置要求**: C++、网络编程

**核心概念**:
- Endpoint端点
- Connection连接
- Handler处理器
- 消息类型 (text/binary)

**代码示例**:
```cpp
// WebSocket客户端连接
websocketpp::client<websocketpp::config::asio_client> client;
client.init_asio();
client.set_message_handler(&on_message);
websocketpp::lib::error_code ec;
client.connect("ws://172.32.0.100:8000", ec);
```

---

### 6.3 JSON 消息格式

| 特性 | 说明 |
|------|------|
| **库** | cJSON 或类似 |
| **用途** | 协议消息格式 |

**难度**: ⭐⭐ 初级  
**前置要求**: 数据结构基础

**Echo-Mate协议示例**:
```json
{
  "type": "chat",
  "text": "你好，今天天气怎么样？",
  "session_id": "abc123"
}
```

---

## ☁️ 7. 云服务

### 7.1 阿里云 (DashScope)

| 特性 | 说明 |
|------|------|
| **服务** | CosyVoice、Qwen、ASR |
| **API端点** | dashscope.aliyuncs.com |
| **地域** | 新加坡、北京、香港 |

**官方文档**:
- [模型服务平台](https://www.alibabacloud.com/help/en/model-studio/)
- [API密钥管理](https://dashscope.console.aliyun.com/)

**难度**: ⭐⭐ 初级  
**前置要求**: REST API基础

**核心概念**:
- API认证 (Bearer Token)
- 流式响应处理
- 速率限制管理

---

### 7.2 高德地图 (天气API)

| 特性 | 说明 |
|------|------|
| **用途** | 天气数据 |
| **API** | restapi.amap.com/v3/weather/weatherInfo |
| **数据** | 实况天气 + 预报 |

**官方文档**:
- [天气API文档](https://lbs-dev.amap.com/api/webservice/guide/api/weatherinfo)
- [城市编码参考](https://lbs.amap.com/api/webservice/guide/newcity)

**难度**: ⭐⭐ 初级  
**前置要求**: HTTP GET请求

---

## 🛠️ 8. 开发工具

### 8.1 交叉编译工具链

| 组件 | 用途 |
|------|------|
| ARM编译器 | arm-linux-gnueabihf-gcc |
| GDB | 调试 |
| CMake | 构建配置 |

**安装**:
```bash
# Ubuntu/Debian
sudo apt-get install gcc-arm-linux-gnueabihf gdb-multiarch cmake
```

---

### 8.2 Git 子模块

| 模块 | 用途 |
|------|------|
| SenseVoice | ASR模型 |
| FSMN-VAD | 语音活动检测 |
| LVGL | GUI框架 |
| rkmpi_demos | Rockchip媒体示例 |

**命令**:
```bash
# 初始化子模块
git submodule update --init --recursive

# 更新子模块
git submodule update --remote
```

---

### 8.3 NTP 时间同步

| 特性 | 说明 |
|------|------|
| **用途** | 网络时间同步 |
| **服务** | 阿里云NTP服务器 |

---

## 📚 推荐学习资源

### 在线课程

1. **嵌入式Linux**
   - [Buildroot官方文档](https://buildroot.org/downloads/manual/manual.html)
   - [LVGL官方教程](https://docs.lvgl.io/9.2/)

2. **AI/语音**
   - [FunASR教程](https://github.com/alibaba-damo-academy/funasr)
   - [CosyVoice论文](https://arxiv.org/abs/2407.04051)

3. **WebSocket**
   - [WebSocket++示例](https://github.com/zaphoyd/websocketpp/tree/master/examples)

### 书籍推荐

1. 《嵌入式C语言自我修养》
2. 《Linux设备驱动开发详解》
3. 《CMake实践》
4. 《深度学习入门》

---

## ✅ 学习检查清单

### Phase 1: 基础 (1-2周)
- [ ] C语言指针与内存管理
- [ ] C++类与继承
- [ ] LVGL基础组件使用
- [ ] LVGL事件处理

### Phase 2: 构建系统 (3-4周)
- [ ] CMake基础语法
- [ ] 交叉编译配置
- [ ] Buildroot根文件系统
- [ ] 烧录与启动

### Phase 3: 音频 (5-6周)
- [ ] PortAudio采集播放
- [ ] Opus编解码
- [ ] ALSA配置
- [ ] 音频pipeline设计

### Phase 4: 通信 (7-8周)
- [ ] WebSocket协议理解
- [ ] WebSocket++客户端实现
- [ ] JSON消息设计
- [ ] 二进制音频流传输

### Phase 5: AI服务端 (9-10周)
- [ ] Python异步编程
- [ ] SenseVoice ASR部署
- [ ] CosyVoice TTS集成
- [ ] 通义千问API调用
- [ ] WebSocket服务器实现

### Phase 6: 目标检测 (11-12周)
- [ ] YOLOv5模型训练
- [ ] ONNX导出
- [ ] RKNN模型转换
- [ ] NPU推理优化

### Phase 7: 云服务 (13-14周)
- [ ] 阿里云API认证
- [ ] DashScope流式API
- [ ] 高德天气API
- [ ] API错误处理

### Phase 8: 集成 (15-16周)
- [ ] 模块联调
- [ ] 系统优化
- [ ] 压力测试
- [ ] 文档完善

---

## 🔗 相关文档

- [项目根目录AGENTS.md](/home/GreedArch/project/echoAI/AGENTS.md)
- [DeepWiki文档](/home/GreedArch/project/echoAI/deepwiki/)
- [硬件BOM清单](/home/GreedArch/project/echoAI/assets/元器件清单.md)

---

*文档最后更新: 2026-04-14*  
*作者: AI学习助手*