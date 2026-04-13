# Server Architecture

> **Relevant source files**
> * [AIChat_demo/Client/README.md](https://github.com/No-Chicken/Demo4Echo/blob/80ef46db/AIChat_demo/Client/README.md?plain=1)
> * [AIChat_demo/Server/README.md](https://github.com/No-Chicken/Demo4Echo/blob/80ef46db/AIChat_demo/Server/README.md?plain=1)
> * [AIChat_demo/Server/main.py](https://github.com/No-Chicken/Demo4Echo/blob/80ef46db/AIChat_demo/Server/main.py)
> * [AIChat_demo/assets/AIchat_states.png](https://github.com/No-Chicken/Demo4Echo/blob/80ef46db/AIChat_demo/assets/AIchat_states.png)
> * [assets/AIChatDiagram.png](https://github.com/No-Chicken/Demo4Echo/blob/80ef46db/assets/AIChatDiagram.png)
> * [assets/AIchat_states.png](https://github.com/No-Chicken/Demo4Echo/blob/80ef46db/assets/AIchat_states.png)

This document details the Python server implementation of the AIChat demo's voice assistant system. The server provides WebSocket-based communication, manages AI service pipelines, and handles real-time audio processing through a multi-threaded architecture.

For information about the client-side C++ implementation, see [Client Architecture](/No-Chicken/Demo4Echo/5.1-client-architecture). For details about the communication protocol between client and server, see [Communication Protocol](/No-Chicken/Demo4Echo/5.3-communication-protocol).

## Overview

The AIChat server is a Python-based WebSocket server that orchestrates multiple AI services to provide real-time voice interaction capabilities. The architecture follows a service-oriented design with dedicated threads for different processing pipelines and a central service manager for coordination.

**Core Server Components**

```

```

Sources: [AIChat_demo/Server/main.py L1-L50](https://github.com/No-Chicken/Demo4Echo/blob/80ef46db/AIChat_demo/Server/main.py#L1-L50)

 [AIChat_demo/Server/README.md L27-L47](https://github.com/No-Chicken/Demo4Echo/blob/80ef46db/AIChat_demo/Server/README.md?plain=1#L27-L47)

## Service Management Layer

The `ServiceManager` class serves as the central orchestrator for all AI services, providing a unified interface for VAD (Voice Activity Detection), ASR (Automatic Speech Recognition), chat processing, intent recognition, and TTS (Text-to-Speech) services.

**ServiceManager Architecture**

```

```

Sources: [AIChat_demo/Server/main.py L12-L13](https://github.com/No-Chicken/Demo4Echo/blob/80ef46db/AIChat_demo/Server/main.py#L12-L13)

 [AIChat_demo/Server/README.md L36-L37](https://github.com/No-Chicken/Demo4Echo/blob/80ef46db/AIChat_demo/Server/README.md?plain=1#L36-L37)

## Threading Architecture

The server implements a multi-threaded architecture to handle concurrent audio processing, TTS generation, and WebSocket communication without blocking operations.

**Thread Communication Model**

```

```

The threading model uses `stop_event` from `ServiceManager` to coordinate graceful shutdown across all threads. The main thread runs an asyncio event loop for WebSocket handling, while dedicated threads manage TTS generation and audio transmission.

Sources: [AIChat_demo/Server/main.py L15-L21](https://github.com/No-Chicken/Demo4Echo/blob/80ef46db/AIChat_demo/Server/main.py#L15-L21)

 [AIChat_demo/Server/main.py L31-L34](https://github.com/No-Chicken/Demo4Echo/blob/80ef46db/AIChat_demo/Server/main.py#L31-L34)

## WebSocket Server Implementation

The `WebSocketServer` class in `ws_server.py` manages client connections and routes incoming messages to appropriate handlers based on message type and content.

**Message Handling Pipeline**

```

```

The server implements the binary protocol specified in the client documentation, handling both JSON messages and binary audio data with a structured header format.

Sources: [AIChat_demo/Server/main.py

24](https://github.com/No-Chicken/Demo4Echo/blob/80ef46db/AIChat_demo/Server/main.py#L24-L24)

 [AIChat_demo/Server/README.md L32-L35](https://github.com/No-Chicken/Demo4Echo/blob/80ef46db/AIChat_demo/Server/README.md?plain=1#L32-L35)

 [AIChat_demo/Client/README.md L128-L137](https://github.com/No-Chicken/Demo4Echo/blob/80ef46db/AIChat_demo/Client/README.md?plain=1#L128-L137)

## Request Processing Pipeline

The server processes incoming requests through a multi-stage pipeline that transforms audio input into intelligent responses.

**Audio Processing Flow**

```

```

Sources: [AIChat_demo/Server/README.md L63-L100](https://github.com/No-Chicken/Demo4Echo/blob/80ef46db/AIChat_demo/Server/README.md?plain=1#L63-L100)

 [AIChat_demo/Client/README.md L92-L176](https://github.com/No-Chicken/Demo4Echo/blob/80ef46db/AIChat_demo/Client/README.md?plain=1#L92-L176)

## AI Model Integration

The server integrates multiple AI models through service abstractions, allowing for flexible model swapping and configuration.

**Model Service Architecture**

```

```

The service layer abstracts model implementations, enabling the system to support both local models (VAD, ASR, Intent) and cloud-based services (Chat, TTS) through a unified interface.

Sources: [AIChat_demo/Server/README.md L21-L24](https://github.com/No-Chicken/Demo4Echo/blob/80ef46db/AIChat_demo/Server/README.md?plain=1#L21-L24)

 [AIChat_demo/Server/main.py L12-L13](https://github.com/No-Chicken/Demo4Echo/blob/80ef46db/AIChat_demo/Server/main.py#L12-L13)

## Configuration and Startup

The server startup process initializes all services and threads in a coordinated manner, with proper error handling and graceful shutdown capabilities.

| Component | Initialization Order | Purpose |
| --- | --- | --- |
| `ServiceManager` | 1 | Initialize AI services (VAD, ASR, Chat, Intent, TTS) |
| `TTSGenerateThread` | 2 | Start TTS generation pipeline |
| `AudioSendThread` | 3 | Start audio transmission pipeline |
| `WebSocketServer` | 4 | Start WebSocket listener on port 8000 |

The server accepts command-line parameters for `access_token` and `aliyun_api_key` configuration, with default values for development use.

**Shutdown Sequence**

```

```

Sources: [AIChat_demo/Server/main.py L10-L35](https://github.com/No-Chicken/Demo4Echo/blob/80ef46db/AIChat_demo/Server/main.py#L10-L35)

 [AIChat_demo/Server/main.py L36-L49](https://github.com/No-Chicken/Demo4Echo/blob/80ef46db/AIChat_demo/Server/main.py#L36-L49)