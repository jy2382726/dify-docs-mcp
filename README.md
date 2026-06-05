# Dify Workflow Skill

让 Claude Code 根据自然语言生成可直接导入 Dify 的工作流 DSL YAML。

## 解决什么问题

团队使用 Dify 开发 AI 工作流时面临三个痛点：

1. **手动编写易错** — 变量引用格式 `{{#node_id.field#}}` 容易写错，节点配置字段遗漏常见
2. **查文档效率低** — Dify 文档分散在 20+ 页面，开发者频繁切换文档和编辑器
3. **无校验手段** — Dify 没有独立的 DSL 校验 API，错误只能在导入时发现

本项目通过 Claude Code Skill 为 Claude Code 注入 Dify 工作流领域知识，使其能直接生成、修改、校验 DSL YAML。

## 工作原理

```
用户描述需求 → Claude Code 调用 Skill → 查询 Dify 文档（Mintlify MCP）→ 生成 DSL YAML → 校验 → 可导入 Dify
```

核心组件：

- **Skill 知识文件**（`.claude/skills/dify-workflow/`）— 包含 DSL 结构、节点 Schema、变量语法、模板等
- **Mintlify MCP** — 连接 Dify 官方文档，提供实时文档查询能力
- **DSL 校验脚本** — 自动检查生成的 YAML 是否符合规范

## 项目结构

```
.
├── .claude/skills/dify-workflow/    # 核心 Skill
│   ├── SKILL.md                     # 主 Skill 定义
│   ├── scripts/
│   │   └── validate_dsl.py          # DSL 校验脚本
│   ├── references/                  # 参考知识库（3 层）
│   │   ├── dsl-structure.md         # DSL 整体结构
│   │   ├── node-schemas.md          # 节点类型 Schema
│   │   ├── variable-syntax.md       # 变量语法
│   │   ├── usecase-node-selection.md # 节点选型指南
│   │   ├── node-output-fields.md    # 节点输出字段
│   │   ├── database-tools.md        # 数据库工具
│   │   ├── plugin-marketplace-tools.md # 插件市场工具
│   │   ├── official-0.6-target.md   # 官方 DSL 0.6 规范
│   │   ├── real-world-yml-study.md  # 真实 YAML 案例
│   │   ├── validation-rules.md      # 校验规则
│   │   ├── mcp-usage-guide.md       # MCP 使用指南
│   │   └── templates/               # 工作流模板
│   │       ├── simple-llm.yaml
│   │       ├── rag-retrieval.yaml
│   │       ├── chatflow-multi-turn.yaml
│   │       ├── if-else-branch.yaml
│   │       ├── iteration.yaml
│   │       ├── http-code.yaml
│   │       └── error-handling.yaml
│   └── examples/                    # 集成测试示例
├── .mcp.json                        # MCP 服务器配置
├── specs/                           # PRD + 调研文档
├── docs/                            # 设计文档
├── examples/                        # Dify 工作流示例
└── CLAUDE.md                        # 项目指令
```

### 参考知识库的 3 层组织

| 层级 | 加载时机 | 文件 |
|------|----------|------|
| 第 1 层 | 始终加载 | `dsl-structure.md`、`node-schemas.md` |
| 第 2 层 | 按需加载 | `usecase-node-selection.md`、`variable-syntax.md`、`node-output-fields.md` |
| 第 3 层 | 特定场景 | `database-tools.md`、`plugin-marketplace-tools.md`、`official-0.6-target.md` 等 |

## 快速开始

### 前置条件

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) 已安装并可用
- 项目根目录的 `.mcp.json` 已配置 Mintlify MCP
```json
{
  "mcpServers": {
    "dify-docs": {
      "type": "streamable-http",
      "url": "https://dify-6c0370d8.main-kill-isr.mintlify.me/mcp"
    }
  }
}
```

### 使用方式

1. 在 Claude Code 中打开本项目
2. 描述你需要的 Dify 工作流，例如：

```
帮我创建一个 RAG 工作流：用户输入问题，从知识库检索相关文档，调用 LLM 生成回答
```

3. Claude Code 会自动调用 dify-workflow Skill，生成 DSL YAML
4. 将生成的 YAML 导入 Dify 即可使用

### 校验 DSL

```bash
python3 .claude/skills/dify-workflow/scripts/validate_dsl.py <your-workflow.yaml>
```

## 支持的节点类型

| 节点 | 说明 |
|------|------|
| Start | 工作流入口，定义输入变量 |
| End | 工作流出口，定义输出 |
| LLM | 大语言模型调用 |
| Knowledge Retrieval | 知识库检索（RAG） |
| Code | 自定义 Python/JS 代码 |
| HTTP Request | HTTP API 调用 |
| If-Else | 条件分支 |
| Iteration | 循环迭代 |
| Variable Aggregator | 变量聚合 |
| Template Transform | Jinja2 模板转换 |
| Question Classifier | 问题分类 |
| Tool / Plugin | 外部工具调用 |
| Answer | Chatflow 流式回答 |

## 模板一览

| 模板 | 场景 |
|------|------|
| `simple-llm.yaml` | 最简 LLM 调用 |
| `rag-retrieval.yaml` | 知识库检索 + LLM |
| `chatflow-multi-turn.yaml` | 多轮对话 Chatflow |
| `if-else-branch.yaml` | 条件分支 |
| `iteration.yaml` | 循环迭代处理 |
| `http-code.yaml` | HTTP 调用 + Code 节点 |
| `error-handling.yaml` | 异常处理 |

## 技术栈

- Dify DSL v0.6.0（YAML 格式）
- Claude Code Skill（纯 Markdown 知识文件）
- Mintlify MCP（Dify 官方文档查询）
- Python 3.11（校验脚本）

## 开发

### 校验规则变更

编辑 `scripts/validate_dsl.py`。

### 新增模板

在 `.claude/skills/dify-workflow/references/templates/` 下新建 YAML 文件。

### 节点 Schema 变更

编辑 `.claude/skills/dify-workflow/references/node-schemas.md`。

### 查询 Dify 最新文档

项目配置了 Mintlify MCP，Claude Code 可直接查询 `docs.dify.ai` 的最新文档。

## 维护指南（Dify 升级后）

本项目依赖 Dify DSL 规范，Dify 平台升级可能带来节点类型、Schema、版本号的变化。
以下是保持 Skill 与 Dify 同步的维护策略。

### 信息源分层

| 信息源 | 用途 | 维护方式 |
|--------|------|----------|
| Mintlify MCP（实时） | 查询最新节点文档、API 变更 | 自动，无需手动更新 |
| `official-0.6-target.md`（本地） | 锁定当前 DSL 版本的权威规范 | Dify 升级时手动更新 |
| `real-world-yml-study.md`（本地） | 兼容性参考、图结构、工具节点模式 | 定期采样公共 DSL 补充 |

**原则**：Mintlify MCP 是第一信息源，本地文件是降级方案。MCP 可用时优先查文档，MCP 不可用时依赖本地文件。

### 维护步骤

**1. 检查 DSL 版本变更**

Dify 没有官方页面专门发布 DSL 版本号，需要通过以下方式获取：

| 方式 | 地址 | 可靠度 | 操作 |
|------|------|--------|------|
| 导出工作流 | 从自己的 Dify 实例导出任意工作流，查看 YAML 中的 `version` 字段 | 最高 | 首选 |
| Dify 源码 | https://github.com/langgenius/dify — 搜索 DSL 版本常量定义 | 最高 | 源码级确认 |
| GitHub Releases | https://github.com/langgenius/dify/releases — changelog 中可能提及 DSL 变更 | 中 | 关注更新日志 |
| Mintlify MCP | 查询 `docs.dify.ai` 的 DSL 相关页面 | 低 | 文档通常滞后于源码 |

**推荐流程**：Dify 升级后，从实例导出一个工作流检查 `version` 字段，与 `official-0.6-target.md` 对比。
版本号变化时，更新 `official-0.6-target.md` 和 `SKILL.md` 中的 `version` 字段。

**2. 导出新节点类型的最小 DSL**

Dify 新增节点类型时：

1. 在 Dify 中创建包含该节点的最小工作流
2. 导出 DSL YAML
3. 将导出的 YAML 作为 Schema 来源，补充到 `node-schemas.md`
4. 在 `templates/` 下新建对应的最小模板

**3. 采样公共 DSL（季度）**

定期从活跃的公共 Dify DSL 仓库采样，将重复出现的模式折叠到 `references/`：
- `Awesome-Dify-Workflow`（https://github.com/svcvit/Awesome-Dify-Workflow）
- `DifyAIA`（https://github.com/BannyLon/DifyAIA）
- `dify-for-dsl`（https://github.com/wwwzhouhui/dify-for-dsl）

采样重点：新节点组合模式、工具节点配置变化、变量引用新语法。

**4. 更新校验脚本**

当发现重复的导入错误模式时，将确定性检查添加到 `scripts/validate_dsl.py`。

**5. 编辑后验证**

```bash
# 校验 Skill 元数据
python3 .claude/skills/dify-workflow/scripts/validate_dsl.py examples/*.yaml
```

### 维护红线

- 稳定模式加到 `references/`，**不加到 `SKILL.md`** — SKILL.md 保持精简
- `official-0.6-target.md` 与 `real-world-yml-study.md` **始终分开** — 官方规范不被公共样本污染
- 新增工具节点时，**优先使用导出的 DSL 作为 Schema 来源**，不凭工具名猜测参数

### MCP 不可用时的降级

如果 Mintlify MCP 不可用（网络问题、服务下线）：

1. 依赖 `official-0.6-target.md` 中的本地规范
2. 依赖 `node-schemas.md` 中的节点 Schema
3. 新节点类型可能无法生成，需等 MCP 恢复或手动补充 Schema

## 设计文档

- PRD: `specs/prd.md`
- 调研文档: `specs/research/`（共 7 篇）
- 收敛设计: `docs/superpowers/specs/`

## 致谢

本项目基于以下开源项目和资源：

### 主要基座

- **[yzmw123/dify-workflow-dsl-skill](https://github.com/yzmw123/dify-workflow-dsl-skill)** — 本项目的 Skill 架构、节点 Schema、校验脚本、模板均基于此项目适配而来。覆盖 25+ 节点类型、变量引用语法、172+ 真实 DSL 分析。

### 参考项目

- **[r-hashi01/skills-to-dify-workflow](https://github.com/r-hashi01/skills-to-dify-workflow)** — 节点输出字段参考（从 Dify 源码 NodeRunResult 验证）
- **[R3flector/skills](https://github.com/R3flector/skills)** — dify-workflow-builder，8 种工作流模式参考
- **[svcvit/Awesome-Dify-Workflow](https://github.com/svcvit/Awesome-Dify-Workflow)** — 社区 Dify 工作流 DSL 模板库（10k+ stars），采样公共 DSL 的数据来源

### 基础设施

- **[Dify](https://github.com/langgenius/dify)** — 开源 LLM 应用开发平台
- **[Mintlify](https://mintlify.com/)** — Dify 官方文档的托管平台，本项目通过 Mintlify MCP 实时查询文档

## License

内部项目，暂未公开。
