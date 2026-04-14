# WSL2 开发环境搭建与无硬件学习指南

**项目**: Echo-Mate AI Desktop Robot  
**适用场景**: 暂无 RV1106 开发板，使用 WSL2 进行预研和学习  
**目标**: 在获得硬件前完成 **65-75%** 的软件开发准备工作（软件部分可基本完成，硬件相关部分需预留适配时间）  
**预计周期**: 8-12 周（无硬件阶段）

---

## 📋 目录

1. [环境概览](#环境概览)
2. [WSL2 系统配置](#wsl2-系统配置)
3. [开发工具链安装](#开发工具链安装)
4. [分阶段学习计划](#分阶段学习计划)
5. [Simulator 模式开发](#simulator-模式开发)
6. [AI 服务端开发](#ai-服务端开发)
7. [代码预研与架构设计](#代码预研与架构设计)
8. [硬件准备清单](#硬件准备清单)
9. [从 Simulator 到真板的迁移](#从-simulator-到真板的迁移)

---

## 环境概览

### 无硬件时可完成的工作

| 模块 | 可完成度 | 说明 |
|------|---------|------|
| **AI 服务端** | 100% | Python + 阿里云 API，可完整开发测试 |
| **LVGL GUI 设计** | 90% | SDL Simulator 可运行大部分 UI |
| **WebSocket 通信** | 100% | PC 端可完整调试 |
| **音频处理逻辑** | 70% | 算法可验证，硬件相关部分需适配 |
| **YOLOv5 模型** | 80% | PC 端可训练、转换、验证 |
| **RKNN 部署** | ❌ 0% | 必须有 RV1106 硬件，无法模拟 |
| **硬件驱动** | ❌ 0% | GPIO/I2C/SPI 等需真板，无法模拟 |

### ❌ 无硬件时**无法完成**的工作（需标注）

| 功能 | 原因 | 现实替代方案 | 预计真机工作量 |
|------|------|-------------|---------------|
| **RKNN 模型推理** | 需要 NPU 硬件加速，无法在 PC 模拟 | ① 准备转换脚本 ② 用 PyTorch CPU 推理验证算法 | 3-5 天 |
| **MIPI 摄像头驱动** | 需要硬件接口和驱动 | ① 用图片/视频文件模拟数据流 ② 提前阅读驱动代码 | 2-3 天 |
| **GPIO 控制** | 硬件引脚操作 | ① Mock 函数模拟 GPIO 状态 ② 用配置文件模拟输入 | 1-2 天 |
| **I2C/SPI 设备通信** | 需要硬件总线 | ① 虚拟数据模拟返回值 ② 提前了解设备寄存器手册 | 2-3 天 |
| **ALSA 音频硬件** | 需要声卡驱动和硬件 | ① PortAudio 模拟 ② 文件输入输出测试 ③ 注意：延迟和缓冲区大小差异需真机调整 | 3-5 天 |
| **帧缓冲显示** | 需要 /dev/fb0 | SDL 模拟显示，但像素格式和分辨率需真机验证 | 1-2 天 |
| **系统性能调优** | 需要实际硬件性能数据 | 预留调优时间，记录可疑代码位置 | 持续 |
| **WiFi 网络连接** | 需要无线网卡驱动 | 本地回环测试，预留网络配置代码 | 1 天 |

### ⚠️ 无硬件时**部分可完成**的工作（需谨慎）

| 功能 | WSL2 能做的 | ❌ 真机后必须调整的 | 建议预研时间占比 |
|------|------------|-------------------|-----------------|
| **音频 Pipeline** | 算法验证、Opus 编解码 | ALSA 驱动、缓冲区大小、实时性、硬件延迟 | 70% |
| **显示驱动** | SDL 窗口渲染 | 帧缓冲 /dev/fb0、分辨率适配、像素格式 | 80% |
| **输入设备** | 鼠标键盘 | 触摸屏校准、手势识别、多点触摸 | 60% |
| **系统参数配置** | 编辑 conf 文件 | 实际运行时硬件参数（WiFi MAC、GPIO 编号等） | 90% |
| **网络通信** | 本地测试 | 实际 WiFi 延迟、丢包、重连逻辑 | 85% |

### WSL2 优势与限制

**优势**:
- ✅ 完整的 Linux 环境
- ✅ 与 Windows 文件系统互通
- ✅ 可运行 Docker
- ✅ 支持 GPU（CUDA for WSL）
- ✅ 网络配置灵活

**限制**:
- ❌ 无法直接访问 USB 设备（需要 USB/IP 方案）
- ❌ 无法运行 ARM 二进制（需要 qemu-user 或交叉编译）
- ❌ 无法测试硬件相关功能（GPIO、I2C、摄像头等）

---

## WSL2 系统配置

### 1.1 安装与更新

```powershell
# PowerShell 管理员模式
# 安装 WSL2
wsl --install -d Ubuntu-22.04

# 更新到 WSL2
wsl --set-version Ubuntu-22.04 2

# 设置为默认版本
wsl --set-default-version 2
```

### 1.2 WSL2 配置优化

创建/编辑 `%USERPROFILE%\.wslconfig`：

```ini
[wsl2]
memory=8GB
processors=4
swap=2GB
localhostForwarding=true

# 如需 GPU 加速
gpuSupport=true
```

在 WSL2 内部配置 `/etc/wsl.conf`：

```ini
[boot]
systemd=true

[user]
default=your-username

[interop]
enabled=true
appendWindowsPath=true

[filesystem]
umask=022
```

### 1.3 系统初始化

```bash
# 更新系统
sudo apt update && sudo apt upgrade -y

# 安装基础工具
sudo apt install -y \
    build-essential \
    git \
    curl \
    wget \
    vim \
    htop \
    tree \
    unzip \
    zip \
    net-tools \
    iputils-ping

# 设置时区
sudo timedatectl set-timezone Asia/Shanghai
```

---

## 开发工具链安装

### 2.1 C/C++ 开发环境

```bash
# GCC/G++
sudo apt install -y gcc g++ gdb cmake make

# 安装 SDL2（用于 LVGL Simulator）
sudo apt install -y \
    libsdl2-dev \
    libsdl2-image-dev \
    libsdl2-mixer-dev \
    libsdl2-ttf-dev

# 安装音频库
sudo apt install -y \
    libportaudio2 \
    libportaudiocpp0 \
    portaudio19-dev \
    libopus-dev \
    libopusfile-dev

# 安装 JSON 库
sudo apt install -y libjsoncpp-dev

# 安装 WebSocket++ 依赖
sudo apt install -y libboost-all-dev
```

### 2.2 Python 环境（AI 服务端）

```bash
# 安装 Python 3.10
sudo apt install -y python3.10 python3.10-venv python3.10-dev python3-pip

# 创建项目虚拟环境
python3.10 -m venv ~/echoai-env
source ~/echoai-env/bin/activate

# 升级 pip
pip install --upgrade pip

# 安装基础包
pip install \
    numpy \
    scipy \
    pillow \
    opencv-python \
    pyyaml \
    requests \
    websockets \
    aiohttp

# 安装 AI 相关包（按需安装）
pip install \
    torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cpu \
    funasr \
    modelscope
```

### 2.3 交叉编译工具链（预装）

虽然暂时没有硬件，但先准备好交叉编译器：

```bash
# 安装 ARM 交叉编译器
sudo apt install -y \
    gcc-arm-linux-gnueabihf \
    g++-arm-linux-gnueabihf \
    gdb-multiarch

# 验证安装
arm-linux-gnueabihf-gcc --version
```

### 2.4 开发工具

```bash
# 安装 VS Code Server（可选）
curl -fsSL https://code-server.dev/install.sh | sh

# 安装 Git LFS（用于大文件）
sudo apt install -y git-lfs
git lfs install

# 安装 tree（目录结构查看）
sudo apt install -y tree

# 安装 minicom（串口调试，USB 透传后可用）
sudo apt install -y minicom
```

### 2.5 网络工具

```bash
# 安装网络调试工具
sudo apt install -y \
    nmap \
    netcat \
    tcpdump \
    wireshark \
    socat

# 安装 mqtt 客户端（如有需要）
sudo apt install -y mosquitto-clients
```

---

## 分阶段学习计划

### Phase 1: 环境搭建周（第 1 周）

**目标**: 完成所有软件安装，验证环境

| 天数 | 任务 | 验证方式 |
|------|------|----------|
| Day 1 | WSL2 安装配置 | `wsl -l -v` 显示版本 2 |
| Day 2 | C/C++ 工具链 | `gcc --version` 正常输出 |
| Day 3 | SDL2 安装 | 运行 SDL2 测试程序 |
| Day 4 | Python 环境 | `python3.10 --version` |
| Day 5 | 交叉编译器 | `arm-linux-gnueabihf-gcc --version` |
| Day 6-7 | 克隆项目仓库 | `git submodule update --init --recursive` |

---

### Phase 2: LVGL Simulator（第 2-4 周）

**目标**: 在 PC 上运行 LVGL，掌握 GUI 开发

#### Week 2: LVGL 基础

```bash
# 克隆 LVGL 示例
mkdir -p ~/projects
cd ~/projects
git clone https://github.com/lvgl/lv_port_pc_eclipse.git
cd lv_port_pc_eclipse
git submodule update --init --recursive

# 编译 Simulator
mkdir build && cd build
cmake .. && make -j$(nproc)

# 运行
./bin/main
```

**学习内容**:
- [ ] LVGL 显示驱动概念
- [ ] 基础组件：Button, Label, Image
- [ ] 事件系统：clicked, pressed, released
- [ ] 样式系统：颜色、边框、阴影
- [ ] 布局：Flex、Grid

**练习任务**:
1. 创建一个登录界面（用户名、密码输入框，登录按钮）
2. 实现页面切换动画
3. 添加图片显示功能

#### Week 3: Echo-Mate GUI 预研

```bash
# 克隆 Echo-Mate 上游仓库
cd ~/projects
git clone https://github.com/No-Chicken/Demo4Echo.git
cd Demo4Echo

# 分析 DeskBot_demo 的 GUI 结构
tree DeskBot_demo/gui_app/pages -L 2

# 尝试在 PC 上编译 Simulator 版本
```

**学习内容**:
- [ ] 阅读 `DeskBot-Demo---AI-Desktop-Assistant.md`
- [ ] 理解 11 个页面的架构
- [ ] 学习 Page Manager (lv_lib_pm)
- [ ] 分析动画实现

**练习任务**:
1. 复刻 HomePage 界面布局
2. 实现天气卡片组件
3. 创建滑动导航菜单

#### Week 4: 高级 GUI 功能

**学习内容**:
- [ ] 自定义绘图（Canvas）
- [ ] 动画缓动函数
- [ ] 主题切换
- [ ] 字体和图标
- [ ] 输入设备处理

**练习任务**:
1. 实现计算器界面
2. 创建日历组件
3. 实现设置页面的开关、滑块控件

---

### Phase 3: WebSocket 通信（第 5-6 周）

**目标**: 实现客户端-服务端通信

#### Week 5: WebSocket 基础

```bash
# 创建测试项目
mkdir -p ~/projects/websocket-test
cd ~/projects/websocket-test

# 安装 WebSocket 库
sudo apt install -y libwebsocketpp-dev

# 创建简单测试代码
```

**学习内容**:
- [ ] WebSocket 协议握手
- [ ] WebSocket++ 基础用法
- [ ] JSON 消息封装 (cJSON)
- [ ] 连接管理（重连、心跳）
- [ ] 错误处理

**代码示例**:

```cpp
// websocket_client.cpp
#include <websocketpp/config/asio_client.hpp>
#include <websocketpp/client.hpp>
#include <iostream>

typedef websocketpp::client<websocketpp::config::asio_client> client;

class WebSocketClient {
public:
    void connect(const std::string& uri) {
        websocketpp::lib::error_code ec;
        client::connection_ptr con = m_client.get_connection(uri, ec);
        
        if (ec) {
            std::cout << "Connection error: " << ec.message() << std::endl;
            return;
        }
        
        m_client.connect(con);
    }
    
    void send(const std::string& msg) {
        websocketpp::lib::error_code ec;
        m_client.send(m_hdl, msg, websocketpp::frame::opcode::text, ec);
    }
    
private:
    client m_client;
    websocketpp::connection_hdl m_hdl;
};

int main() {
    WebSocketClient ws;
    ws.connect("ws://localhost:8000");
    return 0;
}
```

#### Week 6: Echo-Mate 协议实现

**学习内容**:
- [ ] 阅读 `Communication-Protocol.md`
- [ ] 实现 Echo-Mate JSON 协议
- [ ] 二进制音频流处理
- [ ] Opus 编解码集成
- [ ] 协议状态机设计

**练习任务**:
1. 实现协议消息封装/解析
2. 创建 Mock Server 进行测试
3. 实现音频数据分包发送
4. 压力测试（高频消息）

---

### Phase 4: AI 服务端（第 7-9 周）

**目标**: 完整实现 Python AI 服务端

#### Week 7: 基础服务端

```bash
# 创建项目结构
mkdir -p ~/projects/echoai-server
cd ~/projects/echoai-server

# 创建虚拟环境
python3.10 -m venv venv
source venv/bin/activate

# 安装依赖
pip install websockets aiohttp

# 创建目录结构
mkdir -p {src,models,config,tests}
```

**项目结构**:
```
echoai-server/
├── src/
│   ├── __init__.py
│   ├── server.py          # WebSocket 服务端
│   ├── asr_handler.py     # ASR 处理
│   ├── tts_handler.py     # TTS 处理
│   ├── chat_handler.py    # LLM 对话
│   └── audio_pipeline.py  # 音频处理
├── models/                # 模型文件
├── config/
│   └── server.conf
├── tests/
│   └── test_server.py
└── main.py               # 入口
```

**学习内容**:
- [ ] Python asyncio 编程
- [ ] WebSocket 服务端实现
- [ ] 多线程/多进程设计
- [ ] 配置文件管理

#### Week 8: 阿里云 API 集成

**准备工作**:
1. 注册阿里云账号
2. 开通 DashScope 服务
3. 创建 API Key
4. 申请 CosyVoice、SenseVoice 权限

```python
# 配置文件 config/server.conf
ALIYUN_API_KEY = "sk-xxxxxxxxxxxxxxxx"
COSY_VOICE_MODEL = "cosyvoice-v1"
SENSE_VOICE_MODEL = "sensevoice-v1"
DEFAULT_REGION = "cn-hangzhou"
```

**学习内容**:
- [ ] 阿里云 DashScope API 调用
- [ ] CosyVoice TTS 集成
- [ ] SenseVoice ASR 集成
- [ ] 通义千问 LLM 集成
- [ ] 流式响应处理
- [ ] API 错误处理与重试

**代码示例**:

```python
# src/asr_handler.py
import asyncio
import aiohttp
import json
from typing import AsyncIterator

class ASRHandler:
    def __init__(self, api_key: str):
        self.api_key = api_key
        self.base_url = "https://dashscope.aliyuncs.com/api/v1"
    
    async def recognize_stream(self, audio_data: bytes) -> AsyncIterator[str]:
        """流式语音识别"""
        headers = {
            "Authorization": f"Bearer {self.api_key}",
            "Content-Type": "application/json"
        }
        
        payload = {
            "model": "sensevoice-v1",
            "input": {
                "audio": audio_data.hex()
            }
        }
        
        async with aiohttp.ClientSession() as session:
            async with session.post(
                f"{self.base_url}/services/aigc/multimodal/...",
                headers=headers,
                json=payload
            ) as response:
                async for line in response.content:
                    if line:
                        yield line.decode('utf-8')
```

#### Week 9: 完整 Pipeline

**音频 Pipeline 实现**:

```
音频输入 → VAD 检测 → ASR 识别 → 意图理解 → LLM 对话 → TTS 合成 → 音频输出
    ↑                                                       ↓
    └──────────────── 对话上下文管理 ───────────────────────┘
```

**学习内容**:
- [ ] VAD 集成（FSMN-VAD）
- [ ] 意图分类（FastText）
- [ ] 对话上下文管理
- [ ] 音频缓存与播放
- [ ] 性能优化

**练习任务**:
1. 实现完整的语音对话流程
2. 添加天气查询功能（高德 API）
3. 实现语音控制指令（播放音乐、查询时间）
4. 压力测试（并发连接）

---

### Phase 5: YOLOv5 预研（第 10-11 周）

**目标**: PC 端完成模型训练和验证

#### Week 10: YOLOv5 基础

```bash
# 克隆 YOLOv5
mkdir -p ~/projects
cd ~/projects
git clone https://github.com/ultralytics/yolov5.git
cd yolov5

# 安装依赖
pip install -r requirements.txt
```

**学习内容**:
- [ ] YOLOv5 架构理解
- [ ] 数据集准备（COCO 格式）
- [ ] 模型训练
- [ ] 模型评估
- [ ] ONNX 导出

**练习任务**:
1. 下载 COCO 数据集子集
2. 训练自定义类别（如：猫、狗、人）
3. 导出 ONNX 模型
4. 在 PC 上测试推理

#### Week 11: RKNN 准备

**目标**: 准备 RV1106 部署所需文件

**学习内容**:
- [ ] RKNN Toolkit2 安装（PC 版）
- [ ] ONNX 转 RKNN
- [ ] 量化配置（FP16/INT8）
- [ ] 模型优化

```bash
# 安装 RKNN Toolkit2
pip install rknn-toolkit2

# 转换脚本（准备）
# 注意：需要真机才能完整测试，但可以先准备脚本
```

**代码示例**:

```python
# convert_to_rknn.py
from rknn.api import RKNN

rknn = RKNN(verbose=True)

# 配置模型
rknn.config(
    mean_values=[[0, 0, 0]],
    std_values=[[255, 255, 255]],
    target_platform='rv1106'
)

# 加载 ONNX
rknn.load_onnx(model='yolov5s.onnx')

# 构建 RKNN
rknn.build(do_quantization=True, dataset='dataset.txt')

# 导出
rknn.export_rknn('yolov5s.rknn')
```

---

### Phase 6: 代码预研与架构（第 12 周）

**目标**: 深入理解项目架构，准备真机代码

#### 学习任务

**阅读文档**:
- [ ] `deepwiki/Overview.md` - 整体架构
- [ ] `deepwiki/Server-Architecture.md` - 服务端设计
- [ ] `deepwiki/Communication-Protocol.md` - 通信协议
- [ ] `deepwiki/Configuration-Reference.md` - 配置系统

**代码分析**:
```bash
# 分析上游代码结构
cd ~/projects/Demo4Echo

# 分析 DeskBot_demo
tree DeskBot_demo -L 3

# 分析关键文件
vim DeskBot_demo/main.c              # 入口
vim DeskBot_demo/gui_app/pages/*.c   # 页面实现
vim AIChat_demo/Client/*.cc          # AI 客户端
vim AIChat_demo/Server/main.py       # AI 服务端
```

**输出物**:
1. 系统架构图（自己绘制）
2. 模块依赖关系图
3. 关键数据流程图
4. 自己的代码实现计划

---

## Simulator 模式开发

### 4.1 编译配置

```bash
cd ~/projects/Demo4Echo/DeskBot_demo

# 修改配置
cat conf/dev_conf.h
# 确保：#define LV_USE_SIMULATOR 1

# 编译
mkdir -p build && cd build
cmake .. && make -j$(nproc)

# 运行
./bin/main
```

### 4.2 Simulator 限制

| 功能 | Simulator | 真机差异 |
|------|-----------|----------|
| 显示 | SDL 窗口 | /dev/fb0 帧缓冲 |
| 输入 | 鼠标/键盘 | 触摸屏 |
| 音频 | PortAudio | ALSA + 硬件 |
| 网络 | 正常 | 可能需 WiFi 配置 |
| 摄像头 | 模拟/禁用 | MIPI 摄像头 |
| NPU | CPU 模拟 | RKNN 加速 |
| GPIO | 不可用 | 可用 |

### 4.3 Simulator 开发技巧

**调试技巧**:
```bash
# 启用调试输出
export LV_LOG_LEVEL=DEBUG

# 使用 GDB
gdb ./bin/main
(gdb) break main
(gdb) run
```

**性能分析**:
```bash
# 使用 valgrind 检查内存
sudo apt install -y valgrind
valgrind --leak-check=full ./bin/main
```

---

## AI 服务端开发

### 5.1 完整服务端代码框架

```python
#!/usr/bin/env python3
# main.py - Echo-Mate AI Server

import asyncio
import websockets
import json
import logging
from typing import Dict, Set

# 配置日志
logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

class EchoAIServer:
    def __init__(self, host='0.0.0.0', port=8000):
        self.host = host
        self.port = port
        self.clients: Set[websockets.WebSocketServerProtocol] = set()
        
    async def register(self, websocket):
        self.clients.add(websocket)
        logger.info(f"Client connected: {websocket.remote_address}")
    
    async def unregister(self, websocket):
        self.clients.discard(websocket)
        logger.info(f"Client disconnected: {websocket.remote_address}")
    
    async def handle_message(self, websocket, message):
        """处理客户端消息"""
        try:
            data = json.loads(message)
            msg_type = data.get('type')
            
            if msg_type == 'chat':
                response = await self.handle_chat(data)
                await websocket.send(json.dumps(response))
            elif msg_type == 'audio':
                response = await self.handle_audio(data)
                await websocket.send(json.dumps(response))
            elif msg_type == 'ping':
                await websocket.send(json.dumps({'type': 'pong'}))
            else:
                logger.warning(f"Unknown message type: {msg_type}")
                
        except json.JSONDecodeError:
            logger.error("Invalid JSON received")
            await websocket.send(json.dumps({'type': 'error', 'msg': 'Invalid JSON'}))
    
    async def handle_chat(self, data):
        """处理对话请求"""
        # TODO: 调用阿里云 LLM API
        return {
            'type': 'chat_response',
            'text': '这是模拟回复',
            'session_id': data.get('session_id')
        }
    
    async def handle_audio(self, data):
        """处理音频数据"""
        # TODO: ASR → LLM → TTS
        return {
            'type': 'audio_response',
            'audio_url': 'http://...',
            'text': '识别的文本'
        }
    
    async def handler(self, websocket, path):
        """WebSocket 连接处理器"""
        await self.register(websocket)
        try:
            async for message in websocket:
                await self.handle_message(websocket, message)
        except websockets.exceptions.ConnectionClosed:
            logger.info("Connection closed")
        finally:
            await self.unregister(websocket)
    
    async def start(self):
        """启动服务器"""
        logger.info(f"Starting server on {self.host}:{self.port}")
        async with websockets.serve(self.handler, self.host, self.port):
            await asyncio.Future()  # 永久运行

if __name__ == '__main__':
    server = EchoAIServer()
    asyncio.run(server.start())
```

### 5.2 运行服务端

```bash
cd ~/projects/echoai-server
source venv/bin/activate

# 运行
python main.py --access_token="123456" --aliyun_api_key="sk-xxx"

# 测试连接
# 在另一个终端
wscat -c ws://localhost:8000
> {"type": "ping"}
< {"type": "pong"}
```

---

## 代码预研与架构设计

### 6.1 代码仓库结构规划

建议的 myGo/code 结构：

```
myGo/
├── code/
│   ├── lvgl-simulator/     # LVGL 学习代码
│   │   ├── src/
│   │   ├── include/
│   │   └── CMakeLists.txt
│   ├── websocket-client/   # WebSocket 客户端
│   │   ├── src/
│   │   └── CMakeLists.txt
│   ├── audio-test/         # 音频处理测试
│   │   └── portaudio-test/
│   ├── yolo-pc/            # PC 端 YOLOv5
│   │   ├── train/
│   │   └── inference/
│   └── ai-server/          # Python AI 服务端
│       ├── src/
│       ├── tests/
│       └── requirements.txt
├── Plan/                   # 学习文档
└── docs/                   # 个人笔记
```

### 6.2 关键代码预研点

**LVGL 页面管理**:
```c
// 学习 lv_lib_pm 的使用
#include "lv_lib_pm.h"

// 页面注册
lv_pm_page_t *home_page = lv_pm_create_page("home", home_page_create, home_page_destroy);
lv_pm_open_page(home_page, NULL);
```

**WebSocket 消息处理**:
```c
// 学习协议解析
typedef struct {
    char type[32];
    char text[256];
    char session_id[64];
} EchoMessage;

void parse_message(const char *json_str, EchoMessage *msg) {
    // 使用 cJSON 解析
}
```

**音频 Pipeline**:
```c
// 学习音频数据流转
void audio_pipeline_process(int16_t *input_buffer, size_t samples) {
    // 1. VAD 检测
    // 2. Opus 编码
    // 3. WebSocket 发送
}
```

---

## 硬件准备清单

### 7.1 必需硬件

| 序号 | 物品 | 预估价格 | 购买渠道 |
|------|------|---------|----------|
| 1 | RV1106 开发板 | ¥200-400 | 立创商城、淘宝 |
| 2 | USB 转 TTL 串口线 | ¥15-30 | 淘宝、京东 |
| 3 | 5V/2A 电源适配器 | ¥20 | 淘宝 |
| 4 | TF 卡 32GB | ¥30 | 京东 |
| 5 | USB 摄像头 (OV5640/IMX415) | ¥50-150 | 淘宝 |
| 6 | USB 麦克风 + 音箱 | ¥50-100 | 京东 |
| 7 | 外壳配件 | ¥100-200 | 立创开源平台 |

### 7.2 推荐开发板

**选项 1: 立创开发板**
- 型号: 立创·泰山派（RV1106）
- 价格: ¥299（套件）
- 优点: 社区活跃，文档齐全

**选项 2: 正点原子**
- 型号: 正点原子 RV1106 开发板
- 价格: ¥350-500
- 优点: 教程丰富

**选项 3: 自制 PCB**
- 参考: `assets/元器件清单.md`
- 成本: ¥150-250（BOM）
- 难度: 高，需要焊接 SMD

### 7.3 预购检查清单

- [ ] 确认开发板包含 RV1106 核心板
- [ ] 检查是否有 MIPI CSI 摄像头接口
- [ ] 确认有 USB OTG 接口（用于烧录）
- [ ] 检查是否有 UART 调试接口
- [ ] 购买前询问卖家提供 SDK 资料

---

## 从 Simulator 到真板的迁移

### 8.1 代码迁移检查清单

**显示驱动迁移**:
```c
// Simulator (SDL)
#define LV_USE_SDL 1

// Real Hardware (FrameBuffer)
#define LV_USE_LINUX_FBDEV 1
#define LV_LINUX_FBDEV_BSD 1
```

**音频迁移**:
```c
// Simulator (PortAudio)
Pa_OpenDefaultStream(...)

// Real Hardware (ALSA)
snd_pcm_open(&handle, "default", SND_PCM_STREAM_PLAYBACK, 0)
```

**配置修改**:
```bash
# 修改 dev_conf.h
sed -i 's/LV_USE_SIMULATOR 1/LV_USE_SIMULATOR 0/' conf/dev_conf.h

# 重新编译
cd build
rm -rf *
cmake .. -DTARGET_ARM=ON
make
```

### 8.2 调试技巧

**串口调试**:
```bash
# 连接串口到 WSL2
# Windows 端（PowerShell 管理员）:
usbipd wsl attach --busid <busid>

# WSL2 端:
sudo minicom -D /dev/ttyUSB0 -b 1500000
```

**网络调试**:
```bash
# 开发板 IP: 172.32.0.93
# 服务端 IP: 172.32.0.100 (WSL2)

# 从 WSL2 ping 开发板
ping 172.32.0.93

# SSH 连接（如已配置）
ssh root@172.32.0.93
```

### 8.3 常见问题

**问题 1: 交叉编译失败**
```bash
# 检查工具链
which arm-linux-gnueabihf-gcc
arm-linux-gnueabihf-gcc --version

# 检查环境变量
export CROSS_COMPILE=arm-linux-gnueabihf-
export ARCH=arm
```

**问题 2: 运行时库缺失**
```bash
# 检查依赖
arm-linux-gnueabihf-readelf -d bin/main | grep NEEDED

# 复制库到目标
scp /usr/arm-linux-gnueabihf/lib/libxxx.so root@172.32.0.93:/lib/
```

**问题 3: 显示异常**
```bash
# 检查帧缓冲
cat /sys/class/graphics/fb0/virtual_size

# 检查权限
ls -la /dev/fb0
```

---

## 📅 时间规划总结

### 16 周学习计划（无硬件）

| 周次 | 阶段 | 主要任务 | 输出物 |
|------|------|----------|--------|
| 1 | 环境搭建 | WSL2、工具链安装 | 可编译环境 |
| 2-4 | LVGL Simulator | GUI 开发学习 | 3 个练习界面 |
| 5-6 | WebSocket | 通信协议实现 | Mock Server |
| 7-9 | AI 服务端 | Python 服务端 | 完整服务端 |
| 10-11 | YOLOv5 | 模型训练 | ONNX 模型 |
| 12 | 架构预研 | 代码分析 | 架构文档 |
| 13-16 | 等待硬件 | 代码优化、测试 | 可部署代码 |

### 获得硬件后的 4 周

| 周次 | 任务 |
|------|------|
| 1 | 环境烧录、交叉编译验证 |
| 2 | LVGL 真机调试 |
| 3 | AI 服务端联调 |
| 4 | YOLOv5 + RKNN 部署 |

---

## 🔍 现实检查：无硬件开发的真正预期

### ✅ 无硬件时**可以高质量完成**的（投入产出比高）

| 任务 | 建议投入时间 | 真机后工作量 | 产出质量 |
|------|-------------|-------------|----------|
| **AI 服务端开发** | 40% | 1-2 天（仅测试）| ⭐⭐⭐⭐⭐ |
| **LVGL GUI 设计** | 30% | 2-3 天（触摸适配）| ⭐⭐⭐⭐ |
| **WebSocket 通信协议** | 15% | 半天（网络配置）| ⭐⭐⭐⭐⭐ |
| **YOLOv5 模型训练** | 15% | 3-5 天（RKNN 转换）| ⭐⭐⭐ |

### ❌ 无硬件时**必须跳过或标记**的（投入产出比低）

| 任务 | 建议处理方式 | 风险说明 |
|------|-------------|----------|
| **RKNN 部署** | ❌ 完全跳过，真机后再做 | 无硬件无法验证，不要做无用功 |
| **GPIO 控制** | ⚠️ 用 Mock 函数占位 | 代码占位，真机后替换实现 |
| **ALSA 音频** | ⚠️ 用 PortAudio 模拟 | 注意：延迟差异需真机大改 |
| **MIPI 摄像头** | ⚠️ 用图片/视频模拟输入 | 数据流逻辑可验证，驱动真机写 |
| **系统性能调优** | ❌ 完全跳过 | 无性能数据无法调优 |

### 📊 修正后的预期

```
无硬件阶段完成度：约 65-75%
├── AI 服务端：100% ✅
├── GUI 界面：90% ✅（需触摸适配）
├── WebSocket：100% ✅
├── 音频处理：60% ⚠️（ALSA 需重写）
├── YOLOv5：80% ✅（RKNN 需转换）
├── 硬件驱动：10% ❌（仅预留接口）
└── 系统优化：0% ❌（必须真机）

获得真机后还需：4-6 周
├── Week 1: 环境烧录 + 基础测试
├── Week 2: 显示/触摸/音频适配
├── Week 3: RKNN + 摄像头调试
├── Week 4-5: 系统集成 + Bug 修复
└── Week 6: 性能优化 + 稳定性测试
```

### 🎯 建议的学习策略

**策略 1: 软件优先**
- 先完整开发 AI 服务端（产出最确定）
- 再用 SDL Simulator 做 GUI（90% 可复用）
- 音频/视频预留接口，真机后实现

**策略 2: 不要试图模拟硬件**
- ❌ 不要试图在 WSL2 模拟 GPIO 状态
- ❌ 不要试图用 QEMU 运行 ARM 程序（除非有特定需求）
- ✅ 用 Mock 数据替代硬件输入

**策略 3: 预留适配代码**
```c
// 好的做法：预留硬件抽象层
#ifdef SIMULATOR
    // SDL 模拟实现
#else
    // 真机硬件实现（暂时空着）
#endif
```

## 🔗 相关文档

- [根目录 AGENTS.md](/home/GreedArch/project/echoAI/AGENTS.md)
- [技术栈学习计划](./Echo-Mate-技术栈学习计划.md)
- [DeepWiki 文档](/home/GreedArch/project/echoAI/deepwiki/)
- [硬件 BOM 清单](/home/GreedArch/project/echoAI/assets/元器件清单.md)

---

## 📚 附录：常用命令速查

### WSL2 常用

```bash
# 进入 WSL2
wsl

# 从 Windows 访问 WSL 文件
\\wsl$\Ubuntu-22.04\home\username

# 从 WSL 访问 Windows 文件
/mnt/c/Users/Username

# 重启 WSL
wsl --shutdown
```

### Git 常用

```bash
# 克隆仓库
git clone https://github.com/No-Chicken/Demo4Echo.git

# 初始化子模块
git submodule update --init --recursive

# 查看仓库大小
du -sh .git
```

### CMake 常用

```bash
# 创建构建目录
mkdir build && cd build

# 配置
cmake ..

# 编译
make -j$(nproc)

# 清理
rm -rf build/*
```

---

*文档生成时间: 2026-04-14*  
*适用环境: WSL2 + Ubuntu 22.04*