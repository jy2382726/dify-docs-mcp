# 官方 DSL 0.6.0 规范

需要确认 Dify 官方 DSL 规范时加载此文件。包含从 Dify 源码提取的权威信息。

## 适用场景

- 需要确认官方导出结构（version、kind、app、workflow 字段）
- 需要确认版本兼容性规则
- 需要查询官方节点类型完整枚举（30 个）
- 需要确认依赖类型（marketplace/package/github）
- 需要确认输入变量类型列表

---

## 目录

- 参考源码
- 目标规则
- 导入与版本兼容性
- 导出结构
- 导出清理
- 依赖类型与来源
- 官方节点类型集
- 通用节点和边元数据
- 输入变量类型
- 公共样本可用性

## 参考源码

- Dify DSL 版本常量：
  https://github.com/langgenius/dify/blob/main/api/constants/dsl_version.py
- Dify 应用导入/导出服务：
  https://github.com/langgenius/dify/blob/main/api/services/app_dsl_service.py
- Dify 插件依赖分析：
  https://github.com/langgenius/dify/blob/main/api/services/plugin/dependencies_analysis.py
- Dify 插件依赖实体：
  https://github.com/langgenius/dify/blob/main/api/core/plugin/entities/plugin.py
- Dify 工作流前端类型：
  https://github.com/langgenius/dify/blob/main/web/app/components/workflow/types.ts
- Dify 工作流通用节点注册表：
  https://github.com/langgenius/dify/blob/main/web/app/components/workflow/constants/node.ts

## 目标规则

- 新生成的 Dify 应用 DSL 应使用 `version: "0.6.0"`
- `version` 保持引号包裹。Dify 导入期望字符串，拒绝非字符串值
- 使用 `kind: app`
- 新工作优先使用基于图的 `workflow` 和 `advanced-chat` 模式
- 公共 DSL 仅作为兼容性和工作流设计参考。大多数采样的公共仓库是遗留导出；后续扫描发现了少量公共 `0.6.0` 样本，但官方源码仍是目标权威

## 导入与版本兼容性

Dify 将导入的 DSL 版本与当前版本比较：

- 相同版本或相同 minor/micro 兼容的旧版本：正常导入
- 旧 minor 版本：可完成导入但有警告
- 旧 major 版本或比当前更新的版本：导入可能需要用户确认或迁移
- 缺失 `version` 会被导入逻辑填充为旧版 `0.1.0`，但生成的 DSL 不应依赖此回退

## 导出结构

官方导出构建以下顶层结构：

```yaml
version: "0.6.0"
kind: app
app:
  name: "应用名称"
  mode: advanced-chat
  icon: "🤖"
  icon_type: emoji
  icon_background: "#FFEAD5"
  description: ""
  use_icon_as_answer_icon: false
dependencies: []
workflow:
  graph:
    nodes: []
    edges: []
```

`advanced-chat` 和 `workflow` 模式导出追加 `workflow`。`chat`、`agent-chat` 和 `completion` 模式导出追加顶层 `model_config`。

## 导出清理

官方导出移除或重置工作区特定字段：

- 除非显式包含密钥，否则移除工具节点的 `credential_id`
- 除非显式包含密钥，否则移除 Agent 工具的 `credential_id`
- 触发器调度配置重置为默认导出形式
- 触发器 webhook 的 `webhook_url` 和 `webhook_debug_url` 被清空
- 触发器插件的 `subscription_id` 被清空
- 知识检索的 `dataset_ids` 可能在导出时加密、导入时解密，取决于 Dify 服务器配置

不要在公共模板中硬编码真实凭据、webhook URL、订阅 ID、数据库密码或私有数据集 ID。

## 依赖类型

官方依赖条目可以是 `marketplace`、`package` 或 `github`。

依赖条目格式（marketplace / package / github）的完整 YAML 示例见 `dsl-structure.md` 的 Dependencies 章节。

远程插件不可通过官方 Dify 导出逻辑导出。

## 依赖来源

Dify 从以下位置提取依赖：

- LLM 节点：`model.provider`
- 问题分类器节点：`model.provider`
- 参数提取器节点：`model.provider`
- 工具节点：`provider_id`
- 知识检索节点：reranking 模型、加权分数 embedding provider 或单一检索模型 provider
- 模型配置应用：主模型、数据集 reranking 模型和 Agent 工具

Agent 节点策略/工具依赖在实践中可能比此列表更复杂。尽可能保留导出的 `dependencies`。

## 官方节点类型集

当前工作流前端类型枚举（30 个）：

```text
start, end, answer, llm, knowledge-retrieval, question-classifier, if-else,
code, template-transform, http-request, variable-assigner,
variable-aggregator, tool, parameter-extractor, iteration,
document-extractor, list-operator, iteration-start, assigner, agent, loop,
loop-start, loop-end, human-input, datasource, datasource-empty,
knowledge-index, trigger-schedule, trigger-webhook, trigger-plugin
```

通用节点注册表当前暴露以下常见创作默认值：

```text
llm, knowledge-retrieval, agent, question-classifier, if-else, iteration,
iteration-start, loop, loop-start, loop-end, code, template-transform,
variable-aggregator, document-extractor, assigner, parameter-extractor,
http-request, list-operator, tool, human-input
```

触发器、数据源和知识库索引节点存在但更敏感于部署。生产使用优先从目标 Dify 工作区复制新导出。

## 通用节点和边元数据

官方通用节点数据可包含：

```text
isInIteration, iteration_id, isInLoop, loop_id, error_strategy,
retry_config, default_value, credential_id, subscription_id, provider_id
```

官方边数据可包含：

```text
isInIteration, iteration_id, isInLoop, loop_id, sourceType, targetType
```

新图 DSL 在每条边上包含 `sourceType` 和 `targetType`，并保持与各端点节点 `data.type` 同步。

## 输入变量类型

当前前端输入变量类型：

```text
text-input, paragraph, select, number, url, files, json, json_object,
contexts, iterator, file, file-list, loop, checkbox
```

在 start 变量、human-input 表单和节点输入 schema 中使用这些精确字符串（当节点支持时）。

## 公共样本可用性

2026-05-07 版本扫描：

- `BannyLon/DifyAIA`：38 个解析的应用 DSL，`0.6.0`：0
- `svcvit/Awesome-Dify-Workflow`：45 个解析的应用 DSL，`0.6.0`：0
- `wwwzhouhui/dify-for-dsl`：89 个解析的应用 DSL，`0.6.0`：0
- 本地检查的官方 Dify 工作流 fixture：24 个解析的应用 DSL，版本 `0.3.1` 到 `0.5.0`，`0.6.0`：0

2026-05-11 附加扫描：

- `g-krishna0/dify-export-test`：87 个解析的应用 DSL，`0.6.0`：0
- `Petrus-Han/dify-usecase-playground`：3 个解析的应用 DSL，`0.6.0`：1

因此，该技能从官方源码定位 0.6.0，并仅将公共 YAML 语料库用于真实图模式、触发器工作流示例、遗留导入行为和工具节点形状多样性。
