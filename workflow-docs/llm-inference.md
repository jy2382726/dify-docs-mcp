# 大模型推理性能优化技术

大模型（LLM）推理性能直接影响应用响应速度和成本。本文总结主流的推理优化技术。

## KV Cache 优化

### 原理
Transformer 自回归解码时，每生成一个 token 都要重新计算所有历史 token 的 K/V 向量。KV Cache 将这些向量缓存起来避免重复计算。

### PagedAttention
vLLM 提出的分页注意力机制，将 KV Cache 组织为固定大小的 block，减少内存碎片，提升显存利用率至 90%+。

## 模型量化

### INT8 量化
将权重从 FP16 压缩到 INT8，显存减半，速度提升 30-50%，精度损失 <1%。

### INT4 量化（GPTQ、AWQ）
更激进的量化方案，显存降至 1/4，但精度损失较大，适合对延迟敏感的场景。

### GGUF 格式
llama.cpp 推出的量化格式，支持在 CPU 上运行大模型，适合边缘部署。

## 批处理

### 静态批处理
等待请求积累到一定数量后统一处理，吞吐量高但延迟不稳定。

### 动态批处理（Continuous Batching）
请求可以随时加入和退出批次，平衡吞吐和延迟。vLLM、TGI 默认采用此方案。

## 投机解码

### Speculative Decoding
用小模型快速生成候选 token，再用大模型并行验证。若验证通过则一次接受多个 token，速度提升 2-3 倍。

### Medusa
在原模型基础上增加多个解码头，并行预测多个未来 token，无需额外小模型。

## 模型架构优化

### GQA（Grouped Query Attention）
LLaMA 2/3 采用的多头注意力变体，减少 KV Cache 大小，提升推理速度。

### FlashAttention 2
通过分块计算和减少 HBM 访问，将注意力计算速度提升 2-4 倍。

## 部署框架对比

| 框架 | 特点 | 适用场景 |
|------|------|---------|
| vLLM | PagedAttention，高吞吐 | 生产环境高并发 |
| TGI | HuggingFace 官方，易用 | 中小规模部署 |
| llama.cpp | 纯 C++，CPU 友好 | 边缘设备、本地部署 |
| TensorRT-LLM | NVIDIA 官方，性能极致 | GPU 集群 |
| Ollama | 一键部署，开发者友好 | 本地开发测试 |