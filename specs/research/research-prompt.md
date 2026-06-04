# 调研任务：Dify 工作流 MCP 服务 · 产品立项调研

## 项目定位

「Dify 工作流 MCP 服务」是给 **团队内 Claude Code 编程智能体用户** 用的
**文档查询 + 节点 Schema 查询 + 变量引用查询 + DSL 校验 + Dify API 集成**
工具，解决 **智能体开发 Dify 工作流时缺乏官方文档参考、节点配置规范和
语法校验** 的问题。

目标市场：中国大陆为主，团队内部使用。

## 📁 输出路径

所有产出写入 `/root/projects/dify-docs-mcp/specs/research/`：

| 文件 | 对应主题 |
|------|---------|
| `01-产品形态.md` | 主题 1 |
| `02-关键资源.md` | 主题 2 |
| `03-开源项目.md` | 主题 3 |
| `04-实现方案.md` | 主题 4 |
| `05-决策汇总.md` | 主 Claude 串行汇总 |

---

## 调研主题 1：产品形态

围绕「Dify 工作流 MCP 服务」，需要回答这些问题。每个都要有可视证据
（截图描述 / 链接 / 真实产品名）。

1. **同类产品盘点**：参考 n8n 官方 MCP server，在 MCP 服务生态中至少找
   10 个与「AI 工作流开发辅助」或「文档查询类 MCP」相关的同类产品/服务。
   重点关注：
   - n8n MCP server 的完整功能列表和工具定义
   - 其他 AI 工作流平台（Make/Zapier/FastGPT/Coze/Dify）
   是否提供了类似的 MCP 服务
   - 通用文档查询类 MCP server（如 Context7 等）的产品形态
   - Dify 相关的第三方 MCP 服务（如有）
2. **信息架构**：n8n MCP server 暴露了哪些 tools？每个 tool 的输入输出
   是什么？工具分组逻辑是什么？
3. **交互模式**：Claude Code 调用 MCP tool 的典型工作流是什么？
   智能体在什么时机调用文档查询、什么时机调用校验？
4. **AI 化案例**：有没有 AI 辅助开发工作流的端到端案例？
   （从自然语言描述需求 → AI 生成工作流 DSL → 校验 → 部署）
5. **推送/触达形态**：MCP 服务的返回结果格式最佳实践是什么？
   结构化 JSON？Markdown？纯文本？错误信息怎么设计对 AI 最友好？

产出：`01-产品形态.md` —— 同类产品全景图 + n8n MCP 功能对标 + MCP tool 设计参考

---

## 调研主题 2：关键资源

围绕「Dify 工作流 MCP 服务」，核心依赖的 4 类资源怎么获取？

### 2A. Dify 官方文档

1. Dify 官方文档的结构和覆盖范围（工作流相关文档有哪些章节？）
2. 文档获取方式：官方 Git 仓库？在线抓取？API？
3. 文档更新频率和版本管理策略
4. 工作流 DSL 语法规范在文档中的位置和完整度
5. 文档中是否有节点类型的完整参数定义？

### 2B. Dify 源码

1. Dify GitHub 仓库结构，工作流相关代码在哪个目录？
2. 节点类型的定义文件在哪里？（Python class？JSON Schema？）
3. DSL 文件的解析和校验逻辑在源码中的位置
4. 变量引用（`{{#node_id.variable#}}`）的解析逻辑在哪里？
5. 能否从源码自动提取节点 schema？难度如何？

### 2C. Dify API

1. Dify 是否提供公开 API 来获取节点类型列表？
2. 是否有 API 可以导入/导出工作流 DSL？
3. 是否有 API 可以校验 DSL 合法性？
4. 是否有 API 可以在线运行/测试工作流？
5. API 的认证方式、限频、版本管理

### 2D. 人工规范文档（初期可能没有）

1. 团队是否已有内部的 Dify 工作流开发规范？
2. 是否有沉淀的最佳实践、反模式清单？
3. 如果没有，调研阶段建议如何补充？

产出：`02-关键资源.md` —— 4 类资源获取方案 + 可行性评估 + 推荐路径

---

## 调研主题 3：开源项目

围绕「Dify 工作流 MCP 服务」，GitHub / Gitee 上做同类事情的开源项目
至少找 10 个。

关键词参考：
- Dify MCP server / Dify workflow MCP / Dify DSL
- workflow MCP server / n8n MCP / AI workflow tool
- Dify 工作流 / Dify DSL 校验 / Dify 文档查询
- MCP document server / MCP schema validation

1. **可参考项目盘点**：至少 10 个
2. **每个项目的事实**：
   - Star / 最近 commit / License
   - 用什么技术栈（TypeScript/Python/Go？）
   - 做了什么（核心能力）
   - 没做什么（短板）
   - 我能复用什么模块
   - 集成难度（直接 fork / 拆模块 / 仅参考）
3. **生态拼图**：哪个项目擅长文档查询？哪个擅长 schema 校验？
   哪个擅长 MCP server 框架？
4. **推荐组合**：基于「优先 fork 改 > 拆模块用 > 从 0 写」的意图，
   应该 fork / 借鉴哪几个？

产出：`03-开源项目.md` —— 项目清单 + 复用矩阵 + 初版代码源建议

---

## 调研主题 4：实现方案

把系统拆成以下 8 个模块，每个模块给业内主流方案 + 推荐：

| 模块 | 调研维度 |
|------|---------|
| 文档抓取/索引模块 | 抓取 Dify 官方文档 → 建立可检索知识库的主流方案（向量检索？全文检索？本地文件？）|
| DSL Schema 模块 | 从 Dify 源码或文档提取节点类型和参数定义的方式（静态分析？API 获取？手动维护？）|
| 文档查询 MCP Tool | 暴露给 Claude Code 的文档查询接口设计（语义搜索？关键词？返回格式？）|
| 节点/Schema 查询 MCP Tool | 节点类型和参数 schema 查询接口设计（按名称查？按场景推荐？）|
| 变量引用查询 MCP Tool | 变量语法和作用域查询接口设计（`{{#xxx#}}` 语法规则、跨节点引用）|
| DSL 校验 MCP Tool | 工作流 DSL 文件校验方案（JSON Schema 校验？自定义规则引擎？语义校验？）|
| Dify API 集成模块 | 对接 Dify 平台 API 的方案（REST 封装、认证管理、错误处理）|
| 配置/部署模块 | MCP server 的启动配置方式（stdio？SSE？配置文件格式？）|

每个模块产出：
- 业内主流 2-3 个方案 + 各自优劣
- 推荐用什么 + 理由（要写"为什么不用 X"）
- 大致工作量预估

产出：`04-实现方案.md` —— 方案对比 + 推荐技术栈 + 架构图（Mermaid）

---

## 最终决策汇总：`05-决策汇总.md`

1. **产品形态决策**：MCP server 的工具列表 + 每个工具的输入输出设计
2. **关键资源决策**：文档获取方式 + 源码提取方案 + API 集成范围
3. **开源复用决策**：fork / 借鉴哪几个项目
4. **技术栈决策**：8 模块各自选什么方案
5. **架构总图**：一张端到端架构图（Mermaid）
6. **成本估算**：开发工作量 + 运行成本
7. **风险与对策**：最该警惕的坑

---

## 🔑 产出质量要求

1. **每份文档 2000-5000 字**，不要水字数也不要太简略
2. **每个结论必须有来源**，不能凭印象写
3. **推荐方案必须写理由**，包括"为什么不用 X"
4. **不确定的标 [待验证]**，不要编造
5. **文档结构统一**：一级标题 = 章节名，二级标题 = 子问题编号
6. **用 Write 工具写入指定路径**，不要输出到终端

---

## 🔀 MCP 四路调研规范

每个 sub-agent 必须合理使用以下 4 套 MCP 工具，按场景选择最优工具：

| 路径 | 工具 | 最佳场景 |
|------|------|---------|
| **路 A** | `web-search-prime` (`web_search_prime`) | 中文搜索、国内产品、技术博客、知乎/CSDN |
| **路 B** | `web-reader` (`webReader`) | 抓取指定 URL 的完整页面内容 |
| **路 C** | `muyu-search` (`web_search` / `web_fetch` / `web_map`) | 深度搜索（含规划）、网站结构扫描 |
| **路 D** | `zread` (`get_repo_structure` / `read_file` / `search_doc`) | GitHub 仓库结构、源码阅读、文档搜索 |

**工具选择策略**：
- 搜中文信息 / 找国内产品 → 路 A `web-search-prime`
- 读特定网页内容 → 路 B `web-reader`
- 需要深度规划的复杂搜索 → 路 C `muyu-search`（需先走 planning 流程）
- 读 GitHub 源码 / 仓库结构 → 路 D `zread`

**muyu-search 调用约束**：它有 planning 硬门禁，必须先走完
`plan_intent → plan_complexity → plan_sub_query → plan_search_term →
plan_tool_mapping → plan_execution` 才能调 `web_search`，否则会被拒绝。

**事实标注**：每条事实/数据后标来源：
`[web-search-prime]` / `[web-reader]` / `[muyu]` / `[zread]`

同一事实点多个路径都查到一致 → 标 ✅ 双路验证。
只有单路找到 → 标 ⚠️ 单源。
冲突时**不要替我裁决**，原样标出 + 列双方来源。

---

## 并行执行说明

用 Claude Code 原生 **Agent 工具**，在**同一条消息中发起 4 个 Agent 调用**：

| Agent | 任务 | 产出文件 |
|-------|------|---------|
| Agent 1 | 调研主题 1：产品形态 | `01-产品形态.md` |
| Agent 2 | 调研主题 2：关键资源 | `02-关键资源.md` |
| Agent 3 | 调研主题 3：开源项目 | `03-开源项目.md` |
| Agent 4 | 调研主题 4：实现方案 | `04-实现方案.md` |

4 个 Agent 全部返回后，主 Claude 串行写 `05-决策汇总.md`。

每个 Agent 的提示词必须包含（四要素）：
1. **Focused scope**：只做分配的这一个主题，明确边界
2. **Self-contained context**：项目定位、目标市场、对标竞品等所有必要背景
3. **Constraints**：必须用 MCP 四路工具调研 + 来源标注 + 用 Write 写入指定路径
4. **Expected output**：文件路径 + 文档结构 + 质量要求

---

## 下一步（按需走 3-4 段链路，不要跳步）

调研完成（5 份 md 落地）后，按以下顺序推进：

### ⓪ 如果有多个候选技术方案 → 先做对抗调研（强烈建议）

如果 03-开源项目.md 推荐了 >= 2 个候选 fork 项目，或你在 MCP 框架 /
技术栈之间纠结，**先**用 `adversarial-architecture-selection` Skill
做对抗调研：

> 我有多个候选技术方案：[列出候选]
> 请用 adversarial-architecture-selection Skill 做架构基线决策。

产出：`specs/research/06-架构基线决策.md` + `05-决策汇总.md (v2)`
然后进入 ①。

如果只有 0-1 个候选 → 跳过 ⓪，直接进 ①。

### ① 用 brainstorming Skill 收敛 MVP 方向

> 请仔细阅读 specs/research/ 下的所有调研文件，然后用 Superpowers
> brainstorming Skill 帮我收敛 MVP 方向。必须帮我确定以下 11 个问题
> （这是写 PRD 的输入）：
>
> 1. 业务目标 + 产品目标 + Non-Goals
> 2. 1-2 个主画像 + 反画像
> 3. 3-5 条核心用户故事（INVEST 格式）
> 4. MVP 功能列表 + Out-of-Scope 清单
> 5. 每个核心功能的关键边界条件
> 6. 非功能需求（必须用数字）
> 7. 每个用户故事的验收标准（Given/When/Then）
> 8. 优先级（MoSCoW）
> 9. 北极星指标 + 关停线
> 10. 至少 3 个 Open Questions
> 11. 关键依赖与约束
>
> ⚠️ 技术选型相关讨论留给后续 spec-kit /plan 环节。

### ② 用 prd-writer Skill 写 PRD

> 基于 brainstorming 的收敛结果和 specs/research/ 下的调研文件，
> 用 prd-writer Skill 帮我写 PRD（产出到 specs/prd.md）。

### ③ 用 GitHub Spec-Kit 把 PRD 转成可执行 spec

对 PRD 每个 Must-have 功能各跑一次：

> 实现 specs/prd.md 中的 <功能名>，跑全套 spec-kit 流程：
> /speckit.specify → /speckit.plan → /speckit.tasks → /speckit.implement

⚠️ 按 Must-have 拆条跑——禁止把整个 PRD 喂给 /speckit.specify。
