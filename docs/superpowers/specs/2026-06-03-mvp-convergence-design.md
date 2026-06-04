# MVP 收敛设计：Dify 工作流 MCP/Skill 服务

> 日期：2026-06-03
> 基于：specs/research/ 01-07 全部调研 + 5 角色对抗评估 + 三次验证
> 方案：Phase 1 Skill + 量化验证关卡 + MCP Server 骨架（方案 C）

---

## 1. 业务目标 + 产品目标 + Non-Goals

### 业务目标

让团队用 AI 编程智能体（Claude Code）高效开发 Dify 工作流，降低 DSL 编写门槛和错误率。

### 产品目标

交付一个 Claude Code Skill + Mintlify MCP 配置，使 Claude Code 能根据自然语言描述生成可直接导入 Dify 的 DSL YAML 文件。

### Non-Goals（明确不做什么）

- 不做可视化工作流编辑器或 Web UI
- 不做工作流运行时管理（部署/监控/回滚）
- 不做 Dify 平台的替代品
- 不做非 Claude Code 客户端的支持（Phase 2 再考虑）
- 不做本地离线文档搜索（依赖 Mintlify MCP）
- 不做用户认证/权限系统

---

## 2. 主画像 + 反画像

### 主画像：Dify 工作流开发者（使用 Claude Code）

- 技术背景：熟悉 Dify 平台，了解工作流节点和 DSL 结构
- 痛点：手动写 DSL YAML 容易出错（变量引用格式、节点配置遗漏），查文档效率低
- 目标：用自然语言描述需求，让 Claude Code 生成可导入的工作流

### 反画像（明确不服务谁）

- 非技术用户（需要可视化拖拽编辑器）
- 不用 Claude Code 的团队（Cursor/Windsurf 用户，Phase 2 再覆盖）
- 需要管理工作流生命周期的运维角色

---

## 3. 核心用户故事（INVEST 格式）

### US-1：自然语言生成工作流

> 作为 Dify 工作流开发者，我想用自然语言描述一个工作流需求，让 Claude Code 生成可导入的 DSL YAML，这样我就不需要手写复杂的节点配置和变量引用。

### US-2：DSL 校验与修复

> 作为 Dify 工作流开发者，我想让 Claude Code 在生成 DSL 后自动校验并修复常见错误，这样我导入 Dify 时不会遇到格式问题。

### US-3：查询节点文档和变量语法

> 作为 Dify 工作流开发者，我想让 Claude Code 查询特定节点的配置方式和变量引用语法，这样我能正确配置复杂节点（如 LLM 的 Context RAG、Jinja2 模板）。

---

## 4. MVP 功能列表 + Out-of-Scope

### Must-have

| 功能 | 来源 | 说明 |
|------|------|------|
| 文档语义搜索 | Mintlify MCP（Layer 1） | 已配置，免费，21 节点页面 |
| 节点 Schema 参考（25+ 类型） | yzmw123 Skill 适配（Layer 2） | 直接复用 yzmw123 的 node-schemas.md |
| 变量引用语法规则 | yzmw123 + 源码补充 rag 前缀 | 直接复用 + 0.5h 补充 |
| DSL 校验 checklist | yzmw123 validate_dsl.py | 直接复用 15+ 规则 |
| 工作流模板（4+3 个） | yzmw123（4）+ R3flector（3） | 直接复用 + 移植补充 |
| 节点输出字段参考 | r-hashi01 移植 | 1h 移植工作 |
| Mintlify MCP 使用指引 | 新增 Skill 段落 | 0.5h |
| `.mcp.json` 配置 | 已完成 | Mintlify MCP 端点已配置 |

### Should-have

- MCP Server 项目骨架（`pyproject.toml` + 空 `server.py` + 目录结构）
- Skill 使用文档（README）

### Could-have

- 补充更多工作流模板（从 Awesome-Dify-Workflow）
- 节点选择指南（按场景推荐节点）

### Out-of-Scope

| 不做的能力 | 理由（来自 06 架构基线） | 后续迭代备注 |
|-----------|----------------------|------------|
| 程序化 DSL 校验（jsonschema） | Phase 2 能力，需从 graphon 逆向构建 JSON Schema（4-5 天） | — |
| Dify API 集成（run_workflow） | Phase 2 能力，需 httpx 封装 + API Key 管理 | **后续迭代优先级高**：用于导入失败后的自动调试闭环 |
| 本地离线文档搜索（FTS5） | Phase 2 能力，Mintlify MCP 已覆盖 | — |
| 结构化 JSON Schema 查询 | Phase 2 能力，Skill 用 Markdown 够用 | — |
| 多客户端支持（Cursor/Windsurf） | Phase 2 能力，需 MCP Server | — |
| 向量语义检索（FAISS） | Phase 3 能力，200 页文档不需要 | — |

---

## 5. 每个核心功能的关键边界条件

| 功能 | 边界条件 | 处理方式（06 架构能支撑） |
|------|---------|----------------------|
| 文档搜索 | Mintlify 不可用/超时 | Skill 中的 Schema 和模板仍可用；降级为纯 Skill 模式 |
| 文档搜索 | 查询结果不精准 | Claude Code 可用 `query_docs_filesystem` 的 `cat` 命令直接读取特定页面 |
| 节点 Schema | yzmw123 的 Schema 与用户 Dify 版本不一致 | Skill 中标注 DSL 版本（v0.6.0）；差异通过 Mintlify MCP 查最新文档补充 |
| 节点 Schema | 某个节点类型不在 25+ 列表中 | Claude Code 通过 Mintlify MCP 搜索该节点文档，自行推导配置 |
| 变量引用 | 用户不清楚 `{{#...#}}` 和 `{{...}}` 的区别 | Skill 中明确区分两种格式及用途 |
| DSL 校验 | checklist 无法覆盖的语义错误 | 返回 warning 而非 error，提示用户手动检查 |
| 模板 | 没有匹配用户场景的模板 | Claude Code 基于 Schema 和文档从零生成 |
| 并发 | 多人同时使用 | 每人独立 Claude Code 实例 + 独立 Skill 文件，无共享状态 |
| 失败 | 生成的 DSL 导入 Dify 失败 | 用户反馈错误 → Claude Code 通过 Mintlify MCP 查文档 → 修复 → 重新生成 |

---

## 6. 非功能需求（用数字）

| 维度 | 指标 | 目标值 | 依据 |
|------|------|--------|------|
| 性能 | Mintlify MCP 文档搜索响应时间 | < 3 秒（P95） | MCP 端点实测，5000 req/h 限额 |
| 性能 | Skill 文件加载时间 | < 1 秒 | 本地 Markdown 文件，无网络依赖 |
| 性能 | 端到端工作流生成时间 | < 30 秒（从需求描述到 DSL 输出） | 包含 1-2 次 MCP 调用 + Claude Code 生成 |
| 可用性 | Mintlify MCP 可用性 | 99.5% | 外部依赖，不可控但有降级方案 |
| 可用性 | Skill 功能可用性 | 100% | 本地文件，无外部依赖 |
| 合规 | 数据隐私 | 不存储用户数据 | Skill 本地运行，Mintlify 查询公开文档 |
| 容量 | Phase 1 并发用户 | 无上限 | 每人独立实例，无共享资源 |
| 容量 | Phase 2 MCP Server 并发（参考值） | 50 用户 | stdio 模式，每人一个进程 |
| 容量 | Phase 2 Schema 查询响应（参考值） | < 200ms | SQLite 本地查询 |
| 容量 | Phase 2 文档搜索响应（参考值） | < 500ms | FTS5 全文搜索 |

---

## 7. 每个用户故事的验收标准

### US-1：自然语言生成工作流

```
Given: 用户已配置 Mintlify MCP 和 dify-workflow Skill
When: 用户描述 "创建一个工作流：接收用户问题 → 搜索知识库 → 用 LLM 回答"
Then: Claude Code 生成完整的 DSL YAML，包含 start → knowledge-retrieval → llm → end 节点，
      所有节点 data 字段完整，edges 正确连接，变量引用格式为 {{#node_id.field#}}
```

### US-2：DSL 校验与修复

```
Given: Claude Code 已生成一个 DSL YAML
When: Skill 中的校验 checklist 被触发
Then: 检查项包括：必填字段完整性、节点 ID 唯一性、边引用的节点存在、变量引用格式正确；
      发现错误时自动修复并重新输出；
      无法自动修复的问题返回 warning 级提示
```

### US-3：查询节点文档和变量语法

```
Given: 用户询问 "LLM 节点怎么配置 Context RAG？"
When: Claude Code 调用 Mintlify MCP 的 search_dify_docs 或 query_docs_filesystem
Then: 返回 LLM 节点文档中关于 Context RAG 的完整配置说明，
      包括 dataset 变量引用格式和配置步骤
```

---

## 8. 优先级（MoSCoW）

| 优先级 | 功能 | 06 改造成本 |
|--------|------|-----------|
| **Must** | Mintlify MCP 配置 | 0（已完成） |
| **Must** | yzmw123 Skill 适配（SKILL.md + references/） | 低（直接适配） |
| **Must** | 补充 rag 变量前缀文档 | 低（0.5h） |
| **Must** | 节点输出字段移植（r-hashi01） | 低（1h） |
| **Must** | 补充 3 个工作流模板（R3flector） | 低（1-2h） |
| **Must** | Mintlify MCP 使用指引 | 低（0.5h） |
| **Must** | 集成测试（生成 3-5 个工作流） | 低（1-2h） |
| **Should** | MCP Server 项目骨架 | 低（0.5h） |
| **Should** | Skill 使用文档（README） | 低（0.5h） |
| **Could** | 补充更多模板（Awesome-Dify-Workflow） | 中 |
| **Could** | 节点选择指南（按场景推荐） | 中 |
| **Won't** | 程序化 DSL 校验（jsonschema） | 高（4-5 天） |
| **Won't** | Dify API 集成（run_workflow）— 后续迭代优先级高 | 高（2-3 天） |
| **Won't** | 多客户端支持 | 高（架构变更） |
| **Won't** | 本地离线文档搜索 | 高（3-5 天） |
| **Won't** | 向量语义检索 | 高（3-5 天） |

---

## 9. 北极星指标 + 关停线

### 北极星指标

**生成的 Dify 工作流 DSL 首次导入成功率**

- 定义：Claude Code 生成的 DSL YAML 文件在 Dify 平台上首次导入即成功的比例
- 目标：>= 80%（Phase 1）
- 测量方式：团队试用期间记录每次导入结果

### 关停线

- **Phase 1 关停**：如果试用 2 周后首次导入成功率 < 50%，说明 Skill 方案覆盖不足，直接启动 Phase 2
- **Phase 2 扩展上限**：如果团队超过 15 人且 Mintlify MCP 频繁触发速率限制（1000 req/h/站点），需要本地 FTS5 替代

---

## 10. Open Questions

### 从 06 遗留的 Open Questions

| # | 问题 | 来源 | 影响 |
|---|------|------|------|
| OQ-1 | Skill 的实际覆盖率：团队试用 2-4 周后，Skill 能覆盖多少实际场景？ | 06 S6.2 | 决定是否启动 Phase 2 |
| OQ-2 | graphon 0.4.0 -> 0.5.0 的演进方向和 breaking change 风险 | 06 S6.3 | 影响 Phase 2 的 Schema 策略 |
| OQ-3 | 团队是否真的需要程序化 DSL 校验，还是 LLM 自检就够了？ | 06 S6.4 | 影响 Phase 2 的 validate_dsl 优先级 |
| OQ-4 | 团队中是否有人用 Cursor/Windsurf？MCP 协议支持是否刚需？ | 06 S6.5 | 影响 Phase 2 启动时机 |
| OQ-5 | Mintlify MCP 能否覆盖 DSL 层变量引用（`{{#...#}}` 格式）的查询？ | 06 S6.6 | 影响 Skill 中变量文档的补充范围 |

### 新增的产品级 Open Questions

| # | 问题 | 影响 |
|---|------|------|
| OQ-6 | Skill 内容需要覆盖多少个工作流模板才算"够用"？4 个（yzmw123）+ 3 个（R3flector）= 7 个是否足够？ | 影响 Could-have 模板补充范围 |
| OQ-7 | 当用户的 Dify 版本与 Skill 中的 Schema 版本（v0.6.0）不一致时，如何处理？需要版本适配策略吗？ | 影响 Skill 的维护策略 |
| OQ-8 | Skill 是否应该包含"如何部署生成的工作流到 Dify"的指引，还是只管生成不管部署？ | 影响 Skill 内容边界 |
| OQ-9 | `run_workflow` 在后续迭代中的优先级——是否应该作为 Phase 2 的第一个工具？（用户反馈：导入失败自动调试闭环价值高） | 影响 Phase 2 工具排序 |

---

## 11. 依赖与约束

### 外部依赖

| 依赖 | 风险 | 缓解措施 |
|------|------|---------|
| Mintlify MCP 端点稳定性 | 端点变更或服务中断 | `.mcp.json` 一处修改即可恢复；Skill 中的 Schema/模板不受影响 |
| yzmw123 项目持续维护 | 停止维护 | 已计划适配而非直接引用，DSL v0.6.0 锁定 |
| Claude Code Skill 格式兼容性 | Skill 格式变更 | 跟踪 Claude Code 更新日志 |
| Dify 官方文档内容准确性 | 文档与源码不一致 | Skill 中标注"以源码为准"，关键信息从源码验证 |

### 约束

| 约束 | 说明 |
|------|------|
| Skill 不能执行代码或调用 API | Claude Code Skill 是纯知识文件，无运行时能力 |
| Skill 内容受 context window 限制 | yzmw123 的 Skill 约 8-15K token，Claude Code 200K context 可容纳 |
| Mintlify MCP 速率限制 5000 req/h/user | 团队 5-15 人远低于限制 |
| Phase 1 不支持非 Claude Code 客户端 | Cursor/Windsurf 用户需等 Phase 2 |
| Dify DSL 无官方 JSON Schema | 校验依赖逆向工程 + LLM 自检 |

---

## 附录：07 项目架构审核意见

### 架构合理性评估

**通过项：**
- 三层渐进式架构设计合理（Mintlify MCP → Skill → MCP Server）
- stdio 协议选择正确（Claude Code 原生支持，零配置）
- 无需前端——纯开发者工具，通过 Claude Code 终端交互
- SQLite + FTS5 作为 Phase 2 数据层是经过验证的模式（n8n-mcp 验证）
- 部署模型简单：Phase 1 = git clone + 文件复制，Phase 2 = pip install

**需改进项：**
- 缺少测试策略（Skill 内容准确性、MCP 工具正确性、Dify 更新回归）
- Skill 内容更新机制未定义（yzmw123 锁定 v0.6.0，Dify 已 v1.10.x）
- Phase 2 缺少打包计划（pyproject.toml、Dockerfile、版本管理）
- Phase 1→2 触发条件需要量化指标（本设计已补充：导入成功率 < 50%）

### 部署考量

- **Phase 1 部署**：git clone → 复制 `dify-workflow/` 到 `.claude/skills/` → 配置 `.mcp.json`。无服务器进程，无容器。
- **Phase 2 部署**：`pip install` 或 `uv run` → 配置 `.mcp.json` 的 `command` + `env`。stdio 模式，每人本地运行。
- **不需要前端**：所有交互通过 Claude Code 终端完成，Skill 文件是纯 Markdown，MCP 工具返回结构化 JSON。
