---
{"publish":true,"aliases":"localllm Local LLM local-llm-tools","title":"Local LLM Tools","enableToc":true,"created":"2025-11-16T00:22:05.146+02:00","modified":"2025-11-16T00:26:46.885+02:00","tags":["ai-agents","localllm","free","open-source"],"cssclasses":""}
---


## Overview

Free and high-quality local LLM tools and agents that run on your own hardware without requiring cloud services or API keys.

## Tools & Frameworks

### [[ai-agents/localllm/ollama/index\|Ollama]]
- **Description**: Easy-to-use local LLM runner supporting multiple models
- **Models**: Llama 3, Mistral, Phi, CodeLlama, and more
- **Platform**: Windows, macOS, Linux
- **License**: Open source

### [[ai-agents/localllm/lm-studio/index\|LM Studio]]
- **Description**: Desktop app for running local LLMs with a user-friendly interface
- **Models**: Support for GGUF format models
- **Platform**: Windows, macOS, Linux
- **License**: Free for personal use

### [[ai-agents/localllm/jan/index\|Jan]]
- **Description**: Open-source ChatGPT alternative that runs 100% offline
- **Models**: Multiple model support including Llama, Mistral
- **Platform**: Windows, macOS, Linux
- **License**: Open source (AGPLv3)

### [[ai-agents/localllm/gpt4all/index\|GPT4All]]
- **Description**: Free-to-use, locally running, privacy-aware chatbot
- **Models**: Multiple open-source models optimized for consumer hardware
- **Platform**: Windows, macOS, Linux
- **License**: Open source

### [[ai-agents/localllm/localai/index\|LocalAI]]
- **Description**: OpenAI-compatible API for local inference
- **Models**: Support for various model formats (GGML, GGUF)
- **Platform**: Cross-platform (via Docker)
- **License**: Open source (MIT)

### [[ai-agents/localllm/llama-cpp/index\|Llama.cpp]]
- **Description**: C/C++ implementation for running Llama models locally
- **Models**: Llama family and compatible models
- **Platform**: Cross-platform
- **License**: Open source (MIT)

### [[ai-agents/localllm/text-generation-webui/index\|Text Generation WebUI (oobabooga)]]
- **Description**: Gradio web UI for running LLMs
- **Models**: Wide model support including GPTQ, AWQ, GGUF
- **Platform**: Cross-platform (Python)
- **License**: Open source (AGPLv3)

## Recommended Models

### For Coding
- **CodeLlama** (7B, 13B, 34B)
- **DeepSeek Coder** (6.7B, 33B)
- **Phind CodeLlama** (34B)
- **WizardCoder**

### For General Purpose
- **Llama 3** (8B, 70B)
- **Mistral** (7B)
- **Mixtral** (8x7B)
- **Phi-3** (3.8B)

### For Chat
- **Llama 3 Instruct**
- **Mistral Instruct**
- **Zephyr**
- **OpenChat**

## Hardware Requirements

### Minimum (7B models)
- RAM: 8GB+
- GPU: Optional (CPU inference possible)
- Storage: 5-10GB per model

### Recommended (13B-34B models)
- RAM: 16GB+
- GPU: 8GB VRAM+ (NVIDIA recommended)
- Storage: 10-30GB per model

### Optimal (70B+ models)
- RAM: 32GB+
- GPU: 24GB VRAM+ or multiple GPUs
- Storage: 40-80GB per model

## Key Features

- **Privacy**: All processing happens locally
- **No API Costs**: Free to use after initial setup
- **Offline Capable**: No internet connection required
- **Customizable**: Full control over model selection and parameters
- **Open Source**: Most tools are fully open source
