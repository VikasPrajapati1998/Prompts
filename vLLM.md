# vLLM: Easy, Fast, and Cheap LLM Serving

vLLM is an open-source library for large language model (LLM) inference and serving, originally developed at UC Berkeley's Sky Computing Lab. It provides high-throughput, memory-efficient serving with an OpenAI-compatible API.

## Overview

- **Purpose**: Fast and easy LLM inference and serving
- **License**: Open-source
- **Repository**: [github.com/vllm-project/vllm](https://github.com/vllm-project/vllm)
- **Documentation**: [docs.vllm.ai](https://docs.vllm.ai)
- **Community**: Over 2000 contributors from 200+ academic institutions and companies

## Key Features

### Performance & Efficiency
- **PagedAttention**: State-of-the-art memory management for attention key and value caches, enabling higher throughput
- **Continuous Batching**: Dynamically batches incoming requests for optimal GPU utilization
- **Chunked Prefill**: Processes long prompts efficiently alongside ongoing decoding
- **Prefix Caching**: Reuses KV cache for shared prefixes across requests
- **Optimized Kernels**: Supports FlashAttention, FlashInfer, TRTLLM-GEN, FlashMLA, Triton, CUTLASS, and more
- **Speculative Decoding**: Supports n-gram, suffix, EAGLE, and DFlash for faster generation

### Flexibility & Compatibility
- **Wide Model Support**: Works with 200+ model architectures on HuggingFace
  - Decoder-only LLMs (LLaMA, Qwen, Gemma)
  - Mixture-of-Experts (Mixtral, DeepSeek-V3, Qwen-MoE, GPT-OSS)
  - Hybrid attention/state-space models (Mamba, Qwen3.5)
  - Multi-modal models (LLaVA, Qwen-VL, Pixtral)
  - Embedding, retrieval, reward, and classification models
- **Multiple Decoding Strategies**: Parallel sampling, beam search, and more
- **Parallelism**: Tensor, pipeline, data, expert, and context parallelism for distributed inference
- **Quantization Support**: FP8, MXFP8/MXFP4, NVFP4, INT8, INT4, GPTQ/AWQ, GGUF, compressed-tensors, ModelOpt, TorchAO, and more
- **Multi-LoRA**: Efficient support for multiple LoRA adapters on dense and MoE layers
- **Structured Outputs**: Generate structured outputs using xgrammar or guidance
- **Tool Calling & Reasoning**: Built-in parsers for tool use and reasoning

### APIs & Deployment
- **OpenAI-Compatible API**: Drop-in replacement for OpenAI API
- **Anthropic Messages API**: Support for Anthropic's API format
- **gRPC Support**: Available for high-performance gRPC serving
- **Streaming Outputs**: Real-time token streaming
- **Disaggregated Serving**: Separate prefill, decode, and encode stages

### Hardware Support
- **GPUs**: NVIDIA, AMD
- **CPUs**: x86, ARM, PowerPC
- **Accelerators**: Google TPUs, Intel Gaudi, IBM Spyre, Huawei Ascend, Rebellions NPU, Apple Silicon, MetaX GPU, and more via plugins

## Getting Started

### Installation
```bash
pip install vllm
```

For specific hardware or advanced configurations, see the [installation documentation](https://docs.vllm.ai/en/latest/getting_started/installation/).

### Quick Start
Start an OpenAI-compatible server with a model:

```bash
vllm serve meta-llama/Llama-3.1-8B-Instruct
```

Make requests to the server:

```python
from openai import OpenAI

client = OpenAI(base_url="http://localhost:8000/v1", api_key="dummy")
response = client.chat.completions.create(
    model="meta-llama/Llama-3.1-8B-Instruct",
    messages=[{"role": "user", "content": "Hello!"}],
)
print(response.choices[0].message.content)
```

## Use Cases

- **Production LLM Serving**: High-throughput serving for production workloads
- **Local Development**: Run models locally with efficient resource usage
- **Multi-Model Serving**: Serve multiple models with LoRA adapters
- **Research & Experimentation**: Easy to use API for prototyping and research
- **Edge/Custom Hardware**: Deploy on diverse hardware platforms

## Resources

- [Official Documentation](https://docs.vllm.ai) - Complete guides, API reference, and tutorials
- [GitHub Repository](https://github.com/vllm-project/vllm) - Source code, issues, and contributions
- [vLLM Blog](https://blog.vllm.ai) - Announcements and technical deep dives
- [Roadmap](https://roadmap.vllm.ai) - Development roadmap
- [vLLM Paper](https://arxiv.org/abs/2309.06180) - "Efficient Memory Management for Large Language Model Serving with PagedAttention" (SOSP 2023)
- [Community Meetups](https://docs.vllm.ai/en/latest/community/meetups/) - Events and community resources
