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

## 降级触发判断

在决定是否使用 MCP 前，先判断可用性：

1. **检查 MCP 工具是否存在**：如果 `search_dify_docs` 和 `query_docs_filesystem_dify_docs` 不在可用工具列表中，直接进入降级模式，不要尝试调用
2. **首次调用失败时降级**：如果工具存在但调用返回错误（超时、连接拒绝、认证失败等），本次任务不再重试，立即降级
3. **不需要时不调用**：节点在 `node-schemas.md` 的 26 个 Schema 中且用户未指定特殊版本，直接使用本地 Schema，不调用 MCP

判断流程：

```
需要查文档？
├─ 节点在本地 26 个 Schema 中 → 用本地 Schema，不调用 MCP
├─ 节点不在 Schema 中 → 尝试调用 MCP
│   ├─ MCP 工具不可用 → 进入降级模式
│   ├─ 调用成功 → 使用 MCP 结果
│   └─ 调用失败 → 进入降级模式，本次不再重试
└─ 用户指定非 v0.6.0 版本 → 尝试调用 MCP
    ├─ 调用成功 → 使用 MCP 结果
    └─ 调用失败 → 进入降级模式
```

## 降级策略

当 Mintlify MCP 不可用时：
1. 使用 references/node-schemas.md 中的 Schema
2. 使用 references/templates/ 中的模板
3. 对于不确定的配置，返回 warning 提示用户手动确认
4. 告知用户："MCP 不可用，使用本地 reference，生成的 DSL 需要 Dify 导入测试"
5. 不报错，确保 Skill 功能 100% 可用
