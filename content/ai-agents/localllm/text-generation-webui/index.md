---
{"publish":true,"aliases":"Text-Generation-Webui Oobabooga Text-Gen-Webui","title":"Text Generation Webui","enableToc":true,"created":"2025-11-16T00:23:05.531+02:00","modified":"2025-11-16T15:31:33.355+02:00","tags":["ai-agents","localllm","text-generation-webui","oobabooga","local-llm"],"cssclasses":""}
---


## Overview

Text Generation WebUI (also known as oobabooga's web UI) is a Gradio web interface for running Large Language Models locally.

## Key Features

- **Web Interface**: User-friendly Gradio web UI
- **Wide Model Support**: GPTQ, AWQ, GGUF, and other formats
- **Extensions**: Plugin system for additional features
- **Chat Interface**: Built-in chat mode
- **API Support**: REST API for integrations
- **Cross-Platform**: Python-based, runs anywhere

## Installation

### One-Click Installer (Windows)
Download from [GitHub releases](https://github.com/oobabooga/text-generation-webui/releases)

### Manual Installation
```bash
git clone https://github.com/oobabooga/text-generation-webui
cd text-generation-webui
pip install -r requirements.txt
```

## Usage

1. **Download Models**: Place models in the `models` folder
2. **Start WebUI**: Run `start_windows.bat` (Windows) or `python server.py` (Linux/Mac)
3. **Access Interface**: Open browser to `http://localhost:7860`

## Supported Formats

- **GGUF**: Llama.cpp format
- **GPTQ**: Quantized models
- **AWQ**: Another quantization format
- **Transformers**: Hugging Face models
- **And more**

## Extensions

- **Whisper**: Speech-to-text
- **TTS**: Text-to-speech
- **API**: REST API access
- **Character Cards**: Role-playing support
- **And many more**

## Related
- [[ai-agents/localllm/index\|Local LLM Tools]]
- [[ai-agents\|AI Agents]]
