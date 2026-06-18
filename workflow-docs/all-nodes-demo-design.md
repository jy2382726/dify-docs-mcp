# 全节点演示工作流设计文档

> 目标：创建一个 Dify 1.4.2 兼容的工作流，尽可能覆盖所有平台内置节点类型。

## 一、Dify 1.4.2 节点清单

### 版本确认依据

- **Loop 节点**：v1.1.0 引入（代码合入），v1.2.0 正式发布 → 1.4.2 **可用**（版本号待验证，未在官方文档中找到确切出处）
- **Human Input 节点**：v1.13.0（2026-02-11）引入 → 1.4.2 **不可用**（版本号待验证）
- **Human Feedback 节点**：Dify 官方从未发布此独立节点类型

### 可用节点（22 种 + 画布注释，单工作流可同时使用 18 种）

| # | 节点类型 | 分类 | 备注 |
|---|---------|------|------|
| 1 | `start` | 基础流程 | |
| 2 | `end` | 基础流程 | workflow 模式出口 |
| 3 | `answer` | 基础流程 | 仅 chatflow 模式 |
| 4 | `llm` | 推理 | |
| 5 | `code` | 推理 | Python3 |
| 6 | `template-transform` | 推理 | Jinja2 模板 |
| 7 | `if-else` | 分支 | 二分支条件 |
| 8 | `question-classifier` | 分支 | 多分类路由 |
| 9 | `parameter-extractor` | 分支 | 结构化提取 |
| 10 | `http-request` | 外部调用 | REST API |
| 11 | `tool` | 外部调用 | 插件工具节点 |
| 12 | `variable-aggregator` | 变量 | 分支输出合并 |
| 13 | `assigner` | 变量 | 写入会话/循环变量 |
| 14 | `document-extractor` | 文档 | 提取文件文本 |
| 15 | `list-operator` | 文档 | 数组/文件列表操作 |
| 16 | `knowledge-retrieval` | 知识库 | RAG 检索 |
| 17 | `agent` | Agent | Function Calling 等策略 |
| 18 | `iteration` | 容器 | 数组遍历（无状态） |
| 19 | `loop` | 容器 | 条件循环（有状态） |
| 20 | `trigger-schedule` | 触发器 | 替代 start |
| 21 | `trigger-webhook` | 触发器 | 替代 start |
| 22 | `trigger-plugin` | 触发器 | 替代 start |
| — | `custom-note` | 画布 | 非执行注释节点 |

### 排除节点

| 节点 | 原因 |
|------|------|
| `human-input` | v1.13.0+ 才引入，1.4.2 不存在 |
| `datasource` | 插件依赖，需安装特定插件（如 Notion） |
| `knowledge-index` | 版本敏感，非通用内置 |

> **说明**：上表列出 22 种节点类型，但单个 workflow 中有互斥约束——`answer` 仅限 chatflow、3 个 `trigger-*` 与 `start` 互斥（一个入口）、`human-input` 在 1.4.2 不存在。因此单工作流可同时使用的处理节点为 18 种，加上 `custom-note` 共 19 种。

---

## 二、工作流设计

### 基本信息

- **名称**：全节点演示：智能多源分析报告生成器
- **模式**：`workflow`
- **业务场景**：用户输入研究主题（可选上传文件），系统自动分类 → 多源收集 → 深度分析 → 迭代优化 → 输出结构化报告。

### 前置条件

| 依赖项 | 说明 | DSL 中的处理方式 |
|--------|------|-----------------|
| 知识库 | `knowledge-retrieval` 的 `dataset_ids` 必须引用目标工作区中已创建的知识库 ID | 使用占位符 `dataset-id`，导入后需替换为实际 ID |
| 搜索工具插件 | "通用报告"分支的 `tool` 节点需要搜索引擎插件（推荐 `langgenius/tavily/tavily` 的 `tavily_search`） | DSL 中指定 provider 和 tool_name，导入前需先安装插件 |
| Agent 工具 | Agent 节点挂载 `tavily_search` 工具用于最终润色时的信息补充 | Agent 的 `tools.value` 中声明工具配置 |
| LLM 模型 | 所有 LLM/Agent/question-classifier/parameter-extractor 节点需要可用的模型配置 | 使用 `langgenius/tongyi/tongyi` + `qwen3.5-flash` 作为默认模型 |

### 流程图

```
┌─────────────────────────────────────────────────────────────────────────────┐
│ [custom-note] 本工作流演示 Dify 1.4.2 全部 18 种内置处理节点               │
└─────────────────────────────────────────────────────────────────────────────┘

[start]
  │  输入: topic(text-input), files(file-list)
  │
  ↓
[if-else]  ← 判断: files 不为空？
  │
  ├── true ──→ [list-operator] ──→ [document-extractor] ──┐
  │        （取第一个文件）        （提取文档文本）        │
  │                                                       │
  └── false ──→ [code]（生成空字符串） ────────────────────┤
                              [variable-aggregator-1] ←──┘
  │
  ↓
[question-classifier]  ← 分类: 技术调研 | 市场分析 | 通用报告
  │
  ├──(1) 技术调研
  │      [knowledge-retrieval] ──→ [llm-RAG] ────────────────────────────────┐
  │      （检索知识库）        （结合检索结果生成技术分析）                    │
  │                                                                           │
  ├──(2) 市场分析
  │      [http-request] ──→ [code] ──→ [template-transform] ────────────────┤
  │      （调用市场API）  （解析JSON）（Jinja2 格式化为统一结构）             │
  │                                                                           │
  └──(3) 通用报告
         [parameter-extractor] ──→ [tool] ──────────────────────────────────┤
         （提取搜索关键词）       （搜索引擎工具执行搜索）                    │
                                                                           │
                                [variable-aggregator-2] ←──────────────────┘
  │
  ↓
[code]  ← 将聚合文本按维度拆分为数组，供 iteration 遍历
  │
  ↓
[iteration]  ← 遍历各分析维度，逐个深入分析
  │  ┌──────────────────────────────────────────────┐
  │  │ [iteration-start] → [llm]（逐维度深入分析）   │
  │  └──────────────────────────────────────────────┘
  │
  ↓
[code]  ← 将 iteration 输出的 array[string] 合并为单一文本
  │
  ↓
[loop]  ← 迭代优化报告质量，质量分 >= 8 则退出
  │  ┌──────────────────────────────────────────────────────────────────────┐
  │  │ [loop-start] → [llm]（改进报告） → [code]（质量评分） → [assigner] │
  │  └──────────────────────────────────────────────────────────────────────┘
  │
  ↓
[agent]  ← 最终润色（Function Calling 策略，可挂载工具）
  │
  ↓
[end]  ← 输出: final_report
```

> **关于 assigner 节点的说明**：
>
> workflow 模式下，环境变量（environment_variables）是只读常量，**不能被 assigner 修改**；会话变量（conversation_variables）按 Dify 官方文档是 "Chatflow only"，workflow 模式下不持久化。
>
> 因此 workflow 模式中 assigner 节点的合法写入目标只有 **loop 容器内部的 `loop_variables`**。本工作流的 assigner 节点只用在 loop 内部（写回 `report_text` 和 `quality_score`），主流程结尾不再使用额外 assigner。

### 节点详细说明

#### 阶段一：输入与文件处理

| 节点 | 类型 | 输入 | 输出 | 说明 |
|------|------|------|------|------|
| start | start | — | topic(text-input), files(file-list) | 用户输入研究主题，可选上传文件 |
| if-else | if-else | start.files | true/false 分支 | 判断是否上传了文件（条件：files not empty） |
| list-operator | list-operator | start.files | result | 从文件列表中提取第一个文件 |
| document-extractor | document-extractor | list-operator.result | text | 从单个文件中提取文本内容 |
| code（空文本） | code | — | empty_text | false 分支生成空字符串常量，作为 variable-aggregator 的占位输入 |
| variable-aggregator-1 | variable-aggregator | document-extractor.text / code.empty_text | output | 合并有无文件的两个分支（output_type: string） |

#### 阶段二：智能分类与多源信息收集

| 节点 | 类型 | 输入 | 输出 | 说明 |
|------|------|------|------|------|
| question-classifier | question-classifier | variable-aggregator-1.output | 分类 ID (1/2/3) | 将查询分为技术/市场/通用三类 |
| knowledge-retrieval | knowledge-retrieval | start.topic | result | 检索知识库中的相关文档（需预配置 dataset_ids） |
| llm-RAG | llm | knowledge-retrieval.result + start.topic | text | 结合知识库结果进行技术分析 |
| http-request | http-request | start.topic | body | 调用外部市场数据 API |
| code | code | http-request.body | parsed_data | 解析 API 返回的 JSON |
| template-transform | template-transform | code.parsed_data | output | 用 Jinja2 模板格式化市场数据 |
| parameter-extractor | parameter-extractor | start.topic | keywords | 从主题中提取搜索关键词 |
| tool | tool | parameter-extractor.keywords | text | 使用 Tavily Search 搜索引擎插件获取信息（需预安装 langgenius/tavily/tavily） |
| variable-aggregator-2 | variable-aggregator | 三个分支输出 | output | 合并三类信息收集结果 |

#### 阶段三：深度分析与迭代优化

| 节点 | 类型 | 输入 | 输出 | 说明 |
|------|------|------|------|------|
| code（拆分） | code | variable-aggregator-2.output | dimensions | 将聚合文本按维度拆分为字符串数组 |
| iteration | iteration | code（拆分）.dimensions | array[string] | 遍历各分析维度 |
| iteration 内 llm | llm | iteration.item | text | 对每个维度进行深入分析 |
| code（合并） | code | iteration.output | merged_text | 将 array[string] 合并为单一文本 |
| loop | loop | code（合并）.merged_text | 最终优化文本 | 迭代优化至质量达标 |
| loop 内 llm | llm | loop 报告文本 + 评分反馈 | text | 根据质量反馈改进报告 |
| loop 内 code | code | loop 内 llm.text | quality_score | 计算报告质量评分 |
| loop 内 assigner | assigner | code 输出 | — | 将评分和改进文本写回 loop_variables |

#### 阶段四：最终输出

| 节点 | 类型 | 输入 | 输出 | 说明 |
|------|------|------|------|------|
| agent | agent | loop 最终文本 | text | 最终润色，挂载 tavily_search 工具辅助信息补充 |
| end | end | agent.text | final_report | 输出最终报告 |

---

## 三、节点覆盖统计

### 已覆盖（19 种）

| # | 节点类型 | 流程中实例数 |
|---|---------|------------|
| 1 | `start` | 1 |
| 2 | `end` | 1 |
| 3 | `llm` | 4（RAG/维度分析/Loop 内改进/Agent 内） |
| 4 | `code` | 5（拆分维度/解析 JSON/质量评分/合并数组/空文本生成） |
| 5 | `template-transform` | 1 |
| 6 | `if-else` | 1 |
| 7 | `question-classifier` | 1 |
| 8 | `parameter-extractor` | 1 |
| 9 | `http-request` | 1 |
| 10 | `tool` | 1 |
| 11 | `variable-aggregator` | 2 |
| 12 | `assigner` | 1（仅 Loop 内写回 loop_variables） |
| 13 | `document-extractor` | 1 |
| 14 | `list-operator` | 1 |
| 15 | `knowledge-retrieval` | 1 |
| 16 | `agent` | 1 |
| 17 | `iteration` | 1 |
| 18 | `loop` | 1 |
| 19 | `custom-note` | 1 |

### 未包含及原因

| 节点 | 原因 |
|------|------|
| `answer` | 仅 chatflow 模式可用，本工作流使用 workflow 模式 |
| `trigger-schedule` | 替代 start 作为入口，本工作流使用 start 手动触发 |
| `trigger-webhook` | 同上 |
| `trigger-plugin` | 同上 |
| `human-input` | 1.4.2 中不存在（v1.13.0+） |
| `datasource` | 插件依赖，需安装特定插件 |
| `knowledge-index` | 版本敏感，非通用内置 |

---

## 四、关键设计决策

### 1. 两段式分支：if-else + question-classifier 各有分工

- `if-else`：判断有无上传文件（简单二值判断）
- `question-classifier`：对查询进行语义分类（需要 LLM 理解能力）

两种分支节点的设计意图不同，放在不同阶段展示其各自优势。

### 2. iteration vs loop 分离：展示两种容器的本质差异

- `iteration`：遍历数组，每轮通过 `["iter_id", "item"]` 取当前元素，子节点无状态。内部首节点需指定 `startNodeType: llm`（对应内部 LLM 节点）
- `loop`：无 item 概念，状态通过 `loop_variables` + `assigner` 跨轮次传递

### 3. code 出现多次：展示同一节点的不同用途

- 数据转换：拆分文本为数组（iteration 前）、合并数组为文本（loop 前）
- 数据处理：解析 HTTP 返回的 JSON
- 评分逻辑：在 loop 内部计算质量分数
- 占位生成：在 false 分支生成空字符串

### 4. agent 放在末端：展示 Function Calling 策略

Agent 担任"最终润色"角色，挂载 `tavily_search` 工具，展示 Agent 节点的工具编排能力。Agent 必须挂载至少一个工具，否则应改用 LLM 节点。

### 5. 触发器节点不在主流程中

`trigger-schedule`、`trigger-webhook`、`trigger-plugin` 是 start 的替代入口。在单个工作流中只能有一个入口，本 demo 选择 start 以保持简洁。可在文档中标注这些触发器的适用场景。
