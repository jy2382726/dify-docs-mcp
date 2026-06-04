# Dify Workflow Skill

让 Claude Code 能根据自然语言生成可导入 Dify 的工作流 DSL YAML。

## 项目结构

- `.claude/skills/dify-workflow/` — 核心 Skill
  - `SKILL.md` — 主 Skill 文件（基于 yzmw123 适配）
  - `references/` — 参考文件（Schema、语法、校验、模板）
    - `node-schemas.md` — 26 个节点 Schema
    - `dsl-structure.md` — DSL 结构规范
    - `variable-syntax.md` — 变量引用语法 + rag 前缀
    - `node-output-fields.md` — 节点输出字段
    - `validation-rules.md` — 校验规则说明
    - `mcp-usage-guide.md` — Mintlify MCP 使用指引
    - `templates/` — 7 个工作流模板
  - `scripts/validate_dsl.py` — DSL 校验脚本
  - `examples/` — 集成测试示例（3 个工作流）
- `.mcp.json` — Mintlify MCP 配置（docs.dify.ai 文档查询）
- `specs/` — PRD + 调研文档（只读参考，不修改）
- `docs/` — 设计文档
- `.venv/` — Python 虚拟环境（Python 3.11，激活：`source .venv/bin/activate`）

## 技术栈

- Dify DSL v0.6.0（YAML 格式）
- Claude Code Skill（纯 Markdown 知识文件，无运行时能力）
- Mintlify MCP（文档语义搜索，端点在 .mcp.json）

## 工作流

### 生成工作流 DSL
1. 用户描述需求
2. 参考 `references/node-schemas.md` 选择节点
3. 参考 `references/variable-syntax.md` 写变量引用
4. 按 `references/templates/` 中的模板组织结构
5. 用 validate_dsl checklist 校验输出

### 修改 Skill 内容
- 节点 Schema 变更 → 编辑 `references/node-schemas.md`
- 新增模板 → 在 `references/templates/` 下新建文件
- 校验规则变更 → 编辑 `scripts/validate_dsl.py`

## 编码规范

- Skill 文件用中文编写
- 变量引用格式：`{{#node_id.field#}}`（节点输出）、`{{variable}}`（环境变量）
- DSL YAML 缩进为 2 空格
- 节点 ID 使用 snake_case

## 关键约束

- Phase 1 不做程序化校验（jsonschema），只用 LLM 自检 + checklist
- 不做 Dify API 集成（Phase 2 再做）
- 不做非 Claude Code 客户端支持
- Skill 内容锁定 Dify DSL v0.6.0，通过 Mintlify MCP 补充最新差异
- 文档查询依赖 Mintlify MCP（外部服务），Skill 中的 Schema/模板为降级方案

## Behavioral Guidelines (Karpathy-Inspired)

以下 4 条原则适用于全项目所有 task 实现期，目的是减少 AI 编码的常见失误。

### 1. Think Before Coding

**Don't assume. Don't hide confusion. Surface tradeoffs.**

Before implementing:
- State your assumptions explicitly. If uncertain, ask.
- If multiple interpretations exist, present them - don't pick silently.
- If a simpler approach exists, say so. Push back when warranted.
- If something is unclear, stop. Name what's confusing. Ask.

### 2. Simplicity First

**Minimum code that solves the problem. Nothing speculative.**

- No features beyond what was asked.
- No abstractions for single-use code.
- No "flexibility" or "configurability" that wasn't requested.
- No error handling for impossible scenarios.
- If you write 200 lines and it could be 50, rewrite it.

Ask yourself: "Would a senior engineer say this is overcomplicated?" If yes, simplify.

### 3. Surgical Changes

**Touch only what you must. Clean up only your own mess.**

When editing existing code:
- Don't "improve" adjacent code, comments, or formatting.
- Don't refactor things that aren't broken.
- Match existing style, even if you'd do it differently.
- If you notice unrelated dead code, mention it - don't delete it.

When your changes create orphans:
- Remove imports/variables/functions that YOUR changes made unused.
- Don't remove pre-existing dead code unless asked.

The test: Every changed line should trace directly to the user's request.

### 4. Goal-Driven Execution

**Define success criteria. Loop until verified.**

Transform tasks into verifiable goals:
- "Add validation" → "Write tests for invalid inputs, then make them pass"
- "Fix the bug" → "Write a test that reproduces it, then make it pass"
- "Refactor X" → "Ensure tests pass before and after"

For multi-step tasks, state a brief plan:
```
1. [Step] → verify: [check]
2. [Step] → verify: [check]
3. [Step] → verify: [check]
```

Strong success criteria let you loop independently. Weak criteria ("make it work") require constant clarification.
