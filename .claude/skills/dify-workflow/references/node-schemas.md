# 节点 Schema

以下所有代码片段展示的是 `data:` 载荷。使用时需按照 `dsl-structure.md` 中的标准节点包装结构进行包裹。

## 目录

- 基础流程：start、end、answer
- 推理与转换：llm、code、template-transform
- 分支与提取：if-else、question-classifier、parameter-extractor
- 外部访问：http-request、tool、datasource
- 变量与文档：variable-aggregator、assigner、document-extractor、list-operator
- 知识库：knowledge-retrieval、knowledge-index
- Agent 与容器：agent、iteration、loop
- 人工输入/触发/画布节点：human-input、trigger-schedule、trigger-webhook、trigger-plugin、custom-note

## start

```yaml
title: "开始"
type: start
variables:
  - label: "输入文本"
    variable: input_text
    type: paragraph       # text-input | paragraph | select | number | url | file | file-list | json | checkbox
    required: true
    max_length: 50000
```

advanced-chat 模式下，start 变量通常为空，用户输入来自 `{{#sys.query#}}` 和 `{{#sys.files#}}`。

## end

```yaml
title: "结束"
type: end
outputs:
  - variable: result
    value_selector: ["1770000000001", text]
```

用于 workflow 模式应用。

## answer

```yaml
title: "直接回复"
type: answer
answer: "{{#1770000000001.text#}}"
variables: []
```

用于 advanced-chat 模式应用。多个分支可以各自以独立的 answer 节点结束。

## llm

```yaml
title: "LLM"
type: llm
model:
  provider: langgenius/tongyi/tongyi
  name: qwen3.5-flash
  mode: chat
  completion_params:
    temperature: 0.3
prompt_template:
  - id: 9e05cb8e-0000-4000-9000-000000000001
    role: system
    text: "You are a precise assistant."
  - id: 9e05cb8e-0000-4000-9000-000000000002
    role: user
    text: "{{#sys.query#}}"
context:
  enabled: false
  variable_selector: []
memory:
  query_prompt_template: "{{#sys.query#}}\n\n{{#sys.files#}}"
  role_prefix:
    assistant: ""
    user: ""
  window:
    enabled: false
    size: 50
vision:
  enabled: false
selected: false
```

当需要将知识检索结果传入 LLM 时，设置 `context.enabled: true`。

## code

```yaml
title: "清洗 JSON"
type: code
code_language: python3
code: |
  import json

  def main(text: str) -> dict:
      return {"result": json.loads(text)}
variables:
  - value_selector: ["1770000000001", text]
    variable: text
outputs:
  result:
    type: object
    children: null
desc: ""
selected: false
```

**关键要点**：
- `code` 字段必须使用 YAML 多行字符串格式 `|`，不要使用单行转义字符串（会导致 Dify 前端渲染失败）
- `outputs` 中每个字段需要 `children: null`
- 需要 `desc: ""` 字段（即使为空）
- 函数必须命名为 `main`；返回值必须是 dict，且 key 与 `outputs` 中定义的字段匹配

## if-else

```yaml
title: "条件分支"
type: if-else
cases:
  - id: "true"
    case_id: "true"
    logical_operator: and
    conditions:
      - id: 0b5f09ef-0000-4000-9000-000000000001
        variable_selector: ["1770000000000", input_text]
        comparison_operator: contains
        value: "合同"
        varType: string
selected: false
```

**关键要点**：
- 分支边的 `sourceHandle` 必须与目标 case 的 ID 一致
- Dify 的 IF/ELSE **只支持两个分支**：`true` 和 `false`
- 常用比较运算符：`contains`、`not contains`、`is`、`is not`、`empty`、`not empty`、`start with`、`end with`、`=`、`≠`、`>`、`<`、`≥`、`≤`

**多分支实现方式**：

当需要超过 2 个分支时（如 A/B/C/D），需要串联多个 IF/ELSE 节点：

```
                  ┌─ YES → 分支A处理
                  │
起始 → 是否A？ ──┤
                  │
                  └─ NO → 是否B？ ──┬─ YES → 分支B处理
                                     │
                                     └─ NO → 是否C？ ──┬─ YES → 分支C处理
                                                        │
                                                        └─ NO → 分支D处理（默认）
```

**边连接示例**（3 分支）：
```yaml
edges:
  # 第一个 IF/ELSE：true 分支走 A，false 走下一个判断
  - source: "router_A"
    sourceHandle: "true"
    target: "branch_A"
  - source: "router_A"
    sourceHandle: "false"
    target: "router_B"

  # 第二个 IF/ELSE：true 分支走 B，false 走 C
  - source: "router_B"
    sourceHandle: "true"
    target: "branch_B"
  - source: "router_B"
    sourceHandle: "false"
    target: "branch_C"
```

## question-classifier

```yaml
title: "问题分类器"
type: question-classifier
model:
  provider: langgenius/tongyi/tongyi
  name: qwen3.5-flash
  mode: chat
  completion_params:
    temperature: 0
query_variable_selector: ["sys", query]
classes:
  - id: "1"
    name: "上传文件"
  - id: "2"
    name: "查询历史"
instruction: "将用户问题分类到最合适的一类。"
vision:
  enabled: false
selected: false
```

分类器分支边的 handle 即为分类 ID。

## parameter-extractor

```yaml
title: "参数提取器"
type: parameter-extractor
instruction: "从输入中提取 SQL；没有 SQL 时返回空字符串。"
model:
  provider: langgenius/tongyi/tongyi
  name: qwen3.5-plus
  mode: chat
  completion_params:
    temperature: 0.1
parameters:
  - name: SQL
    description: "SQL query"
    required: false
    type: string
query: ["1770000000001", text]
reasoning_mode: prompt
vision:
  enabled: false
selected: false
```

在 LLM 之后使用，用于在进入 `if-else` 或工具调用之前提取严格的结构化字段。

## http-request

```yaml
title: "HTTP 请求"
type: http-request
method: post
url: "https://api.example.com/parse"
headers: "Content-Type: application/json"
params: ""
body:
  type: json
  data:
    - id: body-1
      key: text
      type: string
      value: "{{#1770000000000.input_text#}}"
authorization:
  type: no-auth
  config: null
timeout:
  max_connect_timeout: 10
  max_read_timeout: 60
  max_write_timeout: 10
retry_config:
  retry_enabled: false
selected: false
```

**无请求体时的格式**：
```yaml
body:
  type: none
  data: []
```

**关键要点**：
- `authorization` 需要 `config: null` 字段
- `timeout` 字段名是 `max_connect_timeout`、`max_read_timeout`、`max_write_timeout`
- 常用输出字段：`body`、`status_code`、`headers`、`files`

## template-transform

```yaml
title: "模板"
type: template-transform
variables:
  - variable: rows
    value_selector: ["1770000000001", data]
template: |
  {% for row in rows %}
  - {{ row.file_name }}: {{ row.document_summary }}
  {% endfor %}
selected: false
```

输出字段为 `output`。

## variable-aggregator

```yaml
title: "汇总"
type: variable-aggregator
output_type: string
variables:
  - ["1770000000001", text]
  - ["1770000000002", text]
selected: false
```

用于在 `end` 节点之前合并互斥分支的输出。

## assigner / variable-assigner

```yaml
title: "写入会话变量"
type: assigner
items:
  - variable_selector: [conversation, Memory]
    input_type: variable
    value_selector: ["1770000000001", output]
    operation: over-write
selected: false
```

较新版本的导出文件使用 `variable-assigner`；编辑已有文件时请沿用目标工作区的导出风格。

## document-extractor

```yaml
title: "文档提取器"
type: document-extractor
variable_selector: ["sys", files]
selected: false
```

输出字段通常为 `text`。处理文件列表时，如需获取第一个文件的元数据，可搭配 code/list 节点使用。

## list-operator

```yaml
title: "取第一个文件"
type: list-operator
filter_by:
  enabled: false
order_by:
  enabled: false
extract_by:
  enabled: true
  serial: first
var_type: array[file]
variable: ["sys", files]
selected: false
```

用于在 code/tool 节点之前进行文件列表或数组提取。

## knowledge-retrieval

```yaml
title: "知识检索"
type: knowledge-retrieval
dataset_ids:
  - "dataset-id"
query_variable_selector: ["sys", query]
retrieval_mode: multiple
multiple_retrieval_config:
  top_k: 4
  reranking_enable: false
  score_threshold_enabled: false
  score_threshold: 0
metadata_filtering_mode: disabled
selected: false
```

通过 `context.variable_selector` 将检索结果传入 LLM。

## tool

```yaml
title: "工具"
type: tool
provider_id: langgenius/firecrawl/firecrawl
provider_name: langgenius/firecrawl/firecrawl
provider_type: builtin   # builtin | api | workflow | mcp
plugin_id: langgenius/firecrawl
plugin_unique_identifier: langgenius/firecrawl:0.0.5@...
tool_name: scrape
tool_label: "单页面抓取"
tool_description: "Scrape a page"
tool_node_version: "2"
tool_configurations:
  formats:
    type: constant
    value: markdown
tool_parameters:
  url:
    type: mixed
    value: "https://example.com?q={{#sys.query#}}"
selected: false
```

**Tool 节点 vs Agent 内工具配置**：

| 场景 | 工具类型字段 | 配置字段 |
|------|-------------|---------|
| 独立 Tool 节点 | `provider_type` | `tool_configurations` + `tool_parameters` |
| Agent 内工具 | `type` | `settings` + `parameters` (auto 格式) |

工具输出字段取决于具体插件。常见字段有 `text`、`data`、`result`、`json`、`files` 和 `output`。

`plugin_id`、`plugin_unique_identifier` 和 `tool_node_version` 在较新的 marketplace/package 导出中常见，但并非所有工具都有。内置工具、MCP、API 和 workflow 工具可能会省略这些字段。从导出文件复制时请保留这些字段；不要凭空编造插件标识符。

## agent

```yaml
title: "Agent"
type: agent
tool_node_version: "2"
agent_strategy_provider_name: langgenius/agent/agent
agent_strategy_name: function_calling
agent_strategy_label: FunctionCalling
agent_parameters:
  model:
    type: constant
    value:
      provider: langgenius/tongyi/tongyi
      name: qwen3.5-flash
      mode: chat
      completion_params:
        temperature: 0.3
  query:
    type: constant
    value: "{{#sys.query#}}"
  instruction:
    type: constant
    value: "Answer with tool support when useful."
  tools:
    type: constant
    value: []
output_schema: null
selected: true
```

**关键要点**：
- 需要 `tool_node_version: "2"` 字段
- 需要 `selected: true` 字段（不是 `selected: false`）

### Agent 工具配置模板

当 Agent 需要挂载工具时，`tools.value` 使用以下格式：

```yaml
tools:
  type: constant
  value:
  - enabled: true
    # === 通用字段（所有工具都适用） ===
    type: builtin                              # 工具类型：builtin | api | workflow | mcp
    provider_name: "{plugin_id}"               # 插件 ID，如 langgenius/tavily/tavily
    provider_show_name: "{plugin_id}"          # 显示名称，通常与 provider_name 相同
    tool_name: "{tool_name}"                   # 工具名称，如 tavily_search
    tool_label: "{显示标签}"                    # 工具显示名称
    tool_description: "{工具描述}"              # 工具描述
    extra:
      description: "{工具描述}"                 # 额外描述

    # === 参数配置（使用 auto 格式） ===
    parameters:
      {param_name}:
        auto: 1                                # 1 = 自动填充，0 = 手动指定
        value: null                            # auto=1 时为 null

    # === 工具特定配置 ===
    settings:
      {setting_name}:
        value:
          type: constant
          value: {值}
```

**Tavily Search 示例**：

```yaml
tools:
  type: constant
  value:
  - enabled: true
    type: builtin
    provider_name: langgenius/tavily/tavily
    provider_show_name: langgenius/tavily/tavily
    tool_name: tavily_search
    tool_label: "Tavily Search"
    tool_description: "搜索引擎工具"
    extra:
      description: "搜索引擎工具"
    parameters:
      query:
        auto: 1
        value: null
      search_depth:
        auto: 1
        value: null
      max_results:
        auto: 1
        value: null
    settings:
      max_results:
        value:
          type: constant
          value: 5
      search_depth:
        value:
          type: constant
          value: advanced
```

**字段说明**：

| 字段 | 说明 | 是否通用 |
|------|------|---------|
| `type` | 工具类型（builtin/api/workflow/mcp） | ✅ 通用 |
| `provider_name` | 插件提供者 ID | ✅ 通用格式，值因插件而异 |
| `provider_show_name` | 显示名称 | ✅ 通用 |
| `tool_name` | 工具名称 | ✅ 通用格式，值因工具而异 |
| `tool_label` | 工具显示标签 | ✅ 通用 |
| `tool_description` | 工具描述 | ✅ 通用 |
| `extra.description` | 额外描述 | ✅ 通用 |
| `enabled` | 是否启用 | ✅ 通用 |
| `parameters` + `auto: 1` | 参数自动填充 | ✅ 通用 |
| `settings` | 工具特定配置 | ⚠️ 结构通用，内容因工具而异 |

**注意**：`settings` 中的配置项因工具而异，需要从 Dify 导出文件中复制，不要凭空编造。

## iteration

```yaml
title: "迭代"
type: iteration
iterator_selector: ["1770000000001", items]
output_selector: ["1770000000003", output]
output_type: array[string]
is_parallel: false
parallel_nums: 10
start_node_id: "1770000000002"
selected: false
```

导出的 Dify 图结构中还包含一个 `iteration-start` 辅助节点（wrapper `type: custom-iteration-start`），以及标记了 `isInIteration` 的内部子节点。

### Iteration 完整内部结构示例

以下是 iteration 节点包含一个 code 子节点的完整导出结构（节点 + 边）：

```yaml
# 1. iteration 容器节点
- data:
    title: "遍历文件列表"
    type: iteration
    iterator_selector: ["1770000000000", files]
    output_selector: ["1770000000002", result]
    output_type: array[string]
    is_parallel: false
    parallel_nums: 10
    start_node_id: "1770000000003"
    selected: false
  id: "1770000000001"
  position: { x: 400, y: 300 }
  positionAbsolute: { x: 400, y: 300 }
  selected: false
  type: custom
  width: 243
  height: 89

# 2. iteration-start 辅助节点（容器内必须有）
- data:
    title: ""
    type: ""
    selected: false
  id: "1770000000003"
  position: { x: 50, y: 50 }
  positionAbsolute: { x: 450, y: 350 }
  selected: false
  type: custom-iteration-start
  width: 44
  height: 44

# 3. 容器内的子节点（标记 isInIteration: true）
- data:
    title: "处理单个文件"
    type: code
    code_language: python3
    code: |
      def main(file: dict) -> dict:
          return {"result": file.get("name", "")}
    variables:
      - value_selector: ["1770000000001", item]
        variable: file
    outputs:
      result:
        type: string
    isInIteration: true
    iteration_id: "1770000000001"
    selected: false
  id: "1770000000002"
  position: { x: 200, y: 50 }
  positionAbsolute: { x: 600, y: 350 }
  selected: false
  type: custom
  width: 243
  height: 89
```

容器内边的结构：

```yaml
# iteration-start → 子节点（sourceType 为 iteration-start，不是 start）
- data:
    isInLoop: false
    isInIteration: true
    iteration_id: "1770000000001"
    sourceType: iteration-start
    targetType: code
  id: 1770000000003-source-1770000000002-target
  selected: false
  source: "1770000000003"
  sourceHandle: source
  target: "1770000000002"
  targetHandle: target
  type: custom
  zIndex: 0
```

关键要点：
- 容器外的边 `isInIteration: false`，容器内的边 `isInIteration: true` 且带 `iteration_id`
- `iteration-start` 的 wrapper type 是 `custom-iteration-start`，边的 `sourceType` 为 `iteration-start`
- 子节点通过 `data` 中的 `isInIteration` 和 `iteration_id` 归属容器
- 子节点引用迭代项用 `["iteration_node_id", "item"]`

## loop

```yaml
title: "循环"
type: loop
loop_count: 5
break_conditions:
  - id: 6db087e6-0000-4000-9000-000000000001
    variable_selector: ["1770000000003", done]
    comparison_operator: is
    value: "true"
    varType: boolean
start_node_id: "1770000000002"
selected: false
```

导出的 loop 内部结构使用 `loop-start`，子节点和边标记 `isInLoop`。Dify 当前的节点枚举还包含 `loop-end`；当 loop 包含嵌套分支或多个退出路径时，请从导出文件中复制完整的 loop 内部结构。

### Loop 完整内部结构示例

以下是 loop 节点包含一个 code 子节点的完整导出结构（节点 + 边）：

```yaml
# 1. loop 容器节点
- data:
    title: "重试循环"
    type: loop
    loop_count: 5
    break_conditions:
      - id: 6db087e6-0000-4000-9000-000000000001
        variable_selector: ["1770000000002", done]
        comparison_operator: is
        value: "true"
        varType: boolean
    start_node_id: "1770000000003"
    selected: false
  id: "1770000000001"
  position: { x: 400, y: 300 }
  positionAbsolute: { x: 400, y: 300 }
  selected: false
  type: custom
  width: 243
  height: 89

# 2. loop-start 辅助节点
- data:
    title: ""
    type: ""
    selected: false
  id: "1770000000003"
  position: { x: 50, y: 50 }
  positionAbsolute: { x: 450, y: 350 }
  selected: false
  type: custom-loop-start
  width: 44
  height: 44

# 3. loop-end 辅助节点
- data:
    title: ""
    type: ""
    selected: false
  id: "1770000000004"
  position: { x: 400, y: 50 }
  positionAbsolute: { x: 650, y: 350 }
  selected: false
  type: custom-loop-end
  width: 44
  height: 44

# 4. 容器内的子节点（标记 isInLoop: true）
- data:
    title: "检查状态"
    type: code
    code_language: python3
    code: |
      def main(prev_result: str) -> dict:
          done = prev_result == "success"
          return {"done": done, "output": prev_result}
    variables:
      - value_selector: ["1770000000001", item]
        variable: prev_result
    outputs:
      done:
        type: boolean
      output:
        type: string
    isInLoop: true
    loop_id: "1770000000001"
    selected: false
  id: "1770000000002"
  position: { x: 200, y: 50 }
  positionAbsolute: { x: 600, y: 350 }
  selected: false
  type: custom
  width: 243
  height: 89
```

容器内边的结构：

```yaml
# loop-start → 子节点（sourceType 为 loop-start）
- data:
    isInLoop: true
    isInIteration: false
    loop_id: "1770000000001"
    sourceType: loop-start
    targetType: code
  id: 1770000000003-source-1770000000002-target
  selected: false
  source: "1770000000003"
  sourceHandle: source
  target: "1770000000002"
  targetHandle: target
  type: custom
  zIndex: 0

# 子节点 → loop-end（sourceType 为 code，targetType 为 loop-end）
- data:
    isInLoop: true
    isInIteration: false
    loop_id: "1770000000001"
    sourceType: code
    targetType: loop-end
  id: 1770000000002-source-1770000000004-target
  selected: false
  source: "1770000000002"
  sourceHandle: source
  target: "1770000000004"
  targetHandle: target
  type: custom
  zIndex: 0
```

关键要点：
- loop 与 iteration 类似，但额外支持 `break_conditions` 和 `loop-end` 辅助节点
- `loop-start` wrapper type 是 `custom-loop-start`，边的 `sourceType` 为 `loop-start`
- `loop-end` wrapper type 是 `custom-loop-end`，边的 `targetType` 为 `loop-end`
- 子节点和边通过 `isInLoop` + `loop_id` 归属容器
- `break_conditions` 中的 `variable_selector` 引用子节点输出（如 `["1770000000002", done]`），节点 ID 必须是容器内的实际子节点

## datasource

```yaml
title: "数据源"
type: datasource
provider_id: langgenius/notion/notion
provider_name: langgenius/notion/notion
provider_type: builtin
datasource_name: pages
datasource_label: "Pages"
datasource_parameters: {}
selected: false
```

数据源节点由插件提供支持；请尽可能从实际导出文件中复制。

## knowledge-index

```yaml
title: "知识库写入"
type: knowledge-index
dataset_id: "dataset-id"
index_chunk_variable_selector: ["1770000000001", text]
keyword_number: 10
retrieval_model:
  top_k: 3
  score_threshold_enabled: false
  score_threshold: 0.5
selected: false
```

仅在目标 Dify 版本支持工作流中的知识库索引功能时使用。

## human-input / human-feedback

```yaml
title: "人工确认"
type: human-input
variables:
  - label: "是否继续"
    variable: approved
    type: select
    required: true
    options:
      - label: "继续"
        value: "yes"
      - label: "停止"
        value: "no"
selected: false
```

这些节点对版本敏感。生产环境 DSL 建议从目标 Dify 工作区的导出文件中复制。

## trigger-schedule

```yaml
title: "定时触发"
type: trigger-schedule
mode: visual
frequency: daily
timezone: Asia/Shanghai
visual_config:
  time: "09:00 AM"
cron_expression: ""
selected: false
```

定时触发支持 `mode: visual`（可视化）或 `mode: cron`（表达式）；可视化频率可选 `hourly`、`daily`、`weekly` 或 `monthly`。官方导出会重置调度配置，生产环境请从当前工作区导出文件复制。

## trigger-webhook

```yaml
title: "Webhook 触发"
type: trigger-webhook
webhook_url: ""
webhook_debug_url: ""
method: POST
content_type: application/json
headers: []
params: []
body: []
async_mode: true
status_code: 200
response_body: ""
variables: []
selected: false
```

官方导出会清空 webhook URL。公开 DSL 中请留空，由目标工作区在导入后生成或配置。

## trigger-plugin

```yaml
title: "插件事件触发"
type: trigger-plugin
provider_id: author/plugin/provider
provider_name: author/plugin/provider
provider_type: builtin
plugin_id: author/plugin
plugin_unique_identifier: author/plugin:0.0.1@...
event_name: event_name
event_label: "Event label"
event_parameters: {}
event_configurations: {}
output_schema: {}
config: {}
selected: false
```

官方导出会清空 `subscription_id`。插件触发器高度依赖具体插件；如需保证生产可靠性，请从目标 Dify 工作区获取最新的最小导出文件。

## custom-note

```yaml
title: ""
type: ""
text: "Canvas note"
theme: blue
author: ""
showAuthor: false
width: 260
height: 120
selected: false
```

节点 wrapper type 为 `custom-note`。注释是有效的非执行画布标注，`data.type` 可以为空。
