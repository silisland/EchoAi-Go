# Echo-Mate AI Desktop Robot - Project Knowledge Base

**Project**: Echo-Mate AI Desktop Robot  
**Platform**: RV1106 (Cortex A7 + 1TOPS NPU)  
**Framework**: LVGL v9.2 + CMake + Buildroot Linux  
**Team**: 4-person startup competition entry

## OVERVIEW

Embedded AI hardware project combining voice interaction, object detection, and LLM capabilities. The system uses a client-server architecture with C++ client on RV1106 board and Python server handling AI model pipelines.

**Core Features:**
- Voice Assistant: Real-time ASR/TTS using SenseVoice + CosyVoice
- Object Detection: YOLOv5 with RKNN acceleration on NPU
- GUI: LVGL-based desktop assistant interface
- Communication: WebSocket + JSON + Opus audio streaming

## PROJECT STRUCTURE

```
echoAI/
├── deepwiki/              # Technical documentation from DeepWiki
│   ├── Overview.md        # Project overview and architecture
│   ├── Getting-Started.md # Build and setup instructions
│   ├── Server-Architecture.md    # Python server design
│   ├── Communication-Protocol.md  # WebSocket protocol specs
│   ├── Configuration-Reference.md
│   ├── External-Dependencies-and-Submodules.md
│   ├── DeskBot-Demo---AI-Desktop-Assistant.md
│   ├── AIChat-Demo---Voice-Assistant.md
│   ├── YOLOv5-Demo---Object-Detection.md
│   └── Object-Detection-Integration.md
├── AGENTS.md            # This file
└── [Project proposal docs]
```

## WHERE TO LOOK

| Task | Location | Notes |
|------|----------|-------|
| Quick project intro | `deepwiki/Overview.md` | High-level architecture |
| First-time setup | `deepwiki/Getting-Started.md` | Build instructions, deps |
| Server design | `deepwiki/Server-Architecture.md` | Python backend, threading |
| Protocol specs | `deepwiki/Communication-Protocol.md` | WebSocket, JSON, binary audio |
| System config | `deepwiki/Configuration-Reference.md` | LVGL, CMake, runtime params |
| Dependencies | `deepwiki/External-Dependencies-and-Submodules.md` | Git submodules, APIs |
| DeskBot GUI | `deepwiki/DeskBot-Demo---AI-Desktop-Assistant.md` | Main integrated app |
| Voice assistant | `deepwiki/AIChat-Demo---Voice-Assistant.md` | Client-server voice AI |
| Object detection | `deepwiki/YOLOv5-Demo---Object-Detection.md` | YOLOv5 + RKNN |
| YOLO integration | `deepwiki/Object-Detection-Integration.md` | C interface layer |

## TECHNOLOGY STACK

| Component | Technology | Purpose |
|-----------|------------|---------|
| GUI Framework | LVGL v9.2 | Embedded GUI library |
| Build System | CMake | Cross-compilation |
| Audio | PortAudio + Opus | Audio I/O + compression |
| Communication | WebSocket + JSON | Client-server protocol |
| AI ASR | SenseVoice | Speech recognition |
| AI VAD | FSMN-VAD | Voice activity detection |
| AI TTS | CosyVoice | Text-to-speech |
| AI Chat | Tongyi Qianwen | Alibaba Cloud LLM |
| CV | YOLOv5 + RKNN | Object detection on NPU |
| Cloud APIs | Alibaba Cloud | TTS, ASR, chat services |

## BUILD CONFIGURATION

**Hardware Target (ARM):**
```bash
# In DeskBot_demo/conf/dev_conf.h
#define LV_USE_SIMULATOR 0

# Build
cd DeskBot_demo/build
cmake .. -DTARGET_ARM=ON
make
```

**Simulator (SDL):**
```bash
# In DeskBot_demo/conf/dev_conf.h
#define LV_USE_SIMULATOR 1

# Build
cd DeskBot_demo/build
cmake ..
make
```

## KEY CONFIGURATION FILES

| File | Purpose | Location |
|------|---------|----------|
| `system_para.conf` | Runtime parameters (API keys, network) | `DeskBot_demo/utils/` |
| `dev_conf.h` | Build target selector | `DeskBot_demo/conf/` |
| `lv_conf.h` | LVGL configuration | `DeskBot_demo/` |
| `toolchain.cmake` | Cross-compilation toolchain | `yolov5_demo/cpp/` |

## AI MODEL PIPELINE

```
Audio Input → VAD (FSMN-VAD) → ASR (SenseVoice) → Intent (FastText)
                                                    ↓
                                              Chat (Tongyi Qianwen)
                                                    ↓
                                              TTS (CosyVoice)
                                                    ↓
                                           Audio Output (Opus)
```

## GIT SUBMODULES

| Submodule | Path | Purpose |
|-----------|------|---------|
| SenseVoice | `AIChat_demo/Server/models/FunAudioLLM/SenseVoice` | ASR model |
| SenseVoiceSmall | `AIChat_demo/Server/models/FunAudioLLM/iic/SenseVoiceSmall` | Lightweight ASR |
| FSMN-VAD | `AIChat_demo/Server/models/FunAudioLLM/iic/speech_fsmn_vad_*` | VAD model |
| LVGL | `DeskBot_demo/lvgl` | GUI framework v9.2 |
| rkmpi_demos | `rkmpi_demos` | Rockchip media examples |

## TEAM SKILLS

- **Languages**: C/C++, Python
- **Hardware**: STM32, RV1106 embedded platform
- **AI/ML**: Machine vision, object detection, speech processing
- **Tools**: Git, Markdown, CMake
- **Deployment**: WSL2, cloud servers (OpenClaw, AstrBot experience)

## CONVENTIONS

- DeepWiki docs are auto-generated from codebase analysis
- Demo naming: `Demo-Name---Feature` format
- File naming: snake_case for C/C++, camelCase for integration
- Build targets: Use `TARGET_ARM` flag for cross-compilation

## ANTI-PATTERNS

- **DO NOT** treat DeepWiki docs as up-to-date reference - verify against actual code
- **DO NOT** use docs for hardware setup - check actual schematics
- **DO NOT** rely on docs for build instructions - verify in `code/` submodules
- **DO NOT** commit API keys to version control
- **DO NOT** skip submodule initialization: `git submodule update --init --recursive`

## DEPLOYMENT NOTES

- Default network: Echo board `172.32.0.93`, Server `172.32.0.100:8000`
- Server token must match in `system_para.conf` and server startup args
- Requires Python 3.10 environment for AI server
- RKNN models must be converted for RV1106 NPU

## EXTERNAL SERVICES

| Service | API | Purpose |
|---------|-----|---------|
| Alibaba Cloud | dashscope.aliyuncs.com | TTS, ASR, Chat |
| Gaode Maps | restapi.amap.com | Weather data |
| NTP | Various servers | Time synchronization |

## COMMANDS

```bash
# Initialize submodules
git submodule update --init --recursive

# Start AI server
cd AIChat_demo/Server
python main.py --access_token="123456" --aliyun_api_key="sk-xxx"

# Build DeskBot (simulator)
cd DeskBot_demo && mkdir build && cd build
cmake .. && make

# Run simulator
../bin/main
```

## REFERENCES

- GitHub: No-Chicken/Demo4Echo
- Hardware: RV1106 (Cortex A7 + 1TOPS NPU)
- Project URL: https://oshwhub.com/no_chicken/ai-desktop-robot-echo
