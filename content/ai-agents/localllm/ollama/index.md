---
{"publish":true,"aliases":"Ollama Ollama","title":"Ollama","enableToc":true,"created":"2025-11-16T00:23:05.440+02:00","modified":"2025-11-16T15:51:14.993+02:00","tags":["ai-agents","localllm","ollama","local-llm"],"cssclasses":""}
---


## Overview

Ollama is an easy-to-use local LLM runner that supports multiple open-source models including Llama 3, Mistral, Phi, CodeLlama, and more.

## Key Features

- **Simple Installation**: Single command installation
- **Model Library**: Access to hundreds of models via registry
- **Cross-Platform**: Windows, macOS, Linux support
- **REST API**: Built-in API for integration
- **GPU Acceleration**: Automatic GPU detection and utilization

## Installation

### macOS
```bash
brew install ollama
```

### Linux/Windows
Download from [ollama.ai](https://ollama.ai)

## Usage

### Pull and Run a Model
```bash
# Pull a model
ollama pull llama3

# Run the model
ollama run llama3
```

### List Available Models
```bash
ollama list
```

## Popular Models

- **llama3** (8B, 70B) - Meta's latest Llama model
- **mistral** (7B) - Fast and capable model
- **codellama** (7B, 13B, 34B) - Code-focused model
- **phi3** (3.8B) - Microsoft's efficient model

## Integration

Ollama provides a REST API at `http://localhost:11434` for easy integration with other tools and applications.

## Related
- [[./localllm/index\|Local Llm Tools]]
- [[./ai-agents\|Ai Agents]]
