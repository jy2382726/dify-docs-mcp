# Dify DSL 结构

本参考文档涵盖应用级结构、图连线、变量、依赖和导入/导出兼容性。

## 目录

- 官方基线
- 顶层 YAML
- 模式规则
- 依赖
- 官方节点类型集
- 节点和边包装器
- 变量引用
- 工作流变量
- 导入敏感细节
- Agent-chat / 模型配置应用

## 官方基线

- Dify 源码声明 `CURRENT_APP_DSL_VERSION = "0.6.0"`。
- 导入时期望 `version` 为字符串类型。
- 新生成的 DSL 应以 `version: 0.6.0` 为目标（**不带引号**），除非用户明确要求兼容旧版 Dify 工作区。
- Dify 可在 `dependencies` 缺失时为非常旧的导入版本（`<=0.1.5`）回填最新依赖。对于新 DSL，显式声明依赖对跨工作区导入更安全。
- **dependencies 可以为空数组** `[]`，不需要声明每个插件依赖。
- 导出的工作流图镜像 ReactFlow 结构：每个节点有一个外部包装器和一个 `data` 对象；边通过 `source`/`target` 节点 ID 和 handle 进行连接。

本 Skill 使用的主要资料来源：

- Dify DSL 版本常量：
  https://github.com/langgenius/dify/blob/main/api/constants/dsl_version.py
- Dify 应用 DSL 导入/导出服务：
  https://github.com/langgenius/dify/blob/main/api/services/app_dsl_service.py
- Dify 依赖分析：
  https://github.com/langgenius/dify/blob/main/api/services/plugin/dependencies_analysis.py
- Dify 工作流前端类型：
  https://github.com/langgenius/dify/blob/main/web/app/components/workflow/types.ts
- 示例工作流仓库：
  https://github.com/BannyLon/DifyAIA,
  https://github.com/svcvit/Awesome-Dify-Workflow,
  https://github.com/wwwzhouhui/dify-for-dsl,
  https://github.com/g-krishna0/dify-export-test,
  https://github.com/Petrus-Han/dify-usecase-playground

## 顶层 YAML

```yaml
app:
  name: "App name"
  description: "What this workflow does"
  icon: "🤖"
  icon_type: emoji
  icon_background: "#FFEAD5"
  mode: advanced-chat       # workflow | advanced-chat | chat | completion | agent-chat
  use_icon_as_answer_icon: false
kind: app
version: 0.6.0
dependencies: []
workflow:
  conversation_variables: []
  environment_variables: []
  features:
    file_upload:
      allowed_file_extensions: []
      allowed_file_types: []
      allowed_file_upload_methods:
      - local_file
      - remote_url
      enabled: false
      fileUploadConfig:
        attachment_image_file_size_limit: 2
        audio_file_size_limit: 50
        batch_count_limit: 5
        file_size_limit: 15
        file_upload_limit: 50
        image_file_batch_limit: 10
        image_file_size_limit: 10
        single_chunk_attachment_limit: 10
        video_file_size_limit: 100
        workflow_file_upload_limit: 10
      image:
        enabled: false
        number_limits: 3
        transfer_methods:
        - local_file
        - remote_url
      number_limits: 3
    opening_statement: ''
    retriever_resource:
      enabled: true
    sensitive_word_avoidance:
      enabled: false
    speech_to_text:
      enabled: false
    suggested_questions: []
    suggested_questions_after_answer:
      enabled: false
    text_to_speech:
      enabled: false
      language: ''
      voice: ''
  graph:
    nodes: []
    edges: []
    viewport:
      x: 0
      y: 0
      zoom: 0.8
```

## 模式规则

| 模式 | 典型用途 | 输入来源 | 终端节点 |
| --- | --- | --- | --- |
| `workflow` | 一次性执行、批处理、触发式或集成自动化 | 起始变量或触发载荷 | `end`；副作用触发流可能在工具节点结束 |
| `advanced-chat` | 支持多轮对话的 Chatflow | `sys.query`、`sys.files`、对话变量 | `answer` |
| `chat` | 旧版/简单聊天应用 | 模型配置 | 模型配置 |
| `completion` | 旧版补全应用 | 模型配置/起始输入 | 模型配置 |
| `agent-chat` | 旧版 Agent 聊天应用 | 模型配置 | 模型配置 |

新生成的 DSL 优先使用 `workflow` 或 `advanced-chat`。默认使用 `workflow`，除非用户需要 Chatflow 行为，如多轮记忆、`sys.query`、`sys.files` 或流式应答。

**advanced-chat 模式的多分支模式**：

在 `advanced-chat` 模式下，当工作流有多个分支时，**每个分支应该有独立的 `answer` 节点**，而不是汇聚到单个 `answer` 节点。这是 Dify Chatflow 的标准模式。

```
                 ┌─ 分支A处理 → Answer_A
                 │
起始 → 路由判断 ─┼─ 分支B处理 → Answer_B
                 │
                 └─ 分支C处理 → Answer_C
```

**错误示例**（会导致渲染问题）：
```
分支A处理 ─┐
分支B处理 ─┼─ 单个Answer（引用多个互斥变量）
分支C处理 ─┘
```

**正确示例**：
```
分支A处理 → Answer_A（只引用分支A的输出）
分支B处理 → Answer_B（只引用分支B的输出）
分支C处理 → Answer_C（只引用分支C的输出）
```

## 依赖

官方 Dify 应用 DSL 依赖类型可以是 `marketplace`、`package` 或 `github`。远程插件无法通过官方 Dify 导出逻辑导出。

Marketplace 插件依赖：

```yaml
dependencies:
  - current_identifier: null
    type: marketplace
    value:
      marketplace_plugin_unique_identifier: langgenius/tongyi:0.1.36@73e3e28eca163b96da65ef9eab8633f9e7257213ff0d2f2bed93b28b552d2cda
      version: null
```

Package/私有插件依赖：

```yaml
dependencies:
  - current_identifier: null
    type: package
    value:
      plugin_unique_identifier: wwwzhouhui/nano_banana2_text2image:0.0.1@4069e9bfe87735fcd0276e08b84eefdd87fb05e82aa5228ef44df17390a8239b
      version: null
```

GitHub 安装的插件依赖：

```yaml
dependencies:
  - current_identifier: null
    type: github
    value:
      repo: author/plugin-repo
      version: 0.0.1
      package: plugin-package-name
      github_plugin_unique_identifier: author/plugin:0.0.1@4069e9bfe87735fcd0276e08b84eefdd87fb05e82aa5228ef44df17390a8239b
```

尽可能使用源 Dify 工作区导出的精确 `marketplace_plugin_unique_identifier` 或 `plugin_unique_identifier`。如果手写公共模板，保持插件名称和版本的真实性，但需告知用户导入后可能需要在 Dify 中重新安装或重新选择插件。

常见依赖来源：

- LLM 提供商：`model.provider`
- 工具节点：用于依赖分析的 `provider_id`，以及节点导出时的 `plugin_unique_identifier`
- Agent 策略和 Agent 节点内的内置工具
- 知识检索/索引和数据源节点

官方导出的依赖提取目前从 LLM、问题分类器、参数提取器、知识检索的 rerank/embedding 配置、工具 `provider_id` 以及模型配置应用中读取模型提供商。当可用时保留导出的 `dependencies`，因为复杂的 Agent/插件配置可能比手写的依赖列表包含更多细节。

## 官方节点类型集

官方节点类型枚举（30 个）和输入变量类型列表见 `official-0.6-target.md` 的「Official Node Type Set」和「Input Variable Types」章节。

## 节点包装器

```yaml
- data:
    title: "Readable title"
    type: llm
    selected: false
  id: "1770000000000"
  position:
    x: 334
    y: 300
  positionAbsolute:
    x: 334
    y: 300
  selected: false
  sourcePosition: right
  targetPosition: left
  type: custom
  width: 243
  height: 89
```

使用字符串 ID。Dify 导出时将数字形式的 ID 作为字符串处理；保持引号包裹。

画布备注节点是不可执行的注释：

```yaml
- data:
    author: ""
    desc: "Implementation note"
    height: 120
    selected: false
    showAuthor: false
    text: "Explain a branch or TODO here."
    theme: blue
    title: ""
    type: ""
    width: 260
  id: "note-1"
  position: { x: 600, y: 120 }
  positionAbsolute: { x: 600, y: 120 }
  selected: false
  type: custom-note
```

当包装器 `type` 为 `custom-note` 时，不要将缺失/空的 `data.type` 视为错误。

## 边包装器

```yaml
- data:
    isInLoop: false
    isInIteration: false
    sourceType: start
    targetType: llm
  id: 1770000000000-source-1770000000001-target
  source: "1770000000000"
  sourceHandle: source
  target: "1770000000001"
  targetHandle: target
  type: custom
  zIndex: 0
```

Handle 约定：

- 线性边：`sourceHandle: source`，`targetHandle: target`
- `if-else`：source handle 为分支 ID（`"true"`、`"false"` 或 UUID）
- `question-classifier`：source handle 为分类 ID（`"1"`、`"2"`、...）
- 迭代/循环内部：当 Dify 导出时包含 `isInIteration` 或 `isInLoop` 以及父节点 ID 字段

**重要提示**：边的 `data.sourceType` 和 `data.targetType` 必须与源节点和目标节点的 `data.type` 一致。例如：
- 源节点 `data.type: if-else` → 边的 `sourceType: if-else`
- 目标节点 `data.type: code` → 边的 `targetType: code`

## 变量引用

Prompt 插值：

```text
{{#node_id.field#}}
{{#sys.query#}}
{{#sys.files#}}
{{#sys.user_id#}}
{{#conversation.Memory#}}
{{#env.API_KEY#}}
```

选择器字段：

```yaml
value_selector: ["1770000000001", text]
variable_selector: ["sys", query]
query: ["1770000000001", text]
```

部分导出的 DSL 在数组中使用点号形式的系统选择器，例如 `["sys.query"]`。新文件优先使用双元素选择器（`["sys", "query"]`），除非需要匹配已有导出。

## 工作流变量

对话变量用于 `advanced-chat` 的记忆/状态：

```yaml
conversation_variables:
  - description: ""
    id: 7a9ac6e3-0000-4000-9000-000000000001
    name: Memory
    selector: [conversation, Memory]
    value: ""
    value_type: string
```

较早的公共导出有时会省略对话变量的 `selector`。新 DSL 应包含它；审查旧 DSL 时，仅缺失 `selector` 不一定导致导入失败。

环境变量是只读的工作流常量：

```yaml
environment_variables:
  - description: "Base URL"
    id: 7a9ac6e3-0000-4000-9000-000000000002
    name: FILES_URL
    selector: [env, FILES_URL]
    value: "https://example.local/"
    value_type: string
```

支持的值类型通常包括 `string`、`number`、`boolean`、`object`、`array[string]`、`array[number]`、`array[object]`、`array[file]` 和 `file`。

## 导入敏感细节

- 为 `version`、节点 ID、分支 handle 和大数值形式的值加引号。部分真实 DSL 将 `version: 0.3.0` 不加引号，它通常也能解析为字符串，但加引号可避免 YAML 解析器差异。
- 保持 `sourceType`/`targetType` 与各端点节点的 `data.type` 同步。
- 不要省略终端节点：`workflow` 需要 `end`；`advanced-chat` 至少需要一个可达的 `answer`。
- `agent-chat`、`chat` 和 `completion` 可能使用顶层 `model_config` 而非 `workflow.graph`；这在旧版/公共导出中很常见。
- 不要硬编码真实的插件凭据。导出的插件授权通常保存在 Dify 中，而非 DSL 里。
- 从真实导出生成的工具节点通常包含 `paramSchemas` 和 `params`。它们较为冗长但提升了 UI 保真度。手写时保留必要的工具标识和参数，如果目标 Dify 导入报错，优先从导出中复制 `paramSchemas`。

## Agent-chat / 模型配置应用

部分公共 DSL，特别是 `agent-chat`，没有工作流图。它们使用顶层 `model_config`：

```yaml
app:
  mode: agent-chat
kind: app
version: "0.3.0"
model_config:
  agent_mode:
    enabled: true
    max_iteration: 10
    strategy: react
    tools:
      - enabled: true
        provider_id: pdftranslate-mcp-server
        provider_name: pdftranslate-mcp-server
        provider_type: mcp
        tool_label: translate_pdf
        tool_name: translate_pdf
        tool_parameters:
          file_input: ""
          filename: ""
  model:
    provider: langgenius/siliconflow/siliconflow
    name: deepseek-ai/DeepSeek-V3
    mode: chat
  user_input_form: []
```

新应用优先使用基于图的 `workflow` 或 `advanced-chat`，除非用户明确要求旧版/简单应用模式。审查现有 DSL 时，不要仅因为 `workflow.graph` 缺失就判定 `agent-chat` 应用失败。
