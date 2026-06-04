# 插件市场工具节点

当工作流需要使用非内置工具（插件、MCP、workflow-provider）时，加载此文件获取工具节点模板和决策逻辑。

## 适用场景

- 用户需要的工具不在 node-schemas.md 的内置列表中
- 需要从 Dify Marketplace 添加插件工具
- 需要评估工具节点的可靠性
- 需要处理插件授权和密钥

---

## 目录

- 可靠性阶梯
- 新工具推荐流程
- 工具节点身份模板
- `paramSchemas` 指引
- 授权与密钥
- 何时拒绝担保

## 可靠性阶梯

| 可用证据 | 可靠性 | 操作方式 |
| --- | --- | --- |
| 用户 Dify 工作区的最小导出 DSL | 最高 | 复制节点外壳、dependency、`paramSchemas`、`tool_configurations` 和 `tool_parameters`；仅修改 value 字段。 |
| 插件源码仓库或 `.difypkg` 包 | 高 | 读取 `manifest.yaml`、provider YAML、tool YAML 和 Python 实现；推断 DSL 节点并提醒仍需导入测试。 |
| 仅官方 marketplace 页面 | 中 | 使用可见的插件/版本/工具信息（如有），但标记节点为候选，因为参数 schema 和授权细节可能不完整。 |
| 仅工具名称 | 低 | 不要声称可用。要求导出、源码/包或截图。仅在用户明确接受风险时生成占位/草稿。 |

## 新工具推荐流程

1. 尽可能要求最小导出：创建空白 Dify 工作流，添加目标工具节点，配置授权和一个示例参数，导出 DSL。
2. 从导出中提取以下字段：
   `dependencies`、`provider_id`、`provider_name`、`provider_type`、`plugin_id`、
   `plugin_unique_identifier`、`tool_name`、`tool_label`、`tool_description`、
   `tool_node_version`、`paramSchemas`、`params`、`tool_configurations`、`tool_parameters`。
3. 保持复制的身份字段不变。仅修改参数 `value` 字段、节点 ID、标题和图位置。
4. 从插件源码/包推断时，按以下方式映射 schema 字段：
   - 插件 manifest/package 身份 → `plugin_id` 和 dependency 条目
   - provider 身份 → `provider_id` 和 `provider_name`
   - tool YAML name → `tool_name`
   - tool labels/descriptions → `tool_label` 和 `tool_description`
   - `form: llm` 的工具参数 → `tool_parameters`
   - `form: form` 的工具参数 → `tool_configurations`
5. 生成 DSL 后，运行 `scripts/validate_dsl.py`。如果节点不是从导出复制的，告知用户仍需在 Dify 中做导入/运行测试。

## 工具节点身份模板

```yaml
title: "工具标题"
type: tool
is_team_authorization: true
provider_id: author/plugin/provider
provider_name: author/plugin/provider
provider_type: builtin
plugin_id: author/plugin
plugin_unique_identifier: author/plugin:0.0.1@sha256-or-marketplace-id
tool_name: tool_name_from_plugin
tool_label: "中文标签"
tool_description: "工具功能描述"
tool_node_version: "2"
tool_configurations: {}
tool_parameters:
  query:
    type: mixed
    value: "{{#sys.query#}}"
selected: false
```

不要为生产 DSL 编造 `plugin_unique_identifier`。如果未知，要么获取导出/源码，要么将工作流标记为尽力草稿。

真实导出可能省略节点内的 `plugin_id`、`plugin_unique_identifier` 和 `tool_node_version`。如果顶层 dependency 或目标工作区/工具授权提供了缺失的身份，它们仍然有效。保留导出形状，而不是将所有内容标准化为统一 schema。

依赖条目格式（marketplace / package / github）见 `dsl-structure.md` 的 Dependencies 章节。

官方 Dify 导出拒绝远程插件。如果目标工作流依赖远程插件，要求用户在期望可移植 DSL 导出/导入前替换为 marketplace、package 或 GitHub 安装方式。

## `paramSchemas` 指引

Dify 导出的许多工具节点带有详细的 `paramSchemas` 列表和 `params` 映射。这些字段保留了工具编辑器 UI 并使导入更忠实。

如果复制的导出包含它们，保留它们。如果从源码手写：

```yaml
paramSchemas:
  - name: query
    type: string
    required: true
    form: llm
    label:
      en_US: Query
      zh_Hans: 查询
    human_description:
      en_US: The query to execute.
      zh_Hans: 要执行的查询。
    llm_description: The query to execute.
    options: []
    default: null
    placeholder: null
    min: null
    max: null
    precision: null
    scope: null
    template: null
params:
  query: ""
```

许多工具没有 `paramSchemas` 也能被 Dify 导入，但复制或重建它们可以减少 UI 和兼容性问题。

## 授权与密钥

- Marketplace 插件通常需要用户/团队授权。DSL 中的 `is_team_authorization` 不是凭据。
- `provider_type` 可以是 `builtin`、`api`、`workflow` 或 `mcp`；MCP 和 workflow 工具通常不能清晰映射到 marketplace dependency。
- 不要在公共 DSL 中硬编码访问令牌、应用密钥、数据库密码或私有 webhook token。
- 优先使用 Dify 插件授权、环境变量或占位符值。
- 如果工具可以接受覆盖凭据参数（如 `api_key` 或 `db_uri`），通过 `{{#env.NAME#}}` 设置。

## 何时拒绝担保

当以下情况时，明确说明工具节点无法保证可导入：

- 插件/版本未知
- 确切的 `provider_id` 或 `tool_name` 未知
- 必需的参数名称或 `form` 类型未知
- 插件需要 DSL 中未表示的授权
- marketplace 页面未暴露足够的 schema 细节

在这些情况下，提供草稿加简短测试计划：

1. 将 DSL 导入 Dify 测试工作区
2. 如果 Dify 提示，重新选择或授权插件
3. 打开工具节点，确认所有必填字段已映射
4. 运行最小测试输入，检查工具输出字段
5. 更新下游 selector（`text`、`data`、`result`、`json`、`files`、`output`）以匹配实际输出
