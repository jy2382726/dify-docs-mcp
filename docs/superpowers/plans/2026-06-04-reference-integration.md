# yzmw123 Reference 文件整合实施计划

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 将 yzmw123 的 5 个 reference 文件引入当前项目，重写 SKILL.md 为路由层，消除内容重复，建立 3 层按需加载体系。

**Architecture:** SKILL.md 从"路由+知识"混合体改为纯路由层（< 250 行），reference 文件按 3 层组织（始终加载 / 按需加载 / 特定场景加载），每个文件有明确触发条件。

**Tech Stack:** Claude Code Skill（纯 Markdown）、Python 3.11（validate_dsl.py 校验）

---

## 文件结构

```
.claude/skills/dify-workflow/
├── SKILL.md                          # 路由层（~230 行）— Task 7 重写
├── references/
│   ├── dsl-structure.md              # 第 1 层：始终加载（已有，不改）
│   ├── node-schemas.md               # 第 1 层：始终加载（已有，不改）
│   ├── usecase-node-selection.md     # 第 2 层：业务需求时加载 — Task 2 引入+改造
│   ├── variable-syntax.md            # 第 2 层：写变量时加载（已有，不改）
│   ├── node-output-fields.md         # 第 2 层：引用输出时加载（已有，不改）
│   ├── database-tools.md             # 第 3 层：数据库场景 — Task 3 引入+改造
│   ├── plugin-marketplace-tools.md   # 第 3 层：插件工具场景 — Task 4 引入+改造
│   ├── official-0.6-target.md        # 第 3 层：确认规范时 — Task 5 引入+改造
│   ├── real-world-yml-study.md       # 第 3 层：参考真实模式 — Task 6 引入+改造
│   ├── validation-rules.md           # 第 3 层：校验时加载（已有，不改）
│   ├── mcp-usage-guide.md            # 第 3 层：MCP 相关（已有，不改）
│   └── templates/                    # 第 3 层：需要模板时（已有，不改）
│       ├── simple-llm.yaml
│       ├── if-else-branch.yaml
│       ├── http-code.yaml
│       ├── chatflow-multi-turn.yaml
│       ├── rag-retrieval.yaml
│       ├── iteration.yaml
│       └── error-handling.yaml
├── scripts/
│   └── validate_dsl.py               # 校验脚本（已有，不改）
└── examples/                         # 集成测试示例（已有，不改）
    ├── rag-workflow.yaml
    ├── conditional-branch.yaml
    └── chatflow-multi-turn.yaml
```

---

## Task 1: 复制 5 个 yzmw123 reference 文件到临时中转区

**[无依赖] [出参验证：5 个文件存在于 /tmp/yzmw123-refs/]**

**Files:**
- Create: `/tmp/yzmw123-refs/database-tools.md`
- Create: `/tmp/yzmw123-refs/official-0.6-target.md`
- Create: `/tmp/yzmw123-refs/plugin-marketplace-tools.md`
- Create: `/tmp/yzmw123-refs/real-world-yml-study.md`
- Create: `/tmp/yzmw123-refs/usecase-node-selection.md`

- [ ] **Step 1: 创建中转目录并复制文件**

```bash
mkdir -p /tmp/yzmw123-refs
cp /tmp/yzmw123-skill/references/database-tools.md /tmp/yzmw123-refs/
cp /tmp/yzmw123-skill/references/official-0.6-target.md /tmp/yzmw123-refs/
cp /tmp/yzmw123-skill/references/plugin-marketplace-tools.md /tmp/yzmw123-refs/
cp /tmp/yzmw123-skill/references/real-world-yml-study.md /tmp/yzmw123-refs/
cp /tmp/yzmw123-skill/references/usecase-node-selection.md /tmp/yzmw123-refs/
```

- [ ] **Step 2: 验证文件完整性**

```bash
for f in /tmp/yzmw123-refs/*.md; do echo "$(basename $f): $(wc -l < $f) lines"; done
```

Expected: 5 个文件，每个 > 50 行

- [ ] **Step 3: 无需 commit（临时文件）**

---

## Task 2: 引入并改造 usecase-node-selection.md

**[依赖 Task 1] [出参验证：文件存在且包含中文标题和 SKILL.md 下沉内容]**

**Files:**
- Create: `.claude/skills/dify-workflow/references/usecase-node-selection.md`

**改造要点：**
- 保留 yzmw123 原文的全部内容（模式选择表、触发器选择表、12 种业务模式、节点选择启发式、文件/集成工作流、可靠性规则）
- 在文件顶部添加中文标题和说明
- 合并 SKILL.md 下沉的内容：New DSL Intake 的 Mode/Trigger 详细说明、Required Decisions 的 Mode 决策

- [ ] **Step 1: 读取 yzmw123 原文**

```bash
cat /tmp/yzmw123-refs/usecase-node-selection.md
```

- [ ] **Step 2: 读取 SKILL.md 中需要下沉的内容**

需要从 SKILL.md 下沉到此文件的内容：
- New DSL Intake 段落（第 43-56 行）的 Mode 和 Trigger 详细说明
- Required Decisions 段落（第 58-74 行）的 Mode 决策详细说明

- [ ] **Step 3: 创建改造后的文件**

文件结构：
```markdown
# 用例与节点选择

收到用户的业务需求时，加载此文件确定工作流模式、触发方式和节点组合。

## 模式选择

[保留 yzmw123 的 Mode Selection 表，补充 SKILL.md 下沉的 Mode 决策内容]

## 触发器选择

[保留 yzmw123 的 Trigger Selection 表]

## 常见业务模式

[保留 yzmw123 的 12 种 Common Business Patterns]

## 节点选择启发式

[保留 yzmw123 的 Node Choice Heuristics]

## 文件与文档工作流

[保留 yzmw123 的 File And Document Workflows]

## 集成工作流

[保留 yzmw123 的 Integration Workflows]

## 可靠性规则

[保留 yzmw123 的 Reliability Rules]
```

- [ ] **Step 4: 验证文件内容**

```bash
grep -q "模式选择" .claude/skills/dify-workflow/references/usecase-node-selection.md && \
grep -q "触发器选择" .claude/skills/dify-workflow/references/usecase-node-selection.md && \
grep -q "常见业务模式" .claude/skills/dify-workflow/references/usecase-node-selection.md && \
echo "OK" || echo "FAIL"
```

Expected: OK

- [ ] **Step 5: Commit**

```bash
git add .claude/skills/dify-workflow/references/usecase-node-selection.md
git commit -m "feat: 引入 usecase-node-selection.md（用例与节点选择）"
```

---

## Task 3: 引入并改造 database-tools.md

**[依赖 Task 1] [出参验证：文件存在且包含中文标题]**

**Files:**
- Create: `.claude/skills/dify-workflow/references/database-tools.md`

**改造要点：**
- 保留 yzmw123 原文全部内容（两种插件模式、INSERT/SELECT 模板、重复检查、NL2SQL、SQL 安全清单）
- 在文件顶部添加中文标题和说明

- [ ] **Step 1: 读取 yzmw123 原文**

```bash
cat /tmp/yzmw123-refs/database-tools.md
```

- [ ] **Step 2: 创建改造后的文件**

在原文顶部添加中文标题：
```markdown
# 数据库工具节点

当工作流涉及数据库读写操作时，加载此文件获取工具节点模板和安全规范。

## 适用场景

- 用户需求涉及 PostgreSQL/SQL 数据库
- 需要构建 NL2SQL 检索模式
- 需要数据库写入（INSERT）或读取（SELECT）模板

---

[以下保留 yzmw123 原文全部内容，包括：]
[Two Useful Plugin Patterns]
[spance/db_client_node Insert Template]
[spance/db_client_node Read Template]
[Duplicate Check Pattern]
[hjlarry/database SQL Execute Template]
[NL2SQL Retrieval Pattern]
[SQL Safety Checklist]
```

- [ ] **Step 3: 验证文件内容**

```bash
grep -q "数据库工具节点" .claude/skills/dify-workflow/references/database-tools.md && \
grep -q "SQL Safety Checklist" .claude/skills/dify-workflow/references/database-tools.md && \
echo "OK" || echo "FAIL"
```

Expected: OK

- [ ] **Step 4: Commit**

```bash
git add .claude/skills/dify-workflow/references/database-tools.md
git commit -m "feat: 引入 database-tools.md（数据库工具节点）"
```

---

## Task 4: 引入并改造 plugin-marketplace-tools.md

**[依赖 Task 1] [出参验证：文件存在且包含中文标题]**

**Files:**
- Create: `.claude/skills/dify-workflow/references/plugin-marketplace-tools.md`

**改造要点：**
- 保留 yzmw123 原文全部内容（可靠性梯度、工具节点模板、paramSchemas、授权处理、何时拒绝保证）
- 在文件顶部添加中文标题和说明
- 补充"适用场景"说明，与 SKILL.md 的降级策略对齐

- [ ] **Step 1: 读取 yzmw123 原文**

```bash
cat /tmp/yzmw123-refs/plugin-marketplace-tools.md
```

- [ ] **Step 2: 创建改造后的文件**

在原文顶部添加中文标题：
```markdown
# 插件市场工具节点

当工作流需要使用非内置工具（插件、MCP、workflow-provider）时，加载此文件获取工具节点模板和决策逻辑。

## 适用场景

- 用户需要的工具不在 node-schemas.md 的内置列表中
- 需要从 Dify Marketplace 添加插件工具
- 需要评估工具节点的可靠性
- 需要处理插件授权和密钥

---

[以下保留 yzmw123 原文全部内容，包括：]
[Reliability Ladder]
[Preferred Workflow For New Tools]
[Tool Node Identity Template]
[paramSchemas Guidance]
[Authorization And Secrets]
[When To Refuse A Guarantee]
```

- [ ] **Step 3: 验证文件内容**

```bash
grep -q "插件市场工具节点" .claude/skills/dify-workflow/references/plugin-marketplace-tools.md && \
grep -q "Reliability Ladder" .claude/skills/dify-workflow/references/plugin-marketplace-tools.md && \
echo "OK" || echo "FAIL"
```

Expected: OK

- [ ] **Step 4: Commit**

```bash
git add .claude/skills/dify-workflow/references/plugin-marketplace-tools.md
git commit -m "feat: 引入 plugin-marketplace-tools.md（插件市场工具节点）"
```

---

## Task 5: 引入并改造 official-0.6-target.md

**[依赖 Task 1] [出参验证：文件存在且包含中文标题]**

**Files:**
- Create: `.claude/skills/dify-workflow/references/official-0.6-target.md`

**改造要点：**
- 保留 yzmw123 原文全部内容（源码分析、目标规则、导入兼容、导出结构、依赖类型、官方节点枚举、公共样本可用性）
- 在文件顶部添加中文标题和说明
- 补充"适用场景"说明

- [ ] **Step 1: 读取 yzmw123 原文**

```bash
cat /tmp/yzmw123-refs/official-0.6-target.md
```

- [ ] **Step 2: 创建改造后的文件**

在原文顶部添加中文标题：
```markdown
# 官方 DSL 0.6.0 规范

需要确认 Dify 官方 DSL 规范时加载此文件。包含从 Dify 源码提取的权威信息。

## 适用场景

- 需要确认官方导出结构（version、kind、app、workflow 字段）
- 需要确认版本兼容性规则
- 需要查询官方节点类型完整枚举（30 个）
- 需要确认依赖类型（marketplace/package/github）
- 需要确认输入变量类型列表

---

[以下保留 yzmw123 原文全部内容，包括：]
[Sources Checked]
[Target Rule]
[Import And Version Compatibility]
[Export Shape]
[Export Sanitization]
[Dependency Types]
[Dependency Sources]
[Official Node Type Set]
[Common Node And Edge Metadata]
[Input Variable Types]
[Public Sample Availability]
```

- [ ] **Step 3: 验证文件内容**

```bash
grep -q "官方 DSL 0.6.0 规范" .claude/skills/dify-workflow/references/official-0.6-target.md && \
grep -q "Official Node Type Set" .claude/skills/dify-workflow/references/official-0.6-target.md && \
echo "OK" || echo "FAIL"
```

Expected: OK

- [ ] **Step 4: Commit**

```bash
git add .claude/skills/dify-workflow/references/official-0.6-target.md
git commit -m "feat: 引入 official-0.6-target.md（官方 DSL 0.6.0 规范）"
```

---

## Task 6: 引入并改造 real-world-yml-study.md

**[依赖 Task 1] [出参验证：文件存在且包含中文标题]**

**Files:**
- Create: `.claude/skills/dify-workflow/references/real-world-yml-study.md`

**改造要点：**
- 保留 yzmw123 原文全部内容（262 个 DSL 分析、版本分布、工具节点现实、触发器现实、业务工作流现实、依赖现实、画布节点、生成器经验、规则修正）
- 在文件顶部添加中文标题和说明
- 补充"适用场景"说明

- [ ] **Step 1: 读取 yzmw123 原文**

```bash
cat /tmp/yzmw123-refs/real-world-yml-study.md
```

- [ ] **Step 2: 创建改造后的文件**

在原文顶部添加中文标题：
```markdown
# 真实 YAML 文件研究

需要参考真实 Dify 工作流模式时加载此文件。基于 262 个公开 DSL 文件的分析结果。

## 适用场景

- 生成的工作流需要与真实世界对齐
- 需要了解实际导出的工具节点格式
- 需要参考触发器和集成工作流模式
- 需要了解公共 DSL 的版本分布

---

[以下保留 yzmw123 原文全部内容，包括：]
[Corpus Checked]
[Detailed Samples]
[Observed Version And Mode Reality]
[Tool Node Reality]
[Trigger And Integration Reality]
[Business Workflow Reality]
[Generator Project Lessons]
[Dependency Reality]
[Canvas And Helper Nodes]
[Rule Corrections For This Skill]
```

- [ ] **Step 3: 验证文件内容**

```bash
grep -q "真实 YAML 文件研究" .claude/skills/dify-workflow/references/real-world-yml-study.md && \
grep -q "Rule Corrections" .claude/skills/dify-workflow/references/real-world-yml-study.md && \
echo "OK" || echo "FAIL"
```

Expected: OK

- [ ] **Step 4: Commit**

```bash
git add .claude/skills/dify-workflow/references/real-world-yml-study.md
git commit -m "feat: 引入 real-world-yml-study.md（真实 YAML 文件研究）"
```

---

## Task 7: 重写 SKILL.md 为路由层

**[依赖 Task 2-6] [出参验证：SKILL.md < 260 行且包含 3 层 Reference Map]**

**Files:**
- Modify: `.claude/skills/dify-workflow/SKILL.md`

**改造要点：**
1. 保留不变的段落：Core Workflow（精简步骤 2/6/7）、Validation Checklist、Mintlify MCP、降级策略、Useful Commands
2. 精简的段落：New DSL Intake（5 行快速清单）、Required Decisions（3 行核心决策）、Authoring Rules（10 条核心规则）
3. 重写 Reference Map 为 3 层条件触发式索引
4. 新增"加载策略"段落
5. 目标行数：< 260 行

- [ ] **Step 1: 读取当前 SKILL.md**

```bash
cat .claude/skills/dify-workflow/SKILL.md
```

- [ ] **Step 2: 创建重写后的 SKILL.md**

完整内容如下：

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
```

- [ ] **Step 3: 验证 SKILL.md 行数**

```bash
wc -l .claude/skills/dify-workflow/SKILL.md
```

Expected: < 260 行

- [ ] **Step 4: 验证 SKILL.md 中引用的所有 reference 文件存在**

```bash
for ref in dsl-structure.md node-schemas.md usecase-node-selection.md variable-syntax.md \
  node-output-fields.md database-tools.md plugin-marketplace-tools.md official-0.6-target.md \
  real-world-yml-study.md validation-rules.md mcp-usage-guide.md; do
  [ -f ".claude/skills/dify-workflow/references/$ref" ] && echo "$ref: OK" || echo "$ref: MISSING"
done
ls .claude/skills/dify-workflow/references/templates/*.yaml > /dev/null && echo "templates/: OK" || echo "templates/: MISSING"
```

Expected: 全部 OK

- [ ] **Step 5: 验证 SKILL.md 包含 frontmatter 和 3 层 Reference Map**

```bash
head -5 .claude/skills/dify-workflow/SKILL.md | grep -q "name: dify-workflow-dsl" && \
grep -q "第 1 层" .claude/skills/dify-workflow/SKILL.md && \
grep -q "第 2 层" .claude/skills/dify-workflow/SKILL.md && \
grep -q "第 3 层" .claude/skills/dify-workflow/SKILL.md && \
grep -q "加载策略" .claude/skills/dify-workflow/SKILL.md && \
echo "OK" || echo "FAIL"
```

Expected: OK

- [ ] **Step 6: 用 validate_dsl.py 校验所有模板和示例**

```bash
python3 .claude/skills/dify-workflow/scripts/validate_dsl.py \
  .claude/skills/dify-workflow/references/templates/*.yaml \
  .claude/skills/dify-workflow/examples/*.yaml
```

Expected: 全部 OK

- [ ] **Step 7: Commit**

```bash
git add .claude/skills/dify-workflow/SKILL.md
git commit -m "feat: 重写 SKILL.md 为路由层（3 层 Reference Map + 加载策略）"
```

---

## Task 8: 更新 CLAUDE.md 项目结构

**[依赖 Task 2-7] [出参验证：CLAUDE.md 包含新增的 reference 文件说明]**

**Files:**
- Modify: `CLAUDE.md`

- [ ] **Step 1: 读取当前 CLAUDE.md**

- [ ] **Step 2: 更新 references/ 目录说明**

将 references/ 部分更新为：

```markdown
  - `references/` — 参考文件（按 3 层组织）
    - 第 1 层（始终加载）：`dsl-structure.md`、`node-schemas.md`
    - 第 2 层（按需加载）：`usecase-node-selection.md`、`variable-syntax.md`、`node-output-fields.md`
    - 第 3 层（特定场景）：`database-tools.md`、`plugin-marketplace-tools.md`、`official-0.6-target.md`、`real-world-yml-study.md`、`validation-rules.md`、`mcp-usage-guide.md`、`templates/`
```

- [ ] **Step 3: Commit**

```bash
git add CLAUDE.md
git commit -m "docs: 更新 CLAUDE.md 的 reference 文件说明（3 层组织）"
```

---

## 依赖图

```
Task 1（复制 5 个文件到中转区）
  ├── Task 2（usecase-node-selection.md）
  ├── Task 3（database-tools.md）
  ├── Task 4（plugin-marketplace-tools.md）
  ├── Task 5（official-0.6-target.md）
  └── Task 6（real-world-yml-study.md）
                                       │
                                       ▼
                              Task 7（重写 SKILL.md）
                                       │
                                       ▼
                              Task 8（更新 CLAUDE.md）
```

**可并行的任务组：**
- Group A（依赖 Task 1）：Task 2, 3, 4, 5, 6（可并行）
- Group B（依赖 Group A）：Task 7
- Group C（依赖 Group B）：Task 8
