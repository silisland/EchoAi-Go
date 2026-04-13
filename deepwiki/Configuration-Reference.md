# Configuration Reference

> **Relevant source files**
> * [AIChat_demo/Client/CMakeLists.txt](https://github.com/No-Chicken/Demo4Echo/blob/80ef46db/AIChat_demo/Client/CMakeLists.txt)
> * [DeskBot_demo/conf/dev_conf.h](https://github.com/No-Chicken/Demo4Echo/blob/80ef46db/DeskBot_demo/conf/dev_conf.h)
> * [DeskBot_demo/lv_conf.h](https://github.com/No-Chicken/Demo4Echo/blob/80ef46db/DeskBot_demo/lv_conf.h)
> * [DeskBot_demo/utils/system_para.conf](https://github.com/No-Chicken/Demo4Echo/blob/80ef46db/DeskBot_demo/utils/system_para.conf)
> * [yolov5_demo/cpp/AIcamera_c_interface.h](https://github.com/No-Chicken/Demo4Echo/blob/80ef46db/yolov5_demo/cpp/AIcamera_c_interface.h)
> * [yolov5_demo/cpp/CMakeLists.txt](https://github.com/No-Chicken/Demo4Echo/blob/80ef46db/yolov5_demo/cpp/CMakeLists.txt)
> * [yolov5_demo/cpp/main.cc](https://github.com/No-Chicken/Demo4Echo/blob/80ef46db/yolov5_demo/cpp/main.cc)
> * [yolov5_demo/cpp/main2.cc](https://github.com/No-Chicken/Demo4Echo/blob/80ef46db/yolov5_demo/cpp/main2.cc)
> * [yolov5_demo/cpp/rv1106_yolov5_demo/c_face_test](https://github.com/No-Chicken/Demo4Echo/blob/80ef46db/yolov5_demo/cpp/rv1106_yolov5_demo/c_face_test)
> * [yolov5_demo/cpp/rv1106_yolov5_demo/rknn_yolov5_demo](https://github.com/No-Chicken/Demo4Echo/blob/80ef46db/yolov5_demo/cpp/rv1106_yolov5_demo/rknn_yolov5_demo)
> * [yolov5_demo/cpp/toolchain.cmake](https://github.com/No-Chicken/Demo4Echo/blob/80ef46db/yolov5_demo/cpp/toolchain.cmake)

This document provides a comprehensive reference for all configuration files, build options, and system parameters used across the Echo development board demo repository. It covers runtime system parameters, GUI framework settings, build configurations, and cross-compilation toolchain setup.

For information about the overall system architecture, see [Overview](/No-Chicken/Demo4Echo/1-overview). For specific application configuration within individual demos, see [DeskBot Demo](/No-Chicken/Demo4Echo/4-deskbot-demo-ai-desktop-assistant) and [AIChat Demo](/No-Chicken/Demo4Echo/5-aichat-demo-voice-assistant).

## System Runtime Configuration

The primary runtime configuration is managed through `system_para.conf`, which contains key-value pairs for system-wide settings that persist across application restarts.

### Core System Parameters

The [DeskBot_demo/utils/system_para.conf L1-L23](https://github.com/No-Chicken/Demo4Echo/blob/80ef46db/DeskBot_demo/utils/system_para.conf#L1-L23)

 file defines essential system state:

| Parameter | Type | Description | Default Value |
| --- | --- | --- | --- |
| `year`, `month`, `day`, `hour`, `minute` | Integer | System date and time | 2025-01-01 00:00 |
| `brightness` | Integer (0-100) | Display brightness level | 50 |
| `sound` | Integer (0-100) | Audio volume level | 50 |
| `wifi_connected` | Boolean | WiFi connection status | false |
| `auto_time` | Boolean | Automatic time synchronization | true |
| `auto_location` | Boolean | Automatic location detection | false |

### Location and API Configuration

Geographic and external service settings:

| Parameter | Purpose |
| --- | --- |
| `city` | Current city name (东城区) |
| `adcode` | Administrative division code (110101) |
| `gaode_api_key` | Gaode Maps API key for weather services |
| `aliyun_api_key` | Alibaba Cloud API key for AI services |

### AIChat Integration Parameters

Voice assistant communication settings:

```
AIChat_server_url=172.32.0.100
AIChat_server_port=8000
AIChat_server_token=123456
AIChat_Client_ID=00:11:22:33:44:55
AIChat_protocol_version=2
AIChat_sample_rate=16000
AIChat_channels=1
AIChat_frame_duration=40
```

**Configuration Loading Flow**

```

```

Sources: [DeskBot_demo/utils/system_para.conf L1-L23](https://github.com/No-Chicken/Demo4Echo/blob/80ef46db/DeskBot_demo/utils/system_para.conf#L1-L23)

## LVGL GUI Framework Configuration

The LVGL configuration in `lv_conf.h` controls all aspects of the graphical user interface framework used by DeskBot demo.

### Display and Rendering Configuration

Core display settings from [DeskBot_demo/lv_conf.h L32-L83](https://github.com/No-Chicken/Demo4Echo/blob/80ef46db/DeskBot_demo/lv_conf.h#L32-L83)

:

| Setting | Value | Purpose |
| --- | --- | --- |
| `LV_COLOR_DEPTH` | 16 | RGB565 color format |
| `LV_MEM_SIZE` | 1024 * 1024 | 1MB memory pool for LVGL |
| `LV_DEF_REFR_PERIOD` | 20 | 50 FPS refresh rate |
| `LV_DPI_DEF` | 130 | Dots per inch for sizing |

### Operating System Integration

The configuration disables OS-specific features for bare-metal operation:

```

```

### Hardware Abstraction Layer

Platform-specific display drivers configured in [DeskBot_demo/lv_conf.h L997-L1003](https://github.com/No-Chicken/Demo4Echo/blob/80ef46db/DeskBot_demo/lv_conf.h#L997-L1003)

:

```

```

### Widget and Feature Configuration

Extensive widget library settings [DeskBot_demo/lv_conf.h L587-L677](https://github.com/No-Chicken/Demo4Echo/blob/80ef46db/DeskBot_demo/lv_conf.h#L587-L677)

:

| Widget Category | Status | Key Widgets |
| --- | --- | --- |
| Basic Widgets | Enabled | Button, Label, Image, Canvas |
| Input Widgets | Enabled | Keyboard, Textarea, Slider |
| Display Widgets | Enabled | Chart, Calendar, Table |
| Layout Systems | Enabled | Flex, Grid layouts |

**LVGL Configuration Architecture**

```

```

Sources: [DeskBot_demo/lv_conf.h L1-L1130](https://github.com/No-Chicken/Demo4Echo/blob/80ef46db/DeskBot_demo/lv_conf.h#L1-L1130)

 [DeskBot_demo/conf/dev_conf.h L1-L24](https://github.com/No-Chicken/Demo4Echo/blob/80ef46db/DeskBot_demo/conf/dev_conf.h#L1-L24)

## Build Configuration System

The repository uses CMake for build configuration across multiple demos, each with specific requirements and dependencies.

### YOLOv5 Demo Build Configuration

The [yolov5_demo/cpp/CMakeLists.txt L1-L100](https://github.com/No-Chicken/Demo4Echo/blob/80ef46db/yolov5_demo/cpp/CMakeLists.txt#L1-L100)

 defines object detection demo build:

**Toolchain and Target Setup**

```

```

**Library Dependencies and Linking**

| Library | Purpose | Link Target |
| --- | --- | --- |
| `librknnmrt.so` | RKNN runtime for AI inference | Static linking |
| OpenCV 4 | Computer vision operations | Dynamic linking |
| Custom utilities | Image processing, file I/O | Static libraries |

### AIChat Demo Build Configuration

The [AIChat_demo/Client/CMakeLists.txt L1-L108](https://github.com/No-Chicken/Demo4Echo/blob/80ef46db/AIChat_demo/Client/CMakeLists.txt#L1-L108)

 configures voice assistant client:

**Cross-compilation Support**

```

```

**Audio and Communication Dependencies**

```

```

**Build Configuration Flow**

```

```

Sources: [yolov5_demo/cpp/CMakeLists.txt L1-L100](https://github.com/No-Chicken/Demo4Echo/blob/80ef46db/yolov5_demo/cpp/CMakeLists.txt#L1-L100)

 [AIChat_demo/Client/CMakeLists.txt L1-L108](https://github.com/No-Chicken/Demo4Echo/blob/80ef46db/AIChat_demo/Client/CMakeLists.txt#L1-L108)

## Development Configuration

The development configuration system allows switching between simulator and hardware targets.

### Hardware vs Simulator Configuration

The [DeskBot_demo/conf/dev_conf.h L8-L18](https://github.com/No-Chicken/Demo4Echo/blob/80ef46db/DeskBot_demo/conf/dev_conf.h#L8-L18)

 file controls build targets:

```

```

**Development Target Matrix**

| Target | Framebuffer | Input | Display Backend | Use Case |
| --- | --- | --- | --- | --- |
| Hardware (`LV_USE_SIMULATOR=0`) | Linux `/dev/fb0` | evdev | Direct hardware | Production deployment |
| Simulator (`LV_USE_SIMULATOR=1`) | SDL window | SDL events | SDL rendering | Development/testing |

Sources: [DeskBot_demo/conf/dev_conf.h L1-L24](https://github.com/No-Chicken/Demo4Echo/blob/80ef46db/DeskBot_demo/conf/dev_conf.h#L1-L24)

## Cross-compilation Toolchain Configuration

The toolchain configuration enables building ARM binaries on x86 development machines.

### Toolchain Setup

The [yolov5_demo/cpp/toolchain.cmake L1-L19](https://github.com/No-Chicken/Demo4Echo/blob/80ef46db/yolov5_demo/cpp/toolchain.cmake#L1-L19)

 defines cross-compilation environment:

```

```

### Cross-compilation Path Resolution

| Component | Path Resolution Mode |
| --- | --- |
| Programs (`CMAKE_FIND_ROOT_PATH_MODE_PROGRAM`) | `NEVER` - Use host tools |
| Libraries (`CMAKE_FIND_ROOT_PATH_MODE_LIBRARY`) | `ONLY` - Search only in sysroot |
| Headers (`CMAKE_FIND_ROOT_PATH_MODE_INCLUDE`) | `ONLY` - Search only in sysroot |

**Cross-compilation Toolchain Flow**

```

```

Sources: [yolov5_demo/cpp/toolchain.cmake L1-L19](https://github.com/No-Chicken/Demo4Echo/blob/80ef46db/yolov5_demo/cpp/toolchain.cmake#L1-L19)

## Configuration Management Patterns

The system employs several patterns for configuration management across different subsystems.

### Configuration File Hierarchy

```

```

### Configuration Loading Mechanisms

| Configuration Type | Loading Method | Modification Method | Persistence |
| --- | --- | --- | --- |
| Runtime parameters | File parsing at startup | Direct file write | `system_para.conf` |
| LVGL settings | Compile-time inclusion | Source code edit | Header files |
| Build configuration | CMake generation | CMakeLists.txt edit | Build files |
| Development target | Preprocessor macros | Header file edit | Source code |

Sources: [DeskBot_demo/utils/system_para.conf L1-L23](https://github.com/No-Chicken/Demo4Echo/blob/80ef46db/DeskBot_demo/utils/system_para.conf#L1-L23)

 [DeskBot_demo/lv_conf.h L1-L1130](https://github.com/No-Chicken/Demo4Echo/blob/80ef46db/DeskBot_demo/lv_conf.h#L1-L1130)

 [DeskBot_demo/conf/dev_conf.h L1-L24](https://github.com/No-Chicken/Demo4Echo/blob/80ef46db/DeskBot_demo/conf/dev_conf.h#L1-L24)

 [yolov5_demo/cpp/CMakeLists.txt L1-L100](https://github.com/No-Chicken/Demo4Echo/blob/80ef46db/yolov5_demo/cpp/CMakeLists.txt#L1-L100)

 [AIChat_demo/Client/CMakeLists.txt L1-L108](https://github.com/No-Chicken/Demo4Echo/blob/80ef46db/AIChat_demo/Client/CMakeLists.txt#L1-L108)

 [yolov5_demo/cpp/toolchain.cmake L1-L19](https://github.com/No-Chicken/Demo4Echo/blob/80ef46db/yolov5_demo/cpp/toolchain.cmake#L1-L19)