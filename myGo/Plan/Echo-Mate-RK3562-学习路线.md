# Echo-Mate 项目 - RK3562 学习路线图

**适用硬件**: RK3562 开发板  
**目标**: 掌握 RK3562 平台开发，为未来 RV1106v3 项目做准备  
**学习周期**: 12-16 周  
**最后更新**: 2026-04-14

---

## 🎯 学习路线图概览

```
Phase 1 (Week 1-2): 环境搭建与基础验证
├── WSL2 + 交叉编译器安装
├── RK3562 SDK 编译
└── 基础系统烧录与测试

Phase 2 (Week 3-5): Linux 系统与驱动
├── Buildroot 定制
├── 设备树 (Device Tree)
├── 驱动开发基础
└── 内核模块编译

Phase 3 (Week 6-8): 多媒体与显示
├── DRM/KMS 显示框架
├── MIPI-DSI 屏幕驱动
├── 摄像头 (MIPI-CSI)
└── V4L2 框架

Phase 4 (Week 9-11): AI 与 NPU
├── RKNN Toolkit 使用
├── 模型转换与量化
├── YOLOv5 部署
└── 多线程 NPU 推理

Phase 5 (Week 12-14): 网络与通信
├── WebSocket 客户端
├── 音频处理 (ALSA)
├── WiFi/以太网配置
└── 系统集成

Phase 6 (Week 15-16): 项目实战
├── LVGL 移植
├── AI Pipeline 集成
├── 性能优化
└── 完整 Echo-Mate 移植
```

---

## 📊 RK3562 vs 项目需求匹配度

| 项目需求 | RK3562 能力 | 匹配度 | 说明 |
|----------|------------|--------|------|
| **LVGL GUI** | Mali-G52 GPU | ⭐⭐⭐⭐⭐ | 硬件加速，动画流畅 |
| **YOLOv5** | 1 TOPS NPU | ⭐⭐⭐⭐⭐ | 比 RV1106 快 2 倍 |
| **ASR/TTS** | 4核 A53 | ⭐⭐⭐⭐⭐ | 并行处理无压力 |
| **WebSocket** | GbE 网口 | ⭐⭐⭐⭐⭐ | 低延迟通信 |
| **音频** | I2S/SAI | ⭐⭐⭐⭐ | 需配置音频驱动 |
| **摄像头** | MIPI-CSI x2 | ⭐⭐⭐⭐⭐ | 双摄像头支持 |

**结论**: RK3562 性能远超项目需求，是学习嵌入式 AI 的理想平台。

---

## Phase 1: 环境搭建 (Week 1-2)

### Week 1: 开发环境准备

#### Day 1-2: WSL2 基础环境

```bash
# 1. WSL2 Ubuntu 22.04 安装
wsl --install -d Ubuntu-22.04

# 2. 进入 WSL2
wsl

# 3. 基础工具安装
sudo apt update && sudo apt upgrade -y
sudo apt install -y \
    build-essential \
    git \
    vim \
    cmake \
    make \
    tree \
    htop
```

#### Day 3-4: RK3562 交叉编译器

```bash
# 安装 aarch64 交叉编译器
sudo apt install -y \
    gcc-aarch64-linux-gnu \
    g++-aarch64-linux-gnu \
    gdb-multiarch \
    binutils-aarch64-linux-gnu

# 验证安装
aarch64-linux-gnu-gcc --version
# 应显示: aarch64-linux-gnu-gcc (Ubuntu 11.3.0) 11.3.0

# 创建工具链文件
cat > ~/rk3562-toolchain.cmake << 'EOF'
set(CMAKE_SYSTEM_NAME Linux)
set(CMAKE_SYSTEM_PROCESSOR aarch64)
set(CMAKE_C_COMPILER aarch64-linux-gnu-gcc)
set(CMAKE_CXX_COMPILER aarch64-linux-gnu-g++)
set(CMAKE_FIND_ROOT_PATH_MODE_PROGRAM NEVER)
set(CMAKE_FIND_ROOT_PATH_MODE_LIBRARY ONLY)
set(CMAKE_FIND_ROOT_PATH_MODE_INCLUDE ONLY)
set(CMAKE_C_FLAGS "-march=armv8-a -O2")
set(CMAKE_CXX_FLAGS "-march=armv8-a -O2")
EOF
```

#### Day 5-7: SDK 下载与编译

```bash
# 创建工作目录
mkdir -p ~/projects/rk3562
cd ~/projects/rk3562

# 下载 RK3562 SDK
# 方式1: 从开发板供应商获取
# 方式2: 从 Rockchip 官方 GitHub
git clone https://github.com/rockchip-linux/buildroot.git -b stable
cd buildroot

# 加载 RK3562 配置
make rockchip_rk3562_defconfig

# 配置额外选项
make menuconfig
# 选择需要的包:
# - Target packages -> Libraries -> Graphics -> [*] lvgl
# - Target packages -> Libraries -> [*] jsoncpp
# - Target packages -> Audio/Sound -> [*] alsa-utils
# - Target packages -> Audio/Sound -> [*] alsa-lib

# 开始编译 (需要数小时)
make -j$(nproc) 2>&1 | tee build.log
```

### Week 2: 系统烧录与验证

#### Day 1-3: 烧录工具准备

```bash
# Linux 安装 rkdeveloptool
sudo apt install -y libusb-1.0-0-dev libudev-dev

git clone https://github.com/rockchip-linux/rkdeveloptool.git
cd rkdeveloptool
autoreconf -i
./configure
make
sudo cp rkdeveloptool /usr/local/bin/

# Windows 下载 RKDevTool (瑞芯微开发工具)
```

#### Day 4-5: 系统烧录

```bash
# 1. 进入烧录模式
#    - 按住 RECOVERY 键
#    - 插入 USB Type-C 供电
#    - 松开 RECOVERY 键

# 2. 检查设备连接
lsusb | grep 2207
# 输出: Bus 001 Device 002: ID 2207:xxxx Fuzhou Rockchip Electronics

# 3. 烧录镜像
sudo rkdeveloptool db MiniLoaderAll.bin
sudo rkdeveloptool wl 0x0 boot.img
sudo rkdeveloptool wl 0x8000 rootfs.img
sudo rkdeveloptool rd
```

#### Day 6-7: 首次启动验证

```bash
# 串口登录 (波特率 1500000)
minicom -D /dev/ttyUSB0 -b 1500000

# 登录后验证
uname -a
# Linux rk3562-buildroot 5.10.198 #1 SMP aarch64 GNU/Linux

# 检查 CPU 信息
cat /proc/cpuinfo | grep "model name"
# 应显示 4 个 Cortex-A53 核心

# 检查内存
free -h
# 应显示 > 1GB 可用内存

# 检查 NPU
ls /dev/rknpu*
# 应存在 /dev/rknpu0
```

### ✅ Phase 1 检查点

- [ ] WSL2 Ubuntu 22.04 运行正常
- [ ] aarch64-linux-gnu-gcc 安装完成
- [ ] RK3562 SDK 编译成功
- [ ] 系统镜像烧录成功
- [ ] 串口登录正常
- [ ] 4 个 CPU 核心识别
- [ ] NPU 设备节点存在

---

## Phase 2: Linux 系统与驱动 (Week 3-5)

### Week 3: Buildroot 深度定制

#### 学习目标
- 理解 Buildroot 配置系统
- 添加自定义软件包
- 创建根文件系统 overlay

#### 实践任务

```bash
cd ~/projects/rk3562/buildroot

# 1. 配置内核
make linux-menuconfig
# 启用:
# - Device Drivers -> Multimedia support -> V4L2
# - Device Drivers -> Graphics support -> Rockchip DRM

# 2. 配置 BusyBox
make busybox-menuconfig
# 添加常用工具

# 3. 创建 Overlay (持久化修改)
mkdir -p board/rockchip/rk3562/rootfs-overlay/etc

# 自定义网络配置
cat > board/rockchip/rk3562/rootfs-overlay/etc/network/interfaces << 'EOF'
auto eth0
iface eth0 inet static
    address 192.168.1.100
    netmask 255.255.255.0
    gateway 192.168.1.1

auto wlan0
iface wlan0 inet dhcp
    wpa-conf /etc/wpa_supplicant.conf
EOF

# 4. 重新编译
make -j$(nproc)
```

### Week 4: 设备树 (Device Tree)

#### 理论基础
- 设备树是描述硬件的标准化方式
- 取代传统的硬编码平台数据
- 设备树编译器 (dtc) 将 .dts 编译为 .dtb

#### 实践任务

```bash
# 1. 查看当前设备树
cat /sys/firmware/devicetree/base/model
# 显示: Rockchip RK3562 EVB

# 2. 反编译设备树
cd /sys/firmware/devicetree/base/
find . -type f | head -20

# 3. 设备树结构理解
# arch/arm64/boot/dts/rockchip/rk3562-evb.dts

# 4. 修改设备树 (添加 GPIO 按键)
cat >> arch/arm64/boot/dts/rockchip/rk3562-evb.dts << 'EOF'
&gpio0 {
    button_pwr: button-pwr {
        label = "power-button";
        gpios = <&gpio0 RK_PC0 GPIO_ACTIVE_LOW>;
        linux,code = <KEY_POWER>;
    };
};
EOF

# 5. 重新编译设备树
make dtbs
```

#### 学习要点
- 理解 `compatible` 属性
- 学习 `reg`, `interrupts`, `clocks` 等标准属性
- 掌握设备树节点引用 (`&label`)
- GPIO 和 pinctrl 配置

### Week 5: 驱动开发基础

#### 学习目标
- 理解 Linux 驱动模型
- 编写简单的字符设备驱动
- 掌握调试技巧

#### 实践: Hello World 驱动

```c
// hello.c - 最简单的内核模块
#include <linux/module.h>
#include <linux/kernel.h>
#include <linux/init.h>

static int __init hello_init(void) {
    printk(KERN_INFO "Hello, RK3562!\n");
    return 0;
}

static void __exit hello_exit(void) {
    printk(KERN_INFO "Goodbye, RK3562!\n");
}

module_init(hello_init);
module_exit(hello_exit);

MODULE_LICENSE("GPL");
MODULE_AUTHOR("Learner");
MODULE_DESCRIPTION("RK3562 Hello World Driver");
```

```makefile
# Makefile
obj-m += hello.o

KDIR := ~/projects/rk3562/buildroot/output/build/linux-custom

all:
    make -C $(KDIR) M=$(PWD) modules \
        ARCH=arm64 \
        CROSS_COMPILE=aarch64-linux-gnu-

clean:
    make -C $(KDIR) M=$(PWD) clean
```

#### 加载与测试

```bash
# 在开发板上
insmod hello.ko
dmesg | tail
# 输出: Hello, RK3562!

rmmod hello
dmesg | tail
# 输出: Goodbye, RK3562!
```

### ✅ Phase 2 检查点

- [ ] 能够修改 Buildroot 配置
- [ ] 理解设备树结构
- [ ] 成功反编译/修改设备树
- [ ] 编写并加载 Hello World 驱动
- [ ] 理解内核模块机制

---

## Phase 3: 多媒体与显示 (Week 6-8)

### Week 6: DRM/KMS 显示框架

#### 理论基础
- DRM (Direct Rendering Manager) - Linux 显示子系统
- KMS (Kernel Mode Setting) - 内核模式设置
- 替代传统 framebuffer，支持硬件加速

#### 实践任务

```bash
# 1. 检查 DRM 设备
ls /dev/dri/
# 应显示: card0  card1  controlD64  renderD128

# 2. 查看显示信息
cat /sys/kernel/debug/dri/0/state

# 3. 使用 modetest 工具
modetest -M rockchip -s 0:1920x1080

# 4. DRM 编程示例 (simple_drm.c)
# 学习:
# - drmModeGetResources()
# - drmModeGetConnector()
# - drmModeSetCrtc()
```

### Week 7: MIPI-DSI 屏幕驱动

#### 硬件连接
- RK3562 支持 MIPI-DSI 显示接口
- 常见屏幕: 7寸 1024x600, 10.1寸 1280x800

#### 设备树配置

```dts
// arch/arm64/boot/dts/rockchip/rk3562-evb.dts
&dsi {
    status = "okay";
    
    panel: panel@0 {
        compatible = "simple-panel-dsi";
        reg = <0>;
        
        dsi,format = <MIPI_DSI_FMT_RGB888>;
        dsi,lanes = <4>;
        
        panel-init-sequence = [
            /* 屏幕初始化序列 */
        ];
        
        display-timings {
            native-mode = <&timing0>;
            timing0: timing0 {
                clock-frequency = <71000000>;
                hactive = <1280>;
                vactive = <800>;
                hback-porch = <80>;
                hfront-porch = <80>;
                vback-porch = <12>;
                vfront-porch = <12>;
                hsync-len = <10>;
                vsync-len = <4>;
            };
        };
    };
};
```

### Week 8: 摄像头与 V4L2

#### 学习内容
- V4L2 (Video for Linux 2) 框架
- MIPI-CSI 摄像头接口
- 摄像头数据采集

#### 实践任务

```bash
# 1. 检查摄像头设备
ls /dev/video*
# 应显示: video0 video1

# 2. 使用 v4l2-ctl 工具
v4l2-ctl --list-devices
v4l2-ctl -d /dev/video0 --list-formats
v4l2-ctl -d /dev/video0 --set-fmt-video=width=1920,height=1080,pixelformat=YUYV

# 3. 拍摄测试图
v4l2-ctl -d /dev/video0 --stream-mmap --stream-count=1 --stream-to=/tmp/frame.raw

# 4. V4L2 编程示例 (capture.c)
# 学习:
# - open() 打开设备
# - ioctl() 设置格式
# - mmap() 内存映射
# - select() 等待帧
```

### ✅ Phase 3 检查点

- [ ] 理解 DRM/KMS 架构
- [ ] 成功运行 modetest
- [ ] MIPI-DSI 屏幕点亮
- [ ] 摄像头识别并捕获图像
- [ ] 理解 V4L2 编程模型

---

## Phase 4: AI 与 NPU (Week 9-11)

### Week 9: RKNN Toolkit 基础

#### 安装与配置

```bash
# PC 端 (WSL2)
pip install rknn-toolkit2

# 验证安装
python -c "from rknn.api import RKNN; print('OK')"
```

#### 模型转换流程

```python
# convert_model.py
from rknn.api import RKNN

# 1. 创建 RKNN 对象
rknn = RKNN(verbose=True)

# 2. 配置
rknn.config(
    mean_values=[[0, 0, 0]],
    std_values=[[255, 255, 255]],
    target_platform='rk3562',
    optimization_level=3
)

# 3. 加载模型
rknn.load_onnx(model='yolov5s.onnx')

# 4. 构建 (量化)
rknn.build(
    do_quantization=True,
    dataset='dataset.txt',  # 校准数据集
    quantized_dtype='float16'  # RK3562 支持 FP16
)

# 5. 导出
rknn.export_rknn('yolov5s_fp16.rknn')
```

### Week 10: YOLOv5 部署

#### 模型准备

```bash
# 1. 下载 YOLOv5
git clone https://github.com/ultralytics/yolov5.git
cd yolov5
pip install -r requirements.txt

# 2. 导出 ONNX
python export.py --weights yolov5s.pt --img 640 --batch 1 --include onnx

# 3. 转换为 RKNN
python convert_rk3562.py
```

#### C++ 推理代码

```cpp
// yolo_inference.cpp
#include <rknn_api.h>

class YOLODetector {
private:
    rknn_context ctx;
    
public:
    bool init(const char* model_path) {
        int ret = rknn_init(&ctx, model_path, 0, 0, nullptr);
        return ret == RKNN_SUCC;
    }
    
    bool inference(cv::Mat& input, std::vector<Detection>& output) {
        // 1. 设置输入
        rknn_input inputs[1];
        inputs[0].index = 0;
        inputs[0].type = RKNN_TENSOR_UINT8;
        inputs[0].fmt = RKNN_TENSOR_NHWC;
        inputs[0].buf = input.data;
        rknn_inputs_set(ctx, 1, inputs);
        
        // 2. 运行推理
        rknn_run(ctx, nullptr);
        
        // 3. 获取输出
        rknn_output outputs[3];
        for (int i = 0; i < 3; i++) {
            outputs[i].want_float = 1;
        }
        rknn_outputs_get(ctx, 3, outputs, nullptr);
        
        // 4. 后处理 (NMS)
        post_process(outputs, output);
        
        // 5. 释放输出
        rknn_outputs_release(ctx, 3, outputs);
        
        return true;
    }
};
```

#### 编译与运行

```bash
# 交叉编译
aarch64-linux-gnu-g++ \
    -o yolo_test yolo_inference.cpp \
    -I~/projects/rk3562/buildroot/output/host/include \
    -L~/projects/rk3562/buildroot/output/host/lib \
    -lrknnrt -lopencv_core -lopencv_imgcodecs \
    -march=armv8-a

# 上传到开发板运行
scp yolo_test yolov5s_fp16.rknn root@192.168.1.100:/opt/
```

### Week 11: 多线程 NPU 优化

#### 学习内容
- NPU 频率调节
- 多线程推理
- 性能监控

#### 优化技巧

```bash
# 1. NPU 频率设置
cat /sys/kernel/debug/clk/clk_scmi_npu/clk_rate
echo 1000000000 > /sys/kernel/debug/clk/clk_scmi_npu/clk_rate  # 1GHz

# 2. CPU 性能模式
for cpu in /sys/devices/system/cpu/cpu[0-3]/cpufreq/scaling_governor; do
    echo performance > $cpu
done

# 3. 内存优化
echo 3 > /proc/sys/vm/drop_caches
```

#### 多线程推理

```cpp
// 使用 OpenMP 并行处理 4 路视频
#pragma omp parallel for num_threads(4)
for (int i = 0; i < 4; i++) {
    detectors[i].inference(frames[i], results[i]);
}
```

### ✅ Phase 4 检查点

- [ ] RKNN Toolkit 安装成功
- [ ] YOLOv5 模型转换成功
- [ ] C++ 推理代码运行正常
- [ ] 理解 NPU 频率调节
- [ ] 性能测试 (FPS > 15)

---

## Phase 5: 网络与通信 (Week 12-14)

### Week 12: WebSocket 客户端

#### 学习内容
- WebSocket++ 库使用
- JSON 协议解析
- 多线程网络编程

#### 代码示例

```cpp
// websocket_client.cpp
#include <websocketpp/config/asio_no_tls_client.hpp>
#include <websocketpp/client.hpp>
#include <jsoncpp/json/json.h>

typedef websocketpp::client<websocketpp::config::asio> client;

class AIWebSocketClient {
private:
    client ws_client;
    websocketpp::connection_hdl connection;
    
public:
    void connect(const std::string& uri) {
        ws_client.init_asio();
        
        ws_client.set_message_handler(
            std::bind(&AIWebSocketClient::on_message, this, ...)
        );
        
        websocketpp::lib::error_code ec;
        client::connection_ptr con = ws_client.get_connection(uri, ec);
        
        ws_client.connect(con);
        ws_client.run();
    }
    
    void send_audio(const std::vector<uint8_t>& opus_data) {
        Json::Value msg;
        msg["type"] = "audio";
        msg["data"] = base64_encode(opus_data);
        
        ws_client.send(connection, msg.toStyledString(), ...);
    }
    
private:
    void on_message(websocketpp::connection_hdl hdl, client::message_ptr msg) {
        Json::Value response;
        Json::Reader reader;
        reader.parse(msg->get_payload(), response);
        
        std::string type = response["type"].asString();
        if (type == "chat_response") {
            process_chat_response(response);
        } else if (type == "audio_response") {
            play_audio(response["audio_url"].asString());
        }
    }
};
```

### Week 13: 音频处理 (ALSA)

#### 学习内容
- ALSA API
- 音频采集与播放
- Opus 编解码

#### 代码示例

```cpp
// audio_capture.cpp
#include <alsa/asoundlib.h>

class ALSAAudio {
private:
    snd_pcm_t* capture_handle;
    snd_pcm_t* playback_handle;
    
public:
    bool init_capture(int rate = 16000, int channels = 1) {
        snd_pcm_open(&capture_handle, "hw:0,0", SND_PCM_STREAM_CAPTURE, 0);
        
        snd_pcm_hw_params_t* params;
        snd_pcm_hw_params_alloca(&params);
        snd_pcm_hw_params_any(capture_handle, params);
        
        // 配置参数
        snd_pcm_hw_params_set_access(capture_handle, params, SND_PCM_ACCESS_RW_INTERLEAVED);
        snd_pcm_hw_params_set_format(capture_handle, params, SND_PCM_FORMAT_S16_LE);
        snd_pcm_hw_params_set_channels(capture_handle, params, channels);
        
        unsigned int val = rate;
        snd_pcm_hw_params_set_rate_near(capture_handle, params, &val, 0);
        
        snd_pcm_hw_params(capture_handle, params);
        return true;
    }
    
    int capture(short* buffer, int frames) {
        return snd_pcm_readi(capture_handle, buffer, frames);
    }
};
```

### Week 14: 系统集成

#### 整合 AI Pipeline

```cpp
// main.cpp - 完整 AI 流程
int main() {
    // 1. 初始化各模块
    AudioCapture audio;
    VADProcessor vad;
    WebSocketClient ws;
    YOLODetector yolo;
    
    audio.init();
    ws.connect("ws://ai-server:8000");
    yolo.init("/opt/models/yolov5s.rknn");
    
    // 2. 主循环
    while (true) {
        // 采集音频
        short pcm_buffer[1600];  // 100ms @ 16kHz
        audio.capture(pcm_buffer, 1600);
        
        // VAD 检测
        if (vad.is_speech(pcm_buffer)) {
            // Opus 编码
            std::vector<uint8_t> opus_data = opus_encode(pcm_buffer);
            
            // WebSocket 发送
            ws.send_audio(opus_data);
        }
        
        // YOLO 检测 (每 100ms 一次)
        if (frame_ready) {
            auto detections = yolo.detect(camera_frame);
            display_results(detections);
        }
    }
}
```

### ✅ Phase 5 检查点

- [ ] WebSocket 连接成功
- [ ] JSON 消息解析正确
- [ ] ALSA 音频采集正常
- [ ] Opus 编解码工作
- [ ] AI Pipeline 集成测试

---

## Phase 6: 项目实战 (Week 15-16)

### Week 15: LVGL 移植

#### 移植步骤

```c
// 1. 修改 lv_conf.h
#define LV_USE_LINUX_FBDEV  1
#define LV_LINUX_FBDEV_BSD  1
#define LV_USE_SDL          0  // 禁用 SDL

// 2. 配置显示驱动
lv_disp_drv_t disp_drv;
lv_disp_drv_init(&disp_drv);
disp_drv.flush_cb = fbdev_flush;  // 帧缓冲刷新
disp_drv.hor_res = 1280;
disp_drv.ver_res = 800;
lv_disp_drv_register(&disp_drv);

// 3. 配置输入设备 (触摸屏)
lv_indev_drv_t indev_drv;
lv_indev_drv_init(&indev_drv);
indev_drv.type = LV_INDEV_TYPE_POINTER;
indev_drv.read_cb = evdev_read;  // 读取 /dev/input/event0
lv_indev_drv_register(&indev_drv);
```

#### 创建第一个界面

```c
// home_page.c
void create_home_page() {
    lv_obj_t* scr = lv_scr_act();
    
    // 背景
    lv_obj_set_style_bg_color(scr, lv_color_hex(0x1a1a2e), 0);
    
    // 标题
    lv_obj_t* title = lv_label_create(scr);
    lv_label_set_text(title, "Echo-Mate");
    lv_obj_set_style_text_font(title, &lv_font_montserrat_28, 0);
    lv_obj_set_style_text_color(title, lv_color_hex(0xeeeeee), 0);
    lv_obj_align(title, LV_ALIGN_TOP_MID, 0, 20);
    
    // 时间显示
    lv_obj_t* time_label = lv_label_create(scr);
    lv_obj_set_style_text_font(time_label, &lv_font_montserrat_48, 0);
    lv_obj_align(time_label, LV_ALIGN_CENTER, 0, -50);
    
    // 更新时间的任务
    lv_timer_create(update_time, 1000, time_label);
    
    // 语音按钮
    lv_obj_t* mic_btn = lv_btn_create(scr);
    lv_obj_set_size(mic_btn, 120, 120);
    lv_obj_align(mic_btn, LV_ALIGN_BOTTOM_MID, 0, -40);
    lv_obj_set_style_bg_color(mic_btn, lv_color_hex(0xe94560), 0);
    lv_obj_set_style_radius(mic_btn, LV_RADIUS_CIRCLE, 0);
    
    lv_obj_t* mic_icon = lv_label_create(mic_btn);
    lv_label_set_text(mic_icon, LV_SYMBOL_AUDIO);
    lv_obj_set_style_text_font(mic_icon, &lv_font_montserrat_32, 0);
    lv_obj_center(mic_icon);
    
    lv_obj_add_event_cb(mic_btn, mic_btn_handler, LV_EVENT_CLICKED, NULL);
}
```

### Week 16: 完整移植与优化

#### 移植检查清单

- [ ] DeskBot_demo 交叉编译成功
- [ ] LVGL 在 RK3562 上显示正常
- [ ] WebSocket 客户端连接服务端
- [ ] 音频输入输出正常
- [ ] YOLOv5 目标检测运行
- [ ] AI Chat Pipeline 工作
- [ ] 触摸屏响应
- [ ] 长时间稳定性测试 (>24h)

#### 性能优化

```bash
# 1. 启动性能分析
cd /opt/deskbot
./main &

# 监控资源使用
top
free -h
cat /sys/class/thermal/thermal_zone0/temp  # CPU 温度

# 2. 优化建议
# - 减少 LVGL 动画复杂度
# - 降低 YOLO 输入分辨率 (640→480)
# - 使用 INT8 量化代替 FP16
# - 调整音频缓冲区大小
```

### ✅ Phase 6 检查点

- [ ] LVGL 界面在 RK3562 上运行
- [ ] 完整 AI Pipeline 工作
- [ ] 系统稳定运行 > 24 小时
- [ ] 性能达到预期 (FPS > 15, 延迟 < 500ms)

---

## 📚 学习资源

### 官方文档

| 文档 | 链接 | 说明 |
|------|------|------|
| RK3562 Datasheet | [PDF](https://www.armdesigner.com/...) | 完整硬件规格 |
| Buildroot 手册 | [官网](https://buildroot.org/manual.html) | Buildroot 详细文档 |
| RKNN 用户手册 | [GitHub](https://github.com/rockchip-linux/rknpu2) | NPU 开发指南 |
| LVGL 文档 | [官网](https://docs.lvgl.io/9.2/) | GUI 开发 |
| Linux Device Drivers | [LDD3](https://static.lwn.net/images/pdf/LDD3/) | 驱动开发圣经 |

### 推荐书籍

1. 《嵌入式Linux系统开发》
2. 《Linux设备驱动开发详解》
3. 《ARM Cortex-A53 技术参考手册》
4. 《深度学习模型压缩与部署》

### 在线课程

- [Buildroot 培训](https://bootlin.com/training/buildroot/)
- [Linux 驱动开发](https://linuxfoundation.org/...)
- [RKNN 模型部署](https://github.com/rockchip-linux/rknpu2/tree/master/examples)

---

## 🎯 未来 RV1106v3 项目准备

### RK3562 学习对 RV1106v3 的价值

| RK3562 经验 | RV1106v3 应用 |
|------------|---------------|
| Buildroot 配置 | 直接适用，仅需换 defconfig |
| 设备树修改 | 概念相同，引脚不同 |
| RKNN 模型转换 | 相同流程，仅 target_platform 不同 |
| C++ 推理代码 | 100% 兼容，重新编译即可 |
| LVGL 开发 | 相同代码，仅需改显示驱动 |
| AI Pipeline 架构 | 逻辑相同，性能调整 |

### 迁移时需注意的差异

```c
// RK3562 -> RV1106v3 差异
// 1. 架构: aarch64 -> arm32
//    工具链: aarch64-linux-gnu -> arm-linux-gnueabihf

// 2. NPU: 1 TOPS -> 0.5 TOPS
//    量化: FP16 -> INT8 (必须量化)

// 3. 内存: 8GB -> 2Gb
//    模型大小: <200MB -> <50MB

// 4. CPU: 4核 -> 1核
//    并行: 多线程 -> 串行/协程

// 5. 显示: GPU -> 2D引擎
//    动画: 硬件加速 -> CPU渲染
```

---

## 📅 完整时间规划

| 阶段 | 周数 | 核心任务 | 产出物 |
|------|------|----------|--------|
| Phase 1 | 1-2 | 环境搭建 | 可烧录系统 |
| Phase 2 | 3-5 | 系统/驱动 | Hello World 驱动 |
| Phase 3 | 6-8 | 多媒体 | 摄像头+显示工作 |
| Phase 4 | 9-11 | AI/NPU | YOLOv5 部署 |
| Phase 5 | 12-14 | 通信 | AI Pipeline |
| Phase 6 | 15-16 | 集成 | 完整项目 |

---

## ✅ 最终检查清单

### 技能掌握
- [ ] 熟练交叉编译 (aarch64)
- [ ] 理解设备树配置
- [ ] 能编写简单驱动
- [ ] 掌握 RKNN 模型转换
- [ ] 熟悉 ALSA 音频编程
- [ ] 熟练使用 LVGL
- [ ] 理解完整 AI Pipeline

### 项目产出
- [ ] 自定义 Buildroot 系统
- [ ] 点亮的 LVGL 界面
- [ ] 运行的 YOLOv5 检测
- [ ] 完整的 AI Chat 流程
- [ ] 移植的 Echo-Mate 项目

### 文档产出
- [ ] 学习笔记
- [ ] 问题解决方案记录
- [ ] 性能测试报告
- [ ] 移植经验总结

---

**文档版本**: v1.0  
**最后更新**: 2026-04-14  
**适用平台**: RK3562  
**目标项目**: Echo-Mate (RV1106v3)