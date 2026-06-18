# 知识库文档素材

> 用途：导入 Dify 知识库后供 `knowledge-retrieval` 节点检索
> 适用测试用例：用例 1（Dify 工作流引擎架构）、用例 4（大模型推理性能优化）
>
> **使用方式**：将下方每份文档（用 `---` 分隔）单独保存为 `.md` 文件，上传到 Dify 知识库。
> 推荐命名：`dify-nodes.md`、`dify-architecture.md`、`llm-inference.md`、`iteration-vs-loop.md`、`rag-design.md`

---

## 文档 1：Dify 工作流节点详解

```markdown
# Dify 工作流节点详解

Dify 工作流引擎提供 30+ 种内置节点类型，覆盖从输入解析到 LLM 推理、外部调用、容器迭代等场景。本文详细介绍核心节点的功能与使用方式。

## 基础流程节点

### start 节点
工作流的入口节点，定义用户输入变量。支持多种变量类型：
- text-input / paragraph：文本输入
- number：数值
- select：下拉选择
- file / file-list：文件上传
- json：结构化数据

### end 节点
workflow 模式的出口，定义输出变量。每个输出变量通过 `value_selector` 引用上游节点的字段。

### answer 节点
chatflow 模式（advanced-chat）的出口，直接生成对话回复。

## 推理与转换节点

### llm 节点
大语言模型调用节点，支持 system/user 角色提示模板，可通过 `context` 字段注入知识库检索结果。支持 chat/completion 两种模式。

### code 节点
Python3 代码执行节点，必须定义 `main` 函数，返回 dict。常用于 JSON 解析、数据转换、字符串处理。

### template-transform 节点
Jinja2 模板渲染节点，运行在沙箱中。注意不能调用 dict 的 `.items()` 等方法，需要先用 code 节点转换为固定字段结构。

## 分支与提取节点

### if-else 节点
条件分支节点，Dify 限制为最多两个分支（true/false）。需要超过两个分支时，需串联多个 if-else。

### question-classifier 节点
基于 LLM 的语义分类节点，支持多个类别，分支 handle 即类别 ID。

### parameter-extractor 节点
结构化参数提取节点，根据定义的参数 schema 从文本中提取字段值。

## 容器节点

### iteration 节点
数组遍历容器，每轮通过 `["iteration_id", "item"]` 取当前元素，子节点无状态。

### loop 节点
条件循环容器，通过 `loop_variables` 维护状态，需要 `assigner` 节点写回更新值。支持 `break_conditions` 提前退出。
```

---

## 文档 2：Dify 架构与设计原理

```markdown
# Dify 架构与设计原理

## 整体架构

Dify 是一个开源的 LLM 应用开发平台，核心架构分为以下层次：

1. **应用层**：提供 Chatflow、Workflow、Agent 等多种应用模式
2. **编排层**：基于 ReactFlow 的 DAG 图编辑器，支持节点拖拽和连线
3. **执行引擎**：负责拓扑排序、并行调度、错误处理、状态管理
4. **集成层**：对接主流模型供应商（OpenAI、Anthropic、通义千问等）和插件生态
5. **存储层**：使用 PostgreSQL 存储应用配置，Redis 缓存运行时状态，Weaviate/Qdrant 做向量检索

## 工作流执行引擎

### 执行流程
1. 从 start 节点开始，按边的拓扑顺序执行
2. 遇到 if-else/question-classifier 时根据条件路由
3. 遇到 iteration 时按数组长度循环内部子图
4. 遇到 loop 时按条件循环，通过 assigner 更新状态
5. 到达 end 节点时输出最终结果

### 并行调度
iteration 节点支持 `is_parallel: true` 并行执行，通过 `parallel_nums` 控制并发数（默认 10）。

### 错误处理
容器节点支持 `error_handle_mode`：
- `terminated`：出错时终止整个工作流
- `remove-abnormally`：跳过异常项继续执行

## 变量系统

### 变量引用语法
- `{{#node_id.field#}}`：引用上游节点输出
- `{{#sys.query#}}`：系统变量（用户查询、文件、用户 ID 等）
- `{{#env.API_KEY#}}`：环境变量
- `{{#conversation.Memory#}}`：对话变量（chatflow）

### 数据类型
支持 string、number、boolean、object、array[string]、array[number]、array[object]、array[file]、file 等。

## 插件生态

Dify 通过插件机制扩展能力：
- 模型供应商插件（如 langgenius/tongyi）
- 工具插件（如 langgenius/tavily 提供搜索能力）
- 数据源插件（如 Notion、GitHub 集成）
- MCP 协议支持

插件通过 marketplace、package、github 三种方式分发。
```

---

## 文档 3：大模型推理性能优化

```markdown
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
```

---

## 文档 4：iteration 与 loop 节点对比

```markdown
# iteration 与 loop 节点对比

Dify 工作流提供两种容器节点：iteration（迭代）和 loop（循环）。它们的本质差异决定了使用场景。

## 核心差异

| 维度 | iteration | loop |
|------|-----------|------|
| 数据来源 | 外部数组（iterator_selector） | 内部状态（loop_variables） |
| 每轮输入 | 通过 `item` 取当前元素 | 通过 `loop_variables` 读取状态 |
| 状态管理 | 无状态，每轮独立 | 有状态，跨轮次传递 |
| 退出条件 | 数组遍历完 | break_conditions 满足或达到 loop_count |
| 写回机制 | 不需要 | 必须用 assigner 节点写回 |

## iteration 适用场景

- 批量处理一组同类数据（如多个文档摘要、多维度分析）
- 每个元素的处理相互独立
- 输出是数组（如 `array[string]`）

**典型示例**：遍历用户上传的多个文件，逐个提取关键信息。

## loop 适用场景

- 需要根据上一轮结果决定下一轮行为
- 有明确的退出条件（如质量达标、收敛）
- 状态需要跨轮次维护

**典型示例**：迭代优化报告，每轮根据评分反馈改进，直到质量分 ≥8。

## 变量引用差异

### iteration
```text
当前元素：{{#iteration_1.item#}}
当前索引：{{#iteration_1.index#}}
```

### loop
```text
循环变量：{{#loop_1.variable_label#}}  （不是 item！）
```

## 常见陷阱

1. **loop 内忘记 assigner**：仅靠代码节点 return 不会更新 loop_variables，下一轮读到的是旧值
2. **loop 内写 `["loop_id", "item"]`**：这是 iteration 的语法，loop 没有 item
3. **iteration 内子节点有状态依赖**：每轮独立执行，前一轮的状态不会保留
4. **break_conditions 的 value 是变量**：必须加 `numberVarType: variable`，否则按常量处理
```

---

## 文档 5：RAG 系统设计与最佳实践

```markdown
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
```

---

## 推荐导入配置

将上述 5 份文档上传到 Dify 知识库时，建议使用以下配置：

| 配置项 | 推荐值 | 原因 |
|--------|--------|------|
| 分段规则 | 自动分段 | Markdown 文档按标题切分效果更好 |
| 分段长度 | 500 token | 平衡召回率和精度 |
| 分段重叠 | 50 token | 避免边界信息丢失 |
| 索引方式 | 高质量（向量+关键词） | 支持混合检索 |
| Embedding 模型 | text-embedding-3-small 或 BGE-large-zh | 中文表现优秀 |
| Rerank 模型 | 可选，建议启用 | 提升检索精度 |

## 检索验证

知识库创建后，先用以下查询测试检索效果（在知识库页面的"召回测试"中）：

| 查询 | 期望命中文档 |
|------|-------------|
| Dify 有哪些节点类型 | 文档 1（节点详解） |
| Dify 的整体架构是什么 | 文档 2（架构原理） |
| KV Cache 是什么 | 文档 3（推理优化） |
| iteration 和 loop 有什么区别 | 文档 4（容器对比） |
| RAG 系统怎么设计 | 文档 5（RAG 设计） |

如检索效果不佳，调整分段长度或 rerank 配置后重新索引。
