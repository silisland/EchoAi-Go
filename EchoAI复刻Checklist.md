# EchoAI复刻技术Checklist

## 开发环境准备

### 硬件准备
- [ ] RV1106开发板
- [ ] 触摸屏（320x240或更大）
- [ ] 麦克风模块
- [ ] 扬声器
- [ ] WiFi模块
- [ ] USB摄像头（用于YOLOv5）
- [ ] 电源适配器

### 软件环境
- [ ] 安装Ubuntu/WSL2开发环境
- [ ] 配置ARM交叉编译工具链
- [ ] 安装必要的依赖包（cmake, SDL2, portaudio, opus, jsoncpp）
- [ ] 配置Git并克隆Demo4Echo仓库
- [ ] 初始化Git子模块：`git submodule update --init --recursive`

---

## Phase 1：基础架构复刻

### 1.1 DeskBot GUI系统
- [ ] 编译DeskBot SDL版本（Ubuntu上测试）
  ```bash
  # 修改DeskBot_demo/conf/dev_conf.h
  #define LV_USE_SIMULATOR 1
  
  cd DeskBot_demo
  mkdir build && cd build
  cmake ..
  make
  ./bin/main
  ```
- [ ] 成功运行LVGL界面，看到11个应用图标
- [ ] 验证页面切换功能正常

### 1.2 配置文件设置
- [ ] 修改`system_para.conf`中的网络配置
  - [ ] WiFi SSID/密码
  - [ ] 服务器IP地址
  - [ ] API密钥（阿里云DashScope）
- [ ] 测试NTP时间同步
- [ ] 测试天气API（高德地图）

---

## Phase 2：语音对话功能

### 2.1 AIChat Server部署
- [ ] 在Linux服务器/Ubuntu上部署Python Server
  ```bash
  cd AIChat_demo/Server
  conda create -n aichat python=3.10
  conda activate aichat
  pip install -r requirements.txt
  ```
- [ ] 下载SenseVoice模型
  ```bash
  cd models/FunAudioLLM
  git submodule update --init --recursive
  ```
- [ ] 配置阿里云API密钥
- [ ] 启动Server：`python main.py --access_token="123456" --aliyun_api_key="sk-xxx"`
- [ ] 验证WebSocket服务在端口8000运行

### 2.2 AIChat Client编译
- [ ] 编译Client SDL版本
  ```bash
  cd AIChat_demo/Client
  mkdir build && cd build
  cmake ..
  make
  ```
- [ ] 测试WebSocket连接
- [ ] 测试音频采集（PortAudio）
- [ ] 测试语音识别→对话→语音合成完整流程

### 2.3 DeskBot集成
- [ ] 在DeskBot中打开ChatBot页面
- [ ] 验证语音对话功能
- [ ] 测试功能注册（robot_move等）

---

## Phase 3：物体检测功能

### 3.1 YOLOv5 Standalone测试
- [ ] 获取YOLOv5 RKNN模型
- [ ] 编译YOLOv5 demo（SDL版本用于测试）
  ```bash
  cd yolov5_demo/cpp
  mkdir build && cd build
  cmake ..
  make
  ```
- [ ] 连接USB摄像头
- [ ] 运行YOLOv5 demo，验证物体检测

### 3.2 DeskBot集成
- [ ] 在DeskBot中编译YOLO页面
- [ ] 验证YOLO页面显示摄像头画面
- [ ] 测试手势返回功能

---

## Phase 4：ARM目标编译

### 4.1 硬件编译配置
- [ ] 修改`dev_conf.h`：`#define LV_USE_SIMULATOR 0`
- [ ] 配置交叉编译工具链
  ```bash
  # 在DeskBot_demo/build
  cmake .. -DTARGET_ARM=ON
  make
  ```

### 4.2 部署到RV1106
- [ ] 将bin目录复制到RV1106
- [ ] 配置RV1106网络连接
- [ ] 启动DeskBot应用程序
- [ ] 验证GUI显示正常

### 4.3 硬件功能验证
- [ ] 触摸屏响应
- [ ] 音频输入（麦克风）
- [ ] 音频输出（扬声器）
- [ ] WiFi连接
- [ ] 摄像头（YOLOv5）

---

## Phase 5：PicoClaw集成（进阶）

### 5.1 PicoClaw交叉编译
- [ ] 安装Go语言交叉编译环境
- [ ] 配置ARM目标编译参数
- [ ] 解决CGO依赖（SQLite等）
- [ ] 静态链接编译PicoClaw

### 5.2 PicoClaw适配
- [ ] 修改PicoClaw配置文件
- [ ] 开发PicoClaw与EchoAI的通信接口
- [ ] 实现双方案切换逻辑

### 5.3 LVGL界面增强
- [ ] 设计PicoClaw状态显示UI
- [ ] 添加"养龙虾"动画效果
- [ ] 集成Skill控制界面

---

## 问题记录

| 问题描述 | 发生时间 | 解决方案 | 状态 |
|----------|----------|----------|------|
| | | | |

---

## 参考资料

- GitHub: https://github.com/No-Chicken/Demo4Echo
- DeepWiki文档: `/home/GreedArch/project/echoAI/deepwiki/`
- AGENTS.md: `/home/GreedArch/project/echoAI/AGENTS.md`
- 技术融合方案: `/home/GreedArch/project/echoAI/技术融合方案.md`

---

## 下一步行动

1. **立即开始**：环境搭建和仓库克隆
2. **本周目标**：成功运行DeskBot SDL版本
3. **里程碑**：语音对话功能跑通
4. **最终目标**：完整EchoAI复刻 + PicoClaw集成
