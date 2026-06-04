# yzmw123 Reference 文件整合设计文档

## 背景

当前项目从 yzmw123 仓库适配了 2 个 reference 文件（node-schemas.md、dsl-structure.md），并新增了 5 个 reference 文件。但 yzmw123 仓库还有 5 个高价值 reference 文件未被引入：

| 文件 | 内容 | 核心价值 |
|------|------|---------|
| `database-tools.md` | PostgreSQL 工具节点模板、NL2SQL 模式、SQL 安全清单 | 数据库工作流的实操模板 |
| `official-0.6-target.md` | Dify 源码分析、导出结构、版本兼容、官方节点枚举 | DSL 生成的权威依据 |
| `plugin-marketplace-tools.md` | 插件可靠性梯度、工具节点模板、授权处理 | 插件工具的决策逻辑 |
| `real-world-yml-study.md` | 262 个公开 DSL 分析、模式统计、经验总结 | 真实世界的经验校准 |
| `usecase-node-selection.md` | 业务需求→模式/节点选择、12 种业务模式 | 需求→工作流的决策矩阵 |

## 问题

盲目将 5 个文件复制到 references/ 会导致：

1. **Reference Map 膨胀** — 从 7 项增长到 12 项，agent 需要加载更多上下文
2. **内容重复** — 新文件与 SKILL.md 现有内容有大量重叠（模式选择、工具节点、SQL 安全等）
3. **决策路径模糊** — agent 不知道何时该读哪个文件，可能同时加载所有文件或遗漏关键文件
4. **SKILL.md 定位模糊** — 目前 SKILL.md 既是入口又是规则手册，引入更多文件后需要明确分层

## 设计目标

1. SKILL.md 保持精简（< 250 行），作为**路由层**而非知识库
2. 每个 reference 文件有明确的**触发条件**，agent 知道何时加载
3. 消除 SKILL.md 与 reference 文件之间的内容重复
4. 新引入的文件与现有文件形成**互补**而非竞争关系

## 当前 SKILL.md 结构分析

现有 SKILL.md（213 行）包含 10 个段落：

| 段落 | 行数 | 职责 | 与新文件的关系 |
|------|------|------|---------------|
| Core Workflow | 25 | 8 步生成流程 | 无冲突 |
| New DSL Intake | 14 | 需求收集清单 | 与 usecase-node-selection 重叠 |
| Required Decisions | 17 | 关键决策点 | 与 usecase-node-selection 重叠 |
| Authoring Rules | 31 | YAML 编写规则 | 与 official-0.6-target 重叠 |
| Validation Checklist | 21 | 校验清单 | 无冲突 |
| Reference Map | 15 | 文件索引 | 需要扩展 |
| Mintlify MCP | 35 | MCP 使用指引 | 无冲突 |
| 降级策略 | 25 | MCP 不可用时的回退 | 无冲突 |
| Useful Commands | 7 | 命令行 | 无冲突 |

**重叠区域**：
- SKILL.md "New DSL Intake" 的 Mode/Trigger/Inputs/Output/Shape 与 `usecase-node-selection.md` 的模式选择表重复
- SKILL.md "Required Decisions" 的 6 个决策点与 `usecase-node-selection.md` 的决策矩阵重复
- SKILL.md "Authoring Rules" 的依赖/工具节点规则与 `official-0.6-target.md` 和 `plugin-marketplace-tools.md` 重复
- SKILL.md 的 SQL 安全提及与 `database-tools.md` 的 SQL 安全清单重复

## 设计方案

### 核心思路：SKILL.md 做路由，reference 做知识

将 SKILL.md 从"路由+知识"混合体改为**纯路由层**：
- 保留流程骨架（Core Workflow、Validation Checklist、Mintlify MCP、降级策略）
- 将知识性内容（模式选择、节点规则、工具模板）下沉到 reference 文件
- Reference Map 改为**条件触发式索引**，而非全量加载

### Reference Map 重新设计

将 reference 文件分为 3 层：

**第 1 层：始终加载（基础层）**
- `dsl-structure.md` — DSL 顶层结构，每次生成都要参考
- `node-schemas.md` — 节点 Schema，每次生成都要参考

**第 2 层：按需加载（决策层）**
- `usecase-node-selection.md` — 收到业务需求时加载，用于选择模式和节点组合
- `variable-syntax.md` — 写变量引用时加载
- `node-output-fields.md` — 引用节点输出时加载

**第 3 层：特定场景加载（专家层）**
- `database-tools.md` — 涉及数据库节点时加载
- `plugin-marketplace-tools.md` — 使用非内置工具时加载
- `official-0.6-target.md` — 需要确认官方规范时加载
- `real-world-yml-study.md` — 需要参考真实工作流模式时加载
- `validation-rules.md` — 校验 DSL 时加载
- `mcp-usage-guide.md` — MCP 不可用或需要查文档时加载
- `templates/` — 需要模板参考时加载

### SKILL.md 改造方案

#### 删除的段落（下沉到 reference）

| 删除的段落 | 下沉目标 | 原因 |
|-----------|---------|------|
| New DSL Intake 的 Mode 详细说明 | `usecase-node-selection.md` Mode Selection 表 | usecase-node-selection 更全面 |
| New DSL Intake 的 Trigger 详细说明 | `usecase-node-selection.md` Trigger Selection 表 | 同上 |
| Required Decisions 的 Mode/Model/Secrets 详细说明 | `usecase-node-selection.md` + `plugin-marketplace-tools.md` | 分散到各自专业文件 |
| Authoring Rules 的依赖/工具节点详细规则 | `official-0.6-target.md` + `plugin-marketplace-tools.md` | 源码级分析更权威 |

#### 保留的段落（精简）

| 保留的段落 | 改动 |
|-----------|------|
| Core Workflow | 保留，但步骤 2 改为"加载 usecase-node-selection.md 确定模式" |
| New DSL Intake | 精简为 5 行快速清单，详细内容指向 usecase-node-selection.md |
| Required Decisions | 精简为 3 行核心决策，详细内容指向 reference 文件 |
| Authoring Rules | 精简为 10 条核心规则，详细规则指向 reference 文件 |
| Validation Checklist | 保留不变 |
| Reference Map | 重写为 3 层条件触发式索引 |
| Mintlify MCP | 保留不变 |
| 降级策略 | 保留不变 |
| Useful Commands | 保留不变 |

#### 新增的段落

| 新增的段落 | 内容 |
|-----------|------|
| 加载策略 | 说明 3 层加载逻辑，agent 何时读哪个文件 |

### Reference 文件改造方案

#### 需要改造的文件

| 文件 | 改造内容 |
|------|---------|
| `usecase-node-selection.md` | 合并 SKILL.md 下沉的 Mode/Trigger 内容，补充中文说明 |
| `plugin-marketplace-tools.md` | 合并 SKILL.md 下沉的工具节点规则，补充中文说明 |
| `official-0.6-target.md` | 合并 SKILL.md 下沉的 Authoring Rules 中的官方规范部分 |
| `database-tools.md` | 保持原样，仅补充中文标题 |
| `real-world-yml-study.md` | 保持原样，仅补充中文标题 |

#### 不需要改造的文件

| 文件 | 原因 |
|------|------|
| `node-schemas.md` | 已完整 |
| `dsl-structure.md` | 已完整 |
| `variable-syntax.md` | 已完整 |
| `node-output-fields.md` | 已完整 |
| `validation-rules.md` | 已完整 |
| `mcp-usage-guide.md` | 已完整 |
| `templates/*` | 已完整 |

## 最终文件结构

```
.claude/skills/dify-workflow/
├── SKILL.md                          # 路由层（~200 行）
├── references/
│   ├── dsl-structure.md              # 第 1 层：始终加载
│   ├── node-schemas.md               # 第 1 层：始终加载
│   ├── usecase-node-selection.md     # 第 2 层：业务需求时加载 [新引入]
│   ├── variable-syntax.md            # 第 2 层：写变量时加载
│   ├── node-output-fields.md         # 第 2 层：引用输出时加载
│   ├── database-tools.md             # 第 3 层：数据库场景 [新引入]
│   ├── plugin-marketplace-tools.md   # 第 3 层：插件工具场景 [新引入]
│   ├── official-0.6-target.md        # 第 3 层：确认规范时 [新引入]
│   ├── real-world-yml-study.md       # 第 3 层：参考真实模式时 [新引入]
│   ├── validation-rules.md           # 第 3 层：校验时加载
│   ├── mcp-usage-guide.md            # 第 3 层：MCP 相关
│   └── templates/                    # 第 3 层：需要模板时
│       ├── simple-llm.yaml
│       ├── if-else-branch.yaml
│       ├── http-code.yaml
│       ├── chatflow-multi-turn.yaml
│       ├── rag-retrieval.yaml
│       ├── iteration.yaml
│       └── error-handling.yaml
├── scripts/
│   └── validate_dsl.py               # 校验脚本
└── examples/                         # 集成测试示例
    ├── rag-workflow.yaml
    ├── conditional-branch.yaml
    └── chatflow-multi-turn.yaml
```

## 实施步骤

### Step 1: 复制 5 个 reference 文件

从 yzmw123 复制到 `references/`，保留原文件名。

### Step 2: 改造 usecase-node-selection.md

- 添加中文标题和说明
- 合并 SKILL.md 下沉的 Mode/Trigger 详细内容
- 确保与 SKILL.md 的精简版不矛盾

### Step 3: 改造 plugin-marketplace-tools.md

- 添加中文标题和说明
- 合并 SKILL.md 下沉的工具节点规则
- 确保可靠性梯度与 SKILL.md 的降级策略一致

### Step 4: 改造 official-0.6-target.md

- 添加中文标题和说明
- 合并 SKILL.md Authoring Rules 中的官方规范部分

### Step 5: 改造 database-tools.md 和 real-world-yml-study.md

- 添加中文标题和说明
- 保持内容原样

### Step 6: 重写 SKILL.md

- 精简 New DSL Intake、Required Decisions、Authoring Rules
- 重写 Reference Map 为 3 层条件触发式索引
- 新增"加载策略"段落
- 确保总行数 < 250

### Step 7: 校验

- 验证 SKILL.md 中的所有 reference 路径都存在
- 验证 SKILL.md 行数 < 250
- 验证 reference 文件之间的交叉引用一致

## 风险

| 风险 | 缓解措施 |
|------|---------|
| agent 加载过多 reference 导致上下文溢出 | 3 层加载策略，按需加载 |
| SKILL.md 精简后丢失重要规则 | 确保下沉到 reference 的内容完整 |
| 新旧内容矛盾 | Step 2-4 中逐一比对消除矛盾 |
| agent 不理解加载策略 | 在 SKILL.md 中用明确的 if-then 规则 |

## 实施结果（2026-06-04）

### 完成状态

全部 8 个 Task 已完成，两轮评估修复已执行。

### 最终文件结构

```
.claude/skills/dify-workflow/
├── SKILL.md                          # 路由层（297 行）
├── references/
│   ├── dsl-structure.md              # 第 1 层：始终加载（325 行）
│   ├── node-schemas.md               # 第 1 层：始终加载（741 行）
│   ├── usecase-node-selection.md     # 第 2 层：业务需求时加载（95 行）
│   ├── variable-syntax.md            # 第 2 层：写变量时加载（54 行）
│   ├── node-output-fields.md         # 第 2 层：引用输出时加载（98 行，18 种节点）
│   ├── database-tools.md             # 第 3 层：数据库场景（270 行）
│   ├── plugin-marketplace-tools.md   # 第 3 层：插件工具场景（137 行）
│   ├── official-0.6-target.md        # 第 3 层：确认规范时（184 行）
│   ├── real-world-yml-study.md       # 第 3 层：参考真实模式（217 行）
│   ├── validation-rules.md           # 第 3 层：校验时加载（74 行，含 16 行修复表）
│   ├── mcp-usage-guide.md            # 第 3 层：MCP 相关（72 行，含降级判断）
│   └── templates/                    # 第 3 层：需要模板时（7 个文件）
│       ├── simple-llm.yaml           # 121 行
│       ├── if-else-branch.yaml       # 233 行
│       ├── http-code.yaml            # 154 行
│       ├── chatflow-multi-turn.yaml  # 245 行
│       ├── rag-retrieval.yaml        # 152 行
│       ├── iteration.yaml            # 224 行
│       └── error-handling.yaml       # 258 行
├── scripts/
│   └── validate_dsl.py               # 校验脚本（355 行）
└── examples/                         # 集成测试示例（3 个文件）
    ├── rag-workflow.yaml             # 165 行
    ├── conditional-branch.yaml       # 260 行
    └── chatflow-multi-turn.yaml      # 196 行
```

总计：22 个文件（12 个 .md + 10 个 .yaml + 1 个 .py）

### 质量指标

| 指标 | 结果 |
|------|------|
| YAML 校验（examples + templates） | 10/10 OK |
| SKILL.md 行数 | 297 行（< 500 上限） |
| 非代码中文占比（reference 文件） | 10/12 > 70%，2 个因节点标题拉低但内容已全中文 |
| 节点 Schema 覆盖 | 26 种节点类型 |
| 节点输出字段覆盖 | 18 种节点类型 |
| 模板覆盖 | 7 种工作流模式 |
| 示例覆盖 | 3 种典型场景（RAG、Chatflow、条件分支） |

### 设计偏差记录

| 设计预期 | 实际情况 | 原因 |
|---------|---------|------|
| SKILL.md < 250 行 | 实际 297 行 | 添加了最小端到端示例（~100 行）和 examples 索引表 |
| node-output-fields.md 14 种节点 | 实际 18 种 | 补充了 template-transform、question-classifier、list-operator、assigner |
| 无评估修复环节 | 经历两轮评估修复 | skill-creator 评估发现语言合规性、冗余、示例一致性问题 |
