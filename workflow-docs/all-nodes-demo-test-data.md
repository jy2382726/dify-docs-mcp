# 全节点演示工作流测试数据

> 配套工作流：`all-nodes-demo.yaml`
> 用途：导入 Dify 后用于功能验证、分支覆盖测试、节点行为演示

---

## 一、前置条件检查清单

测试前必须完成以下准备，否则部分分支会失败。

| # | 依赖项 | 配置位置 | 说明 |
|---|--------|---------|------|
| 1 | 通义千问模型 | Dify 模型供应商 | 所有 LLM/Agent/Classifier/Extractor 节点需要。当前 DSL 使用 `langgenius/tongyi/tongyi` + `qwen3.5-flash`，可在 Dify 中替换为其他可用模型 |
| 2 | Tavily 搜索插件 | Dify 插件市场 | 安装 `langgenius/tavily/tavily`，导入后需重新授权并配置 `TAVILY_API_KEY` |
| 3 | 知识库 | Dify 知识库模块 | 创建任意一个知识库，将 DSL 中 `knowledge-retrieval` 节点的 `dataset_ids` 占位符 `"dataset-id"` 替换为实际 ID |
| 4 | HTTP 端点 | 默认 `https://httpbin.org/post` | 用于市场分析分支，可正常访问即可。如需替换，编辑 `http_req_1` 节点的 `url` |
| 5 | 环境变量 Memory | DSL 已声明 | 自动创建，无需额外操作 |

---

## 二、测试文件素材

以下内容用于"有文件"分支测试。请将内容保存为 `.md` 或 `.txt` 文件，在 start 节点的 `files` 字段上传。

### 文件 A：技术文档素材（用于技术调研场景）

文件名建议：`dify-architecture.md`

```markdown
# Dify 工作流引擎架构概述

## 核心组件

Dify 工作流引擎基于 ReactFlow 实现 DAG 编排，主要包含以下模块：

1. **节点系统**：支持 30+ 种内置节点类型，包括 LLM、Code、HTTP、Tool、Iteration 等
2. **变量系统**：通过 `{{#node_id.field#}}` 语法实现节点间数据传递
3. **执行引擎**：负责拓扑排序、并行调度、错误处理
4. **状态管理**：通过 conversation_variables 和 environment_variables 维护状态

## 关键特性

- 支持迭代（iteration）和循环（loop）两种容器节点
- 提供 if-else、question-classifier 等多种分支机制
- 集成主流模型供应商和插件生态
```

### 文件 B：市场数据素材（备用，可用于任何分支）

文件名建议：`market-brief.txt`

```text
2024 年 AI 编程助手市场报告摘要：全球市场规模约 12 亿美元，年增长率 35%。
主要玩家包括 GitHub Copilot、Cursor、Codeium、Tabnine。企业版渗透率持续提升。
```

---

## 三、测试用例矩阵

| 用例 | 主题 | 文件 | 预期分类 | 触达分支 | 覆盖节点数 |
|------|------|------|---------|---------|-----------|
| 用例 1 | Dify 工作流引擎的架构设计 | 文件 A | 技术调研 (1) | 文件 + RAG | 17/19 |
| 用例 2 | AI 编程助手市场竞品分析 | 无 | 市场分析 (2) | 空文本 + HTTP | 16/19 |
| 用例 3 | 如何规划一场城市马拉松 | 无 | 通用报告 (3) | 空文本 + 搜索 | 16/19 |
| 用例 4 | 大模型推理性能优化技术 | 文件 B | 技术调研 (1) | 文件 + RAG | 17/19 |
| 用例 5 | 2024 年新能源汽车市场分析 | 无 | 市场分析 (2) | 空文本 + HTTP | 16/19 |
| 用例 6 | 推荐一份周末亲子活动方案 | 无 | 通用报告 (3) | 空文本 + 搜索 | 16/19 |

---

## 四、详细测试用例

### 用例 1：技术调研 + 有文件（覆盖最全）

**输入**

| 字段 | 值 |
|------|---|
| topic | `Dify 工作流引擎的架构设计` |
| files | 上传 `dify-architecture.md`（见上方文件 A） |

**预期路径**

```
start → if-else(true) → list-operator → document-extractor
      → var_agg_1 → classifier(技术调研=1)
      → kg_retrieval → llm_rag → var_agg_2
      → code_split → iteration(iter_llm_1)
      → code_merge → loop(改进→评分→写回，至 ≥8 退出)
      → agent → assigner_final → end
```

**预期输出**

- `final_report`：包含技术分析、维度拆解、迭代优化痕迹、最终润色的完整报告
- 环境变量 `Memory`：被写入最终报告内容

**关键观察点**

- `document-extractor` 应正确提取 markdown 文本
- `kg_retrieval` 应返回知识库检索结果（即使不相关也会有返回）
- `iteration` 应按 `\n\n` 拆分维度后逐个深入
- `loop` 应在质量评分 ≥8 时退出（最多循环 5 次）

---

### 用例 2：市场分析 + 无文件

**输入**

| 字段 | 值 |
|------|---|
| topic | `AI 编程助手市场竞品分析` |
| files | （不上传） |

**预期路径**

```
start → if-else(false) → code_empty → var_agg_1
      → classifier(市场分析=2)
      → http_req → code_parse → tpl_transform → var_agg_2
      → code_split → iteration → code_merge → loop
      → agent → assigner_final → end
```

**预期输出**

- `final_report`：包含 HTTP 返回数据的解析结果和市场分析

**关键观察点**

- `if-else` 应走 false 分支（`code_empty` 输出空字符串）
- `http_req` 应返回 httpbin 的回显 JSON
- `code_parse` 应正确解析 JSON 并提取 `topic` 字段
- `tpl_transform` 应输出格式化的市场数据文本

---

### 用例 3：通用报告 + 无文件

**输入**

| 字段 | 值 |
|------|---|
| topic | `如何规划一场城市马拉松` |
| files | （不上传） |

**预期路径**

```
start → if-else(false) → code_empty → var_agg_1
      → classifier(通用报告=3)
      → param_extractor → tool_search → var_agg_2
      → code_split → iteration → code_merge → loop
      → agent → assigner_final → end
```

**预期输出**

- `final_report`：包含 Tavily 搜索结果和综合报告

**关键观察点**

- `param_extractor` 应从主题中提取出搜索关键词（如"城市马拉松 规划"）
- `tool_search` 应调用 Tavily 返回搜索结果文本
- Agent 节点最终可能再次调用 Tavily 补充信息

---

### 用例 4：技术调研 + 不同文件

**输入**

| 字段 | 值 |
|------|---|
| topic | `大模型推理性能优化技术` |
| files | 上传 `market-brief.txt`（见上方文件 B） |

**说明**：主题与文件内容不匹配，用于验证 `question-classifier` 是否依据主题（而非文件）分类。

**预期**：分类仍为"技术调研 (1)"，走 RAG 分支。

---

### 用例 5：市场分析（短主题）

**输入**

| 字段 | 值 |
|------|---|
| topic | `新能源汽车市场` |
| files | （不上传） |

**说明**：极短主题，验证分类器对短文本的处理。

---

### 用例 6：通用报告（生活类主题）

**输入**

| 字段 | 值 |
|------|---|
| topic | `推荐一份周末亲子活动方案` |
| files | （不上传） |

**说明**：明显非技术/市场类，验证分类器归到"通用报告"。

---

## 五、边界与异常测试

| 用例 | 输入 | 预期行为 |
|------|------|---------|
| B1 | topic 留空 | start 节点拒绝（required 校验失败） |
| B2 | topic 超过 200 字符 | start 节点拒绝（max_length 校验失败） |
| B3 | 上传空文件列表但选择"有文件"条件 | if-else 走 false 分支 |
| B4 | 上传非文本文件（如图片） | document-extractor 可能报错或返回空 |
| B5 | 知识库 ID 未替换（仍为 `"dataset-id"`） | kg_retrieval 报错，技术调研分支失败 |
| B6 | Tavily 未授权 | tool_search 报错，通用报告分支失败 |
| B7 | httpbin 不可达 | http_req 超时，市场分析分支失败 |

---

## 六、降级测试方案

当依赖项不可用时，可使用以下方案验证工作流骨架。

### 方案 1：无知识库时验证技术调研分支

- 临时修改 `kg_retrieval_1` 的 `dataset_ids`，指向任意已存在的知识库（内容无关紧要）
- 即使检索结果不相关，`llm_rag` 仍会生成报告，可验证分支连通性

### 方案 2：无 Tavily 时验证通用报告分支

- 在 Dify 中禁用 `tool_search_1` 节点（如果有此功能）
- 或临时将 `classifier_1` 的"通用报告"分类删除，强制走前两类

### 方案 3：仅验证主流程骨架

使用任意主题运行，观察是否在 `classifier_1` 处正确分类。如分类失败（如返回未匹配类），需调整 `classifier_1` 的 `instruction` 或 `classes`。

---

## 七、节点覆盖验证清单

测试完成后，逐项确认每个节点至少被触发一次。可在 Dify 工作流运行历史的"节点执行"视图中查看。

| # | 节点 ID | 节点类型 | 用例 1 | 用例 2 | 用例 3 | 备注 |
|---|---------|---------|--------|--------|--------|------|
| 1 | `start_1` | start | ✓ | ✓ | ✓ | |
| 2 | `if_else_1` | if-else | ✓ | ✓ | ✓ | |
| 3 | `list_op_1` | list-operator | ✓ | — | — | 仅"有文件"时 |
| 4 | `doc_extractor_1` | document-extractor | ✓ | — | — | 仅"有文件"时 |
| 5 | `code_empty` | code | — | ✓ | ✓ | 仅"无文件"时 |
| 6 | `var_agg_1` | variable-aggregator | ✓ | ✓ | ✓ | |
| 7 | `classifier_1` | question-classifier | ✓ | ✓ | ✓ | |
| 8 | `kg_retrieval_1` | knowledge-retrieval | ✓ | — | — | 仅"技术调研"时 |
| 9 | `llm_rag` | llm | ✓ | — | — | 仅"技术调研"时 |
| 10 | `http_req_1` | http-request | — | ✓ | — | 仅"市场分析"时 |
| 11 | `code_parse` | code | — | ✓ | — | 仅"市场分析"时 |
| 12 | `tpl_transform_1` | template-transform | — | ✓ | — | 仅"市场分析"时 |
| 13 | `param_extractor_1` | parameter-extractor | — | — | ✓ | 仅"通用报告"时 |
| 14 | `tool_search_1` | tool | — | — | ✓ | 仅"通用报告"时 |
| 15 | `var_agg_2` | variable-aggregator | ✓ | ✓ | ✓ | |
| 16 | `code_split` | code | ✓ | ✓ | ✓ | |
| 17 | `iteration_1` / `iter_llm_1` | iteration + llm | ✓ | ✓ | ✓ | |
| 18 | `code_merge` | code | ✓ | ✓ | ✓ | |
| 19 | `loop_1` / `loop_llm_1` / `loop_code_1` / `loop_assigner_1` | loop + llm + code + assigner | ✓ | ✓ | ✓ | |
| 20 | `agent_1` | agent | ✓ | ✓ | ✓ | |
| 21 | `assigner_final` | assigner | ✓ | ✓ | ✓ | |
| 22 | `end_1` | end | ✓ | ✓ | ✓ | |

**最小覆盖**：跑完用例 1、2、3 即可覆盖全部 19 种节点类型（用例 1 单独覆盖 17 种，加上用例 2/3 补全另外 2 个分支节点）。

---

## 八、调试建议

1. **逐步运行**：Dify 支持单步执行，建议在 `classifier_1` 后暂停，确认分类正确再继续
2. **查看变量**：每个节点执行后查看输出变量，确认数据格式与下游节点期望一致
3. **Loop 退出条件**：如果 `loop_1` 总是跑满 5 次，说明 `loop_code_1` 的评分逻辑过严，可调整评分代码
4. **Iteration 拆分**：如果 `iteration_1` 只循环一次，说明 `code_split` 的 `\n\n` 分隔符与上游输出格式不匹配，可在 `code_split` 中改为按其他分隔符（如换行符或标题）
