---
{"publish":true,"aliases":"localai LocalAI","title":"LocalAI","enableToc":true,"created":"2025-11-16T00:23:05.466+02:00","modified":"2025-11-16T00:26:46.885+02:00","tags":["ai-agents","localllm","localai","local-llm"],"cssclasses":""}
---


## Overview

LocalAI is an OpenAI-compatible API for local inference, providing a drop-in replacement for OpenAI's API that runs on your hardware.

## Key Features

- **OpenAI Compatible**: Drop-in replacement for OpenAI API
- **Multiple Formats**: Support for GGML, GGUF model formats
- **Docker Support**: Easy deployment via Docker
- **REST API**: Standard REST endpoints
- **Cross-Platform**: Runs on any system with Docker

## Installation

### Docker (Recommended)
```bash
docker run -p 8080:8080 localai/localai:latest
```

### From Source
```bash
git clone https://github.com/go-skynet/LocalAI
cd LocalAI
make build
```

## Usage

1. **Start Server**: Run LocalAI server
2. **Use OpenAI SDK**: Use standard OpenAI Python/Node.js SDKs
3. **Point to Local**: Set base_url to `http://localhost:8080`

## Supported Models

- Llama models
- GPT-2/J
- BERT
- Stable Diffusion (images)
- And more

## Related
- [[ai-agents/localllm/index\|Local LLM Tools]]
- [[ai-agents\|AI Agents]]
