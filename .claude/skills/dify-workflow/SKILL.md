---
name: dify-workflow-dsl
description: >
  Create, modify, review, and debug Dify Workflow/Chatflow DSL YAML files that can
  be imported into Dify. Use for Dify app DSL, workflow YAML, advanced-chat YAML,
  graph nodes/edges, variables, tool nodes, plugin dependencies, database read/write
  tools, and import/export compatibility questions.
---

# Dify Workflow DSL

Use this skill to produce import-ready Dify DSL YAML. Dify calls the exported
workflow file a DSL; it is a YAML app definition with app metadata, dependencies,
workflow variables/features, and a ReactFlow-like graph of nodes and edges.

## Core Workflow

1. **确定模式**：默认 `workflow`。需要多轮对话、memory、`sys.query`、`sys.files`
   或 `answer` 节点时，使用 `advanced-chat`。详见 `references/usecase-node-selection.md`。
2. **收集需求**：确认 app mode、inputs、model/provider、插件、知识库、密钥、
   触发源、期望输出。
3. **选择版本**：新 DSL 使用 `version: "0.6.0"`（YAML 字符串格式）。
4. **绘制草图**：先画节点连接图，再写 YAML。
5. **连接边**：每条边必须有 `sourceType`、`targetType`、`sourceHandle`、`targetHandle`。
6. **添加依赖**：每个插件节点都需要 dependencies 条目。
   详见 `references/plugin-marketplace-tools.md`。
7. **查询文档**：节点不在本地 Schema 中时，用 Mintlify MCP 查询。
   详见 `references/mcp-usage-guide.md`。
8. **校验输出**：`python3 .claude/skills/dify-workflow/scripts/validate_dsl.py <file.yaml>`

## New DSL Intake

从自然语言需求创建工作流时，确认以下 5 项：

- **Mode**：默认 `workflow`；Chatflow 用 `advanced-chat`
- **Trigger**：start variables / chat input / schedule / webhook / plugin event
- **Inputs**：text / files / JSON / form fields / dataset IDs / tool credentials
- **Output**：end values / answer / side-effect tool / generated file
- **Shape**：straight-line / branch / extractor / RAG / iteration / agent

详细模式和触发器选择：`references/usecase-node-selection.md`

## Required Decisions

- **模式**：`workflow` 用于一次性/批量/触发/集成；`advanced-chat` 用于 Chatflow
- **密钥**：不硬编码真实密钥，用 `env` 变量或占位符
- **插件工具**：不承诺未知工具的可靠性，要求导出 DSL 或源码

详细决策矩阵：`references/usecase-node-selection.md`、`references/plugin-marketplace-tools.md`

## Authoring Rules

核心规则（完整规范见 reference 文件）：

1. `version: "0.6.0"` + `kind: app`
2. `workflow.graph.nodes` 和 `workflow.graph.edges` 必须存在
3. 节点 `type` 通常为 `custom`，`data.type` 为实际节点类型
4. 节点 `id` 必须为字符串，不可重复
5. 边必须引用存在的节点 ID
6. `if-else` 分支用 `"true"` / `"false"` sourceHandle
7. `value_selector` / `variable_selector` 为数组 `["node_id", "field"]`
8. Code 节点必须定义 `main` 函数
9. Tool 节点必须有 `provider_id`、`provider_name`、`provider_type`、`tool_name`、`tool_parameters`
10. `custom-note` 是合法的画布注释节点

完整官方规范：`references/official-0.6-target.md`
完整插件规则：`references/plugin-marketplace-tools.md`

## Validation Checklist

校验 DSL 前：

- YAML 解析成功
- `version` 是字符串，`app.mode` 匹配终端节点类型
- 依赖覆盖所有插件节点
- 所有边指向存在的节点且类型匹配
- 变量名唯一
- 每个 LLM 有 model 和 prompt_template
- 每个 tool 有必填字段
- SQL 无尾部逗号，用绑定参数
- 告知用户文件路径和校验结果

详细校验规则：`references/validation-rules.md`

## 加载策略

Reference 文件按 3 层组织，按需加载：

### 第 1 层：始终加载（每次生成 DSL 都需要）

| 文件 | 用途 |
|------|------|
| `references/dsl-structure.md` | DSL 顶层结构、变量、依赖、边 |
| `references/node-schemas.md` | 26 个节点的 Schema 和配置示例 |

### 第 2 层：按需加载（特定任务时加载）

| 文件 | 何时加载 |
|------|---------|
| `references/usecase-node-selection.md` | 收到业务需求，需要确定模式和节点组合 |
| `references/variable-syntax.md` | 需要写变量引用 `{{#node_id.field#}}` |
| `references/node-output-fields.md` | 需要确认节点输出字段名 |

### 第 3 层：特定场景加载（遇到特定场景时加载）

| 文件 | 何时加载 |
|------|---------|
| `references/database-tools.md` | 涉及 PostgreSQL/SQL 数据库操作 |
| `references/plugin-marketplace-tools.md` | 使用非内置工具（插件/MCP/workflow-provider） |
| `references/official-0.6-target.md` | 需要确认官方 DSL 规范、版本兼容、节点枚举 |
| `references/real-world-yml-study.md` | 需要参考真实工作流模式和经验 |
| `references/validation-rules.md` | 校验 DSL 时 |
| `references/mcp-usage-guide.md` | 使用 Mintlify MCP 查询文档时 |
| `references/templates/` | 需要模板参考时 |

## Mintlify MCP 文档查询

当本地 reference 不够用时，调用 MCP 查询 Dify 文档：

- 节点不在 26 个 Schema 列表中
- 用户 Dify 版本与 v0.6.0 不一致
- 查询 Context RAG、Jinja2、Memory 等高级特性
- 确认最新节点变更

可用工具：
- `search_dify_docs(query, version?, language?)` — 语义搜索
- `query_docs_filesystem_dify_docs(command)` — 虚拟文件系统（rg/cat/tree）

详见 `references/mcp-usage-guide.md`

## 降级策略

当 Mintlify MCP 不可用时：

1. 用 `references/node-schemas.md` 的 Schema（26 个节点）
2. 用 `references/templates/` 的模板
3. 用 `references/validation-rules.md` + validate_dsl.py 校验
4. 不确定的配置返回 warning，不报错
5. 告知用户："MCP 不可用，使用本地 reference，生成的 DSL 需要 Dify 导入测试"

## Useful Commands

- `python3 .claude/skills/dify-workflow/scripts/validate_dsl.py <file.yaml>` — 校验 DSL
- `rg -i "keyword" .claude/skills/dify-workflow/references/` — 搜索 reference
- `cat .claude/skills/dify-workflow/references/templates/<template>.yaml` — 查看模板
