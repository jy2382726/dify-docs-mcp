# RAG 系统设计与最佳实践

RAG（Retrieval-Augmented Generation，检索增强生成）通过引入外部知识库降低大模型的幻觉问题，是知识密集型应用的核心架构。

## 基本流程

1. **索引阶段**：将原始文档切块（chunking），用 embedding 模型向量化，存入向量数据库
2. **检索阶段**：用户查询同样向量化，从数据库召回 top-k 相似片段
3. **生成阶段**：将检索结果作为 context 注入 LLM prompt，生成最终回答

## 关键设计点

### 文档切块
- **固定长度切块**：简单但可能切断语义，推荐 256-512 token
- **语义切块**：按段落、标题切分，保留语义完整性
- **重叠切块**：相邻块重叠 50-100 token，避免边界信息丢失

### Embedding 模型选择
- **OpenAI text-embedding-3**：通用性强，多语言支持
- **BGE 系列**：开源，中文表现优秀
- **Cohere embed-v3**：商用，支持多语言
- 维度通常为 768、1024 或 1536

### 检索策略
- **向量检索**：基于语义相似度，适合模糊查询
- **关键词检索（BM25）**：基于词频，适合精确匹配
- **混合检索**：两者结合，召回率最高
- **Rerank**：用 cross-encoder 对召回结果重排序，提升精度

### Top-K 选择
- K 太小：召回不足，漏掉相关信息
- K 太大：噪声多，context 过长导致 LLM 注意力分散
- 实践中通常取 3-5

## Dify 中的 RAG 实现

Dify 通过 `knowledge-retrieval` 节点封装 RAG 检索能力：
- 支持 `multiple` 和 `single` 两种检索模式
- 可配置 top-k、reranking、score_threshold
- 检索结果通过 `context.variable_selector` 注入 LLM 节点

## 常见问题

### 检索不准
- 检查 embedding 模型是否与文档语言匹配
- 调整切块策略（试试更小的块）
- 启用 rerank 模型

### 回答仍有幻觉
- 增加 score_threshold 过滤低质量召回
- 在 prompt 中明确要求"仅基于 context 回答"
- 检查 context 是否被 LLM 正确接收（查看日志）

### 多轮对话上下文丢失
- 用 query rewrite 改写用户查询，融入历史信息
- 或使用 HyDE（Hypothetical Document Embeddings）先生成假设答案再检索