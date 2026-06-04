# Phase 1 Skill 适配 Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 基于 yzmw123/dify-workflow-dsl-skill 适配 + 补充增量内容，交付完整的 `.claude/skills/dify-workflow/` Skill 目录，使 Claude Code 能生成可导入 Dify 的 DSL YAML。

**Architecture:** 三层渐进架构的 Layer 2（结构化知识层）。基座来自 yzmw123 的 SKILL.md + references/ + scripts/，增量补充 rag 变量前缀、节点输出字段（r-hashi01）、3 个模板（R3flector）、Mintlify MCP 使用指引。

**Tech Stack:** Claude Code Skill（纯 Markdown）、Python 3.11（validate_dsl.py）、PyYAML、Mintlify MCP（.mcp.json 已配置）

---

## 文件结构

```
.claude/skills/dify-workflow/
├── SKILL.md                          # 主 Skill 文件（Task 2）
├── references/
│   ├── node-schemas.md               # 25+ 节点 Schema（Task 3）
│   ├── dsl-structure.md              # DSL 结构规范（Task 4）
│   ├── variable-syntax.md            # 变量引用语法 + rag 前缀（Task 5）
│   ├── node-output-fields.md         # 节点输出字段（Task 6）
│   ├── validation-rules.md           # 校验规则说明（Task 7）
│   ├── mcp-usage-guide.md            # Mintlify MCP 使用指引（Task 8）
│   └── templates/                    # 工作流模板（Task 9）
│       ├── simple-llm.yaml
│       ├── if-else-branch.yaml
│       ├── http-code.yaml
│       ├── chatflow-multi-turn.yaml
│       ├── rag-retrieval.yaml
│       ├── iteration.yaml
│       └── error-handling.yaml
├── scripts/
│   └── validate_dsl.py               # 校验脚本（Task 10）
└── examples/                         # 集成测试产物（Task 11）
    ├── rag-workflow.yaml
    ├── conditional-branch.yaml
    └── chatflow-multi-turn.yaml
```

---

## Task 1: 克隆 yzmw123 项目到临时目录

**[FR-全部 来源] [无依赖] [出参验证：目录存在且包含 SKILL.md]**

**Files:**
- Create: `/tmp/yzmw123-skill/` (临时目录)

- [ ] **Step 1: 克隆 yzmw123 仓库**

```bash
git clone https://github.com/yzmw123/dify-workflow-dsl-skill.git /tmp/yzmw123-skill
```

- [ ] **Step 2: 验证关键文件存在**

```bash
ls -la /tmp/yzmw123-skill/SKILL.md /tmp/yzmw123-skill/references/ /tmp/yzmw123-skill/scripts/validate_dsl.py
```

Expected: 三个路径均存在

- [ ] **Step 3: 读取 SKILL.md 确认内容完整性**

```bash
wc -l /tmp/yzmw123-skill/SKILL.md
```

Expected: > 100 行

- [ ] **Step 4: Commit**

```bash
# 无需 commit，临时目录
```

---

## Task 2: 创建 Skill 目录结构 + SKILL.md

**[FR-F2/F3/F4/F5/F7 来源] [依赖 Task 1] [出参验证：SKILL.md 存在且包含 frontmatter]**

**Files:**
- Create: `.claude/skills/dify-workflow/SKILL.md`

- [ ] **Step 1: 创建目录结构**

```bash
mkdir -p .claude/skills/dify-workflow/{references,scripts,examples}
```

- [ ] **Step 2: 读取 yzmw123 的 SKILL.md 作为基座**

```bash
cat /tmp/yzmw123-skill/SKILL.md
```

- [ ] **Step 3: 创建适配后的 SKILL.md**

基于 yzmw123 的 SKILL.md，做以下适配：

1. 保留 frontmatter（name/description）
2. 保留 Core Workflow、New DSL Intake、Required Decisions、Authoring Rules 段落
3. 修改 Reference Map 指向本项目的 references/ 路径
4. 新增 "Mintlify MCP 文档查询" 段落，说明何时调用 MCP
5. 修改 Validation Checklist 引用本项目的 validate_dsl.py 路径
6. 在末尾新增 "降级策略" 段落：Mintlify MCP 不可用时降级为纯 Skill 模式

```markdown
---
name: dify-workflow-dsl
description: >
  Create, modify, review, and debug Dify Workflow/Chatflow DSL YAML files that can
  be imported into Dify. Use for Dify app DSL, workflow YAML, advanced-chat YAML,
  graph nodes/edges, variables, tool nodes, plugin dependencies, database read/write
  tools, and import/export compatibility questions.
---

# Dify Workflow DSL

[从 yzmw123 SKILL.md 复制核心内容，修改 Reference Map 路径]

## Mintlify MCP 文档查询

当遇到以下情况时，调用 Mintlify MCP 查询 Dify 官方文档：
- 节点配置不在 references/node-schemas.md 的 25+ 列表中
- 用户的 Dify 版本与 Skill 中的 Schema 版本（v0.6.0）不一致
- 需要查询 Context RAG、Jinja2 模板、Memory 等高级特性
- 需要确认最新 Dify 版本的节点变更

可用工具：
- `search_dify_docs(query, version?, language?)` — 语义搜索
- `query_docs_filesystem_dify_docs(command)` — 虚拟文件系统（rg/cat/tree）

## 降级策略

当 Mintlify MCP 不可用时：
1. 使用 references/node-schemas.md 中的 Schema 信息
2. 使用 references/templates/ 中的模板
3. 使用 scripts/validate_dsl.py 校验
4. 对于不确定的配置，返回 warning 而非 error
```

- [ ] **Step 4: 验证 SKILL.md 包含 frontmatter**

```bash
head -5 .claude/skills/dify-workflow/SKILL.md | grep -q "name: dify-workflow-dsl" && echo "OK" || echo "FAIL"
```

Expected: OK

- [ ] **Step 5: Commit**

```bash
git add .claude/skills/dify-workflow/SKILL.md
git commit -m "feat: create SKILL.md adapted from yzmw123"
```

---

## Task 3: 适配 node-schemas.md（25+ 节点 Schema）

**[FR-F2 来源] [依赖 Task 1] [出参验证：文件存在且包含 >= 25 个节点类型]**

**Files:**
- Create: `.claude/skills/dify-workflow/references/node-schemas.md`

- [ ] **Step 1: 读取 yzmw123 的 node-schemas.md**

```bash
cat /tmp/yzmw123-skill/references/node-schemas.md
```

- [ ] **Step 2: 复制到目标位置**

```bash
cp /tmp/yzmw123-skill/references/node-schemas.md .claude/skills/dify-workflow/references/node-schemas.md
```

- [ ] **Step 3: 补充缺失的扩展节点**

在文件末尾追加以下扩展节点的 Schema（从 Dify 源码 entities.py 提取）：
- `agent` — Agent 节点
- `knowledge-retrieval` — 知识库检索节点
- `trigger` — 触发器节点
- `datasource` — 数据源节点
- `document-extractor` — 文档提取器

每个节点包含：参数定义、类型约束、配置示例（YAML）。

- [ ] **Step 4: 验证节点数量**

```bash
grep -c "^### " .claude/skills/dify-workflow/references/node-schemas.md
```

Expected: >= 25

- [ ] **Step 5: Commit**

```bash
git add .claude/skills/dify-workflow/references/node-schemas.md
git commit -m "feat: adapt node-schemas.md with 25+ node types"
```

---

## Task 4: 适配 dsl-structure.md（DSL 结构规范）

**[FR-F2 来源] [依赖 Task 1] [出参验证：文件存在且包含 version 0.6.0 说明]**

**Files:**
- Create: `.claude/skills/dify-workflow/references/dsl-structure.md`

- [ ] **Step 1: 读取 yzmw123 的 dsl-structure.md**

```bash
cat /tmp/yzmw123-skill/references/dsl-structure.md
```

- [ ] **Step 2: 复制到目标位置**

```bash
cp /tmp/yzmw123-skill/references/dsl-structure.md .claude/skills/dify-workflow/references/dsl-structure.md
```

- [ ] **Step 3: 验证内容包含版本说明**

```bash
grep -q "0.6.0" .claude/skills/dify-workflow/references/dsl-structure.md && echo "OK" || echo "FAIL"
```

Expected: OK

- [ ] **Step 4: Commit**

```bash
git add .claude/skills/dify-workflow/references/dsl-structure.md
git commit -m "feat: adapt dsl-structure.md from yzmw123"
```

---

## Task 5: 创建 variable-syntax.md（变量引用语法 + rag 前缀）

**[FR-F3 来源] [依赖 Task 1] [出参验证：文件存在且包含 rag 前缀说明]**

**Files:**
- Create: `.claude/skills/dify-workflow/references/variable-syntax.md`

- [ ] **Step 1: 从 yzmw123 SKILL.md 提取变量语法内容**

```bash
grep -A 50 "variable_selector\|value_selector\|{{#" /tmp/yzmw123-skill/SKILL.md
```

- [ ] **Step 2: 创建 variable-syntax.md**

内容包括：

```markdown
# 变量引用语法规则

## 两种变量格式

### 工作流变量引用（含 #）
格式：`{{#node_id.field#}}`
用途：引用上游节点的输出值
示例：`{{#llm_1.text#}}`、`{{#start_1.query#}}`

### Prompt 占位符（不含 #）
格式：`{{variable_name}}`
用途：Agent Prompt 模板中的变量占位
示例：`{{sys.query}}`、`{{user_name}}`

## 变量前缀

| 前缀 | 说明 | 示例 |
|------|------|------|
| `sys` | 系统变量（16 个） | `sys.query`、`sys.files`、`sys.user_id` |
| `env` | 环境变量 | `env.API_KEY`、`env.DB_URL` |
| `conversation` | 对话变量 | `conversation.history` |
| `rag` | RAG 检索上下文 | `rag.top_k`、`rag.score_threshold` |

## value_selector 和 variable_selector

- `value_selector` 和 `variable_selector` 是数组格式：`["node_id", "field"]`
- 示例：`["llm_1", "text"]` 表示引用 llm_1 节点的 text 输出

## 系统变量完整列表（16 个）

| 变量名 | 类型 | 说明 |
|--------|------|------|
| `sys.query` | string | 用户查询文本 |
| `sys.files` | array[File] | 用户上传的文件 |
| `sys.user_id` | string | 用户 ID |
| `sys.conversation_id` | string | 会话 ID |
| `sys.dialogue_count` | number | 对话轮次 |
| `sys.chat_history` | string | 聊天历史 |
| `sys.current_timestamp` | number | 当前时间戳 |
| `sys.current_date` | string | 当前日期 |
| `sys.current_time` | string | 当前时间 |
| `sys.timezone` | string | 时区 |
| `sys.language` | string | 语言 |
| `sys.app_name` | string | 应用名称 |
| `sys.app_description` | string | 应用描述 |
| `sys.recommended_questions` | array[string] | 推荐问题 |
| `time_diff` | string | 时间差 |
| `upload_file` | File | 上传文件 |

## 常见错误

1. 混用 `{{#...#}}` 和 `{{...}}` — 前者用于节点输出引用，后者用于 Prompt 模板
2. 忘记 `#` 号 — `{{node_id.field}}` 不会解析为节点引用
3. 引用不存在的节点 — 会导致导入后运行时错误
```

- [ ] **Step 3: 验证 rag 前缀存在**

```bash
grep -q "rag" .claude/skills/dify-workflow/references/variable-syntax.md && echo "OK" || echo "FAIL"
```

Expected: OK

- [ ] **Step 4: Commit**

```bash
git add .claude/skills/dify-workflow/references/variable-syntax.md
git commit -m "feat: create variable-syntax.md with rag prefix"
```

---

## Task 6: 创建 node-output-fields.md（节点输出字段）

**[FR-F6 来源] [无依赖] [出参验证：文件存在且包含 >= 10 个节点的输出字段]**

**Files:**
- Create: `.claude/skills/dify-workflow/references/node-output-fields.md`

- [ ] **Step 1: 创建 node-output-fields.md**

从 Dify 源码和 r-hashi01 项目提取各节点的输出字段名：

```markdown
# 节点输出字段参考

用于正确生成变量引用 `{{#node_id.field#}}` 中的 field 部分。

## start
无输出字段（变量通过 start node 的 variables 定义）

## llm
| 字段 | 类型 | 说明 |
|------|------|------|
| `text` | string | LLM 生成的文本 |
| `usage` | object | Token 用量（prompt_tokens, completion_tokens, total_tokens） |
| `finish_reason` | string | 结束原因（stop, length 等） |

## code
| 字段 | 类型 | 说明 |
|------|------|------|
| `result` | any | 代码执行结果（对应 outputs 中定义的字段） |

## if-else
无输出字段（通过 sourceHandle 分流）

## knowledge-retrieval
| 字段 | 类型 | 说明 |
|------|------|------|
| `result` | array[object] | 检索结果列表 |
| `result[].content` | string | 文档片段内容 |
| `result[].title` | string | 文档标题 |
| `result[].score` | number | 相似度分数 |

## tool
| 字段 | 类型 | 说明 |
|------|------|------|
| `text` | string | 工具返回的文本结果 |
| `files` | array[File] | 工具返回的文件 |
| `json` | object | 工具返回的 JSON 结果 |

## end
无输出字段（定义 outputs 数组指定返回值）

## answer
无输出字段（answer 字段定义回复内容）

## iteration
| 字段 | 类型 | 说明 |
|------|------|------|
| `output` | array | 迭代输出列表 |

## loop
| 字段 | 类型 | 说明 |
|------|------|------|
| `output` | array | 循环输出列表 |

## parameter-extractor
| 字段 | 类型 | 说明 |
|------|------|------|
| `extracted_parameters` | object | 提取的参数值 |

## http-request
| 字段 | 类型 | 说明 |
|------|------|------|
| `response` | object | HTTP 响应体 |
| `status_code` | number | HTTP 状态码 |
| `headers` | object | 响应头 |

## variable-aggregator
| 字段 | 类型 | 说明 |
|------|------|------|
| `output` | any | 聚合后的变量值 |

## document-extractor
| 字段 | 类型 | 说明 |
|------|------|------|
| `text` | string | 提取的文本内容 |

## agent
| 字段 | 类型 | 说明 |
|------|------|------|
| `text` | string | Agent 生成的文本 |
| `usage` | object | Token 用量 |
| `finish_reason` | string | 结束原因 |
```

- [ ] **Step 2: 验证节点数量**

```bash
grep -c "^## " .claude/skills/dify-workflow/references/node-output-fields.md
```

Expected: >= 10

- [ ] **Step 3: Commit**

```bash
git add .claude/skills/dify-workflow/references/node-output-fields.md
git commit -m "feat: create node-output-fields.md with 14 node types"
```

---

## Task 7: 创建 validation-rules.md（校验规则说明）

**[FR-F4 来源] [依赖 Task 1] [出参验证：文件存在且包含 >= 15 条规则]**

**Files:**
- Create: `.claude/skills/dify-workflow/references/validation-rules.md`

- [ ] **Step 1: 从 yzmw123 的 validate_dsl.py 提取校验规则**

```bash
grep -E "report\.(error|warn)" /tmp/yzmw123-skill/scripts/validate_dsl.py | wc -l
```

Expected: >= 15

- [ ] **Step 2: 创建 validation-rules.md**

基于 validate_dsl.py 中的规则，编写人类可读的校验规则文档：

```markdown
# DSL 校验规则

Claude Code 生成 DSL YAML 后，按以下规则逐项校验。发现错误时自动修复；无法修复时返回 warning。

## 必检项（Error 级别）

| # | 规则 | 检查方式 |
|---|------|---------|
| 1 | YAML 解析成功 | 顶层必须是 mapping |
| 2 | version 是字符串 | `version: "0.6.0"` |
| 3 | app.mode 合法 | workflow / advanced-chat / chat / completion / agent-chat |
| 4 | workflow.graph.nodes 存在 | graph 模式必须有 nodes |
| 5 | workflow.graph.edges 存在 | graph 模式必须有 edges |
| 6 | 节点 ID 唯一 | 不允许重复 ID |
| 7 | 节点 data.type 存在 | 每个节点必须有类型 |
| 8 | 边引用的节点存在 | source 和 target 必须指向已有节点 |
| 9 | 边的 sourceType 匹配节点类型 | edge.data.sourceType == node.data.type |
| 10 | LLM 节点有 prompt_template | 必须是 list 类型 |
| 11 | Code 节点有 def main | Python 必须定义 main 函数 |
| 12 | Code 节点有 outputs | 必须是 dict 类型 |
| 13 | Tool 节点必填字段 | provider_id, provider_name, provider_type, tool_name, tool_parameters |
| 14 | If-else 节点有 cases | 必须是 list 类型 |
| 15 | End 节点有 outputs | 必须是 list 类型 |
| 16 | 变量名唯一 | start/conversation/environment 变量不重复 |

## 警告项（Warning 级别）

| # | 规则 | 说明 |
|---|------|------|
| 1 | 顶层 kind 应为 'app' | 默认值 |
| 2 | LLM 节点有 model.provider 和 model.name | 缺失可能导入失败 |
| 3 | 边有 sourceHandle 和 targetHandle | 缺失可能导致连接异常 |
| 4 | 变量有 name 和 value_type | 缺失可能导入失败 |
| 5 | 变量引用指向已知节点 | `{{#node_id.field#}}` 中的 node_id 必须存在 |
| 6 | SQL 无尾部逗号 | `INSERT INTO t (a,) VALUES (1)` 语法错误 |
| 7 | SQL 无危险关键词 | DROP/TRUNCATE/ALTER |
| 8 | SQL 无变更关键词 | DELETE/UPDATE（需确认意图） |

## 使用方式

Claude Code 生成 DSL 后，按此 checklist 逐项检查：
1. 先检查 Error 级别（必须全部通过）
2. 再检查 Warning 级别（尽量通过，无法修复则提示用户）
3. 校验通过后交付 DSL；发现错误时自动修复后重新输出
```

- [ ] **Step 3: 验证规则数量**

```bash
grep -c "| [0-9]" .claude/skills/dify-workflow/references/validation-rules.md
```

Expected: >= 15

- [ ] **Step 4: Commit**

```bash
git add .claude/skills/dify-workflow/references/validation-rules.md
git commit -m "feat: create validation-rules.md with 16+ rules"
```

---

## Task 8: 创建 mcp-usage-guide.md（Mintlify MCP 使用指引）

**[FR-F7 来源] [无依赖] [出参验证：文件存在且包含两个工具说明]**

**Files:**
- Create: `.claude/skills/dify-workflow/references/mcp-usage-guide.md`

- [ ] **Step 1: 创建 mcp-usage-guide.md**

```markdown
# Mintlify MCP 使用指引

## 何时调用

| 场景 | 是否调用 | 理由 |
|------|---------|------|
| 节点不在 25+ Schema 列表中 | 调用 | Skill 未覆盖的节点需要查文档 |
| 用户 Dify 版本与 v0.6.0 不一致 | 调用 | 需要确认最新版本的差异 |
| 查询 Context RAG、Jinja2 等高级特性 | 调用 | Skill 中可能没有详细说明 |
| 确认节点配置的最新变更 | 调用 | Dify 快速迭代，Schema 可能过时 |
| 节点在 Schema 列表中且版本一致 | 不调用 | Skill 已覆盖，无需网络请求 |
| 生成标准工作流模板 | 不调用 | 模板已在 Skill 中 |

## 可用工具

### search_dify_docs

语义搜索 Dify 官方文档。

参数：
- `query`（必填）：搜索关键词或自然语言问题
- `version`（选填）：Dify 版本号，默认最新
- `language`（选填）：文档语言，默认 en

示例调用：
```
search_dify_docs(query="LLM node Context RAG configuration")
```

### query_docs_filesystem_dify_docs

虚拟文件系统直接读取文档页面。

参数：
- `command`（必填）：shell 命令（rg/cat/tree/head/grep）

示例调用：
```
query_docs_filesystem_dify_docs(command="cat /en/use-dify/nodes/llm.mdx")
query_docs_filesystem_dify_docs(command="rg 'variable' /en/use-dify/nodes/")
```

## 降级策略

当 Mintlify MCP 不可用时：
1. 使用 references/node-schemas.md 中的 Schema
2. 使用 references/templates/ 中的模板
3. 对于不确定的配置，返回 warning 提示用户手动确认
4. 不报错，确保 Skill 功能 100% 可用
```

- [ ] **Step 2: 验证两个工具都有说明**

```bash
grep -q "search_dify_docs" .claude/skills/dify-workflow/references/mcp-usage-guide.md && \
grep -q "query_docs_filesystem_dify_docs" .claude/skills/dify-workflow/references/mcp-usage-guide.md && \
echo "OK" || echo "FAIL"
```

Expected: OK

- [ ] **Step 3: Commit**

```bash
git add .claude/skills/dify-workflow/references/mcp-usage-guide.md
git commit -m "feat: create mcp-usage-guide.md with tool docs and fallback"
```

---

## Task 9: 创建 7 个工作流模板

**[FR-F5 来源] [依赖 Task 4] [出参验证：templates/ 目录下有 7 个 .yaml 文件]**

**Files:**
- Create: `.claude/skills/dify-workflow/references/templates/simple-llm.yaml`
- Create: `.claude/skills/dify-workflow/references/templates/if-else-branch.yaml`
- Create: `.claude/skills/dify-workflow/references/templates/http-code.yaml`
- Create: `.claude/skills/dify-workflow/references/templates/chatflow-multi-turn.yaml`
- Create: `.claude/skills/dify-workflow/references/templates/rag-retrieval.yaml`
- Create: `.claude/skills/dify-workflow/references/templates/iteration.yaml`
- Create: `.claude/skills/dify-workflow/references/templates/error-handling.yaml`

- [ ] **Step 1: 从 yzmw123 提取 4 个模板**

```bash
cat /tmp/yzmw123-skill/references/complete-examples.md
```

从 complete-examples.md 中提取以下模板的 YAML 内容：
- simple-llm.yaml（简单 LLM 对话）
- if-else-branch.yaml（条件分支）
- http-code.yaml（HTTP 请求 + 代码处理）
- chatflow-multi-turn.yaml（Chatflow 多轮对话）

- [ ] **Step 2: 创建 4 个基础模板**

每个模板必须包含完整的 DSL YAML 结构：
- `version: "0.6.0"`
- `kind: app`
- `app.mode`
- `workflow.graph.nodes`
- `workflow.graph.edges`

- [ ] **Step 3: 创建 3 个补充模板（rag-retrieval.yaml）**

```yaml
version: "0.6.0"
kind: app
app:
  name: "RAG Knowledge Retrieval"
  mode: "workflow"
workflow:
  graph:
    nodes:
      - id: "start_1"
        type: "custom"
        data:
          type: "start"
          title: "Start"
          variables: []
      - id: "knowledge_1"
        type: "custom"
        data:
          type: "knowledge-retrieval"
          title: "Knowledge Retrieval"
          dataset_ids: ["<DATASET_ID>"]
          query_variable: ["start_1", "query"]
      - id: "llm_1"
        type: "custom"
        data:
          type: "llm"
          title: "Answer with Context"
          model:
            provider: "langgenius/tongyi/tongyi"
            name: "qwen-max"
          prompt_template:
            - role: "system"
              text: "Answer based on the context below.\nContext: {{#knowledge_1.result#}}"
            - role: "user"
              text: "{{#start_1.query#}}"
      - id: "end_1"
        type: "custom"
        data:
          type: "end"
          title: "End"
          outputs:
            - variable: "answer"
              value_selector: ["llm_1", "text"]
    edges:
      - id: "e1"
        source: "start_1"
        target: "knowledge_1"
        sourceHandle: "source"
        targetHandle: "target"
      - id: "e2"
        source: "knowledge_1"
        target: "llm_1"
        sourceHandle: "source"
        targetHandle: "target"
      - id: "e3"
        source: "llm_1"
        target: "end_1"
        sourceHandle: "source"
        targetHandle: "target"
```

- [ ] **Step 4: 创建 iteration.yaml**

迭代处理列表的模板，包含 iteration 节点。

- [ ] **Step 5: 创建 error-handling.yaml**

错误处理模式模板，包含 try-catch 或 if-else 分支。

- [ ] **Step 6: 验证模板数量**

```bash
ls .claude/skills/dify-workflow/references/templates/*.yaml | wc -l
```

Expected: 7

- [ ] **Step 7: 验证每个模板是合法 YAML**

```bash
for f in .claude/skills/dify-workflow/references/templates/*.yaml; do
  python3 -c "import yaml; yaml.safe_load(open('$f'))" && echo "$f: OK" || echo "$f: FAIL"
done
```

Expected: 7 个 OK

- [ ] **Step 8: Commit**

```bash
git add .claude/skills/dify-workflow/references/templates/
git commit -m "feat: create 7 workflow templates (4 from yzmw123 + 3 new)"
```

---

## Task 10: 适配 validate_dsl.py 校验脚本

**[FR-F4 来源] [依赖 Task 1] [出参验证：脚本可执行且通过自身测试]**

**Files:**
- Create: `.claude/skills/dify-workflow/scripts/validate_dsl.py`

- [ ] **Step 1: 读取 yzmw123 的 validate_dsl.py**

```bash
cat /tmp/yzmw123-skill/scripts/validate_dsl.py
```

- [ ] **Step 2: 复制到目标位置**

```bash
cp /tmp/yzmw123-skill/scripts/validate_dsl.py .claude/skills/dify-workflow/scripts/validate_dsl.py
```

- [ ] **Step 3: 安装 PyYAML 依赖**

```bash
source .venv/bin/activate && pip install pyyaml
```

- [ ] **Step 4: 验证脚本可执行**

```bash
python3 .claude/skills/dify-workflow/scripts/validate_dsl.py --help
```

Expected: 显示 argparse 帮助信息

- [ ] **Step 5: 用模板测试校验脚本**

```bash
python3 .claude/skills/dify-workflow/scripts/validate_dsl.py .claude/skills/dify-workflow/references/templates/*.yaml
```

Expected: 每个模板输出 OK 或仅有 warning（无 error）

- [ ] **Step 6: Commit**

```bash
git add .claude/skills/dify-workflow/scripts/validate_dsl.py
git commit -m "feat: adapt validate_dsl.py from yzmw123"
```

---

## Task 11: 集成测试 — 生成 3 个实际工作流

**[FR-US-1/US-2/US-3 来源] [依赖 Task 2-10] [出参验证：3 个 YAML 文件均通过 validate_dsl.py]**

**Files:**
- Create: `.claude/skills/dify-workflow/examples/rag-workflow.yaml`
- Create: `.claude/skills/dify-workflow/examples/conditional-branch.yaml`
- Create: `.claude/skills/dify-workflow/examples/chatflow-multi-turn.yaml`

- [ ] **Step 1: 创建 rag-workflow.yaml**

基于 templates/rag-retrieval.yaml，扩展为完整的工作流：
- start → knowledge-retrieval → llm → end
- 包含完整的变量引用 `{{#knowledge_1.result#}}`
- 包含 conversation_variables
- 包含 dependencies

- [ ] **Step 2: 创建 conditional-branch.yaml**

基于 templates/if-else-branch.yaml，扩展为完整的工作流：
- start → llm(classifier) → if-else → [branch_a: llm, branch_b: http] → end
- 包含完整的 sourceHandle（true/false）
- 包含环境变量

- [ ] **Step 3: 创建 chatflow-multi-turn.yaml**

基于 templates/chatflow-multi-turn.yaml，扩展为完整的工作流：
- start → llm → answer
- 包含 conversation_variables（history）
- 包含 memory 配置

- [ ] **Step 4: 用 validate_dsl.py 校验所有示例**

```bash
python3 .claude/skills/dify-workflow/scripts/validate_dsl.py .claude/skills/dify-workflow/examples/*.yaml
```

Expected: 每个文件输出 OK 或仅有 warning（无 error）

- [ ] **Step 5: Commit**

```bash
git add .claude/skills/dify-workflow/examples/
git commit -m "feat: add 3 integration test workflow examples"
```

---

## Task 12: 更新 CLAUDE.md 项目结构

**[FR-文档 来源] [依赖 Task 2-11] [出参验证：CLAUDE.md 包含 Skill 目录说明]**

**Files:**
- Modify: `CLAUDE.md`

- [ ] **Step 1: 更新 CLAUDE.md 的项目结构段落**

在 CLAUDE.md 的项目结构中，将 `.claude/skills/dify-workflow/` 展开说明：

```markdown
## 项目结构

- `.claude/skills/dify-workflow/` — 核心 Skill
  - `SKILL.md` — 主 Skill 文件（基于 yzmw123 适配）
  - `references/` — 参考文件（Schema、语法、校验、模板）
  - `scripts/validate_dsl.py` — DSL 校验脚本
  - `examples/` — 集成测试示例
```

- [ ] **Step 2: Commit**

```bash
git add CLAUDE.md
git commit -m "docs: update CLAUDE.md with Skill directory structure"
```

---

## 并行/串行依赖图

```
Task 1 (克隆 yzmw123)
  ├── Task 2 (SKILL.md)          ─┐
  ├── Task 3 (node-schemas.md)    │ 可并行
  ├── Task 4 (dsl-structure.md)   │
  ├── Task 5 (variable-syntax.md) │
  ├── Task 7 (validation-rules)   │
  └── Task 10 (validate_dsl.py)  ─┘
                                  │
Task 6 (node-output-fields.md) ───┤ 可并行（无依赖）
Task 8 (mcp-usage-guide.md) ─────┘
                                  │
                                  ▼
Task 9 (7 templates) ─────────── 依赖 Task 4
                                  │
                                  ▼
Task 11 (集成测试) ──────────── 依赖 Task 2-10
                                  │
                                  ▼
Task 12 (更新 CLAUDE.md) ────── 依赖 Task 2-11
```

**可并行的任务组：**
- Group A（依赖 Task 1）：Task 2, 3, 4, 5, 7, 10
- Group B（无依赖）：Task 6, 8
- Group C（依赖 Group A）：Task 9
- Group D（依赖全部）：Task 11, 12
