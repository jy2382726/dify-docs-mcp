# DSL 校验规则

Claude Code 生成 DSL YAML 后，按以下规则逐项校验。发现错误时自动修复；无法修复时返回 warning。

## 必检项（Error 级别）

| # | 规则 | 检查方式 |
|---|------|---------|
| 1 | YAML 解析成功 | 顶层必须是 mapping |
| 2 | version 是字符串 | `version: "0.6.0"` |
| 3 | app.mode 合法 | workflow / advanced-chat / chat / completion / agent-chat |
| 4 | workflow.graph.nodes 存在 | graph 模式（workflow / advanced-chat）必须有 nodes |
| 5 | workflow.graph.edges 存在 | graph 模式必须有 edges |
| 6 | 节点 ID 唯一 | 不允许重复 ID |
| 7 | 节点 data.type 存在 | 每个节点必须有类型 |
| 8 | 边引用的节点存在 | source 和 target 必须指向已有节点 |
| 9 | 边的 sourceType 匹配节点类型 | edge.data.sourceType == node.data.type |
| 10 | LLM 节点有 prompt_template | 必须是 list 类型 |
| 11 | Code 节点有 def main | Python 必须定义 main 函数；JS/TS 同理 |
| 12 | Code 节点有 outputs | 必须是 dict 类型 |
| 12a | Code 节点 code 字段格式 | 必须使用 YAML 多行字符串格式 `|`，不能使用单行转义字符串 |
| 12b | Code 节点 outputs 有 children | 每个 output 字段需要 `children: null` |
| 12c | Code 节点有 desc 字段 | 需要 `desc: ""`（即使为空） |
| 13 | Tool 节点必填字段 | provider_id, provider_name, provider_type, tool_name, tool_parameters |
| 13a | Agent 节点必填字段 | tool_node_version, agent_strategy_provider_name, agent_strategy_name, agent_parameters |
| 13b | Agent 工具配置格式 | tools.value 列表中每个工具需要 type, provider_name, tool_name |
| 14 | If-else 节点有 cases | 必须是 list 类型 |
| 15 | End 节点有 outputs | 必须是 list 类型 |
| 16 | 变量名唯一 | start/conversation/environment 变量不重复 |
| 17 | Answer 节点有 answer | answer 字段必须存在 |
| 18 | Parameter-extractor 有 parameters | 必须是 list 类型 |
| 19 | 边 ID 唯一 | 不允许重复边 ID |
| 20 | SQL 无尾部逗号 | `INSERT INTO t (a,) VALUES (1)` 语法错误 |
| 21 | dependencies 结构合法 | list 类型，每项 type 在 marketplace/package/github 中 |

## 警告项（Warning 级别）

| # | 规则 | 说明 |
|---|------|------|
| 1 | 顶层 kind 应为 'app' | 默认值，缺失不阻断 |
| 2 | LLM 节点有 model.provider 和 model.name | 缺失可能导入失败 |
| 3 | 边有 sourceHandle 和 targetHandle | 缺失可能导致连接异常 |
| 4 | 变量有 name 和 value_type | 缺失可能导入失败 |
| 5 | 变量引用指向已知节点 | `{{#node_id.field#}}` 中的 node_id 必须存在 |
| 6 | SQL 无危险关键词 | DROP/TRUNCATE/ALTER |
| 7 | SQL 无变更关键词 | DELETE/UPDATE（需确认意图） |
| 8 | 图模式通常需要 end/answer 节点 | 缺失可能导致流程无返回 |
| 9 | template-transform 禁止 dict 方法调用 | `.items()`、`.keys()`、`.values()` 在 Dify Jinja2 沙箱中报 TypeError，应在 code 节点预处理 |

## 使用方式

Claude Code 生成 DSL 后，按此 checklist 逐项检查：

1. 先检查 Error 级别（必须全部通过）
2. 再检查 Warning 级别（尽量通过，无法修复则提示用户）
3. 校验通过后交付 DSL；发现错误时自动修复后重新输出

## 常见错误→修复模式

校验脚本报错时，按以下模式修复：

| 错误信息 | 原因 | 修复方式 |
|---------|------|---------|
| `YAML parse failed` | YAML 语法错误（缩进、引号、特殊字符） | 检查缩进是否为 2 空格；字符串含 `:` `#` `{}` 时加引号 |
| `version must be a string` | version 未加引号 | 改为 `version: "0.6.0"` |
| `app.mode is missing or unsupported` | mode 值拼写错误或缺失 | 使用 `workflow` 或 `advanced-chat` |
| `workflow.graph.nodes is missing` | graph 模式缺少 nodes | 添加 `graph.nodes: []` 至少包含 start 和 end/answer |
| `Duplicate node id` | 节点 ID 重复 | 重新生成唯一 ID（时间戳格式 `17700000000XX`） |
| `Node missing data.type` | 节点缺少类型字段 | 在 `data` 中添加 `type` 字段（如 `llm`、`code`） |
| `Edge source does not exist` | 边引用了不存在的节点 | 检查 `source` 和 `target` 是否为已存在的节点 ID |
| `sourceType does not match node type` | 边的 sourceType 与节点类型不匹配 | 将边的 `sourceType` 改为源节点的 `data.type`；iteration-start 节点用 `iteration-start`，loop-start 用 `loop-start` |
| `missing prompt_template list` | LLM 节点缺少 prompt 模板 | 添加 `prompt_template` 列表，至少一条 user 消息 |
| `must define def main` | Python code 节点缺少 main 函数 | 在 code 中添加 `def main(...) -> dict:` |
| `missing outputs mapping` | Code 节点缺少 outputs | 添加 `outputs` 字典，key 与 main 返回值对应 |
| `missing provider_id / tool_name` | Tool 节点缺少必填字段 | 补齐 `provider_id`、`provider_name`、`provider_type`、`tool_name`、`tool_parameters` |
| `missing cases list` | if-else 节点缺少分支 | 添加 `cases` 列表，至少一个条件分支 |
| `Duplicate edge id` | 边 ID 重复 | 重新生成唯一边 ID（格式 `{source}-source-{target}-target`） |
| `SQL trailing comma` | INSERT 语句列名列表末尾有逗号 | 删除 `)` 前的多余逗号 |
| `Variable reference points to unknown node` | `{{#node_id.field#}}` 引用了不存在的节点 | 检查 node_id 是否正确；确认节点在引用点之前已定义 |
| `dependencies type unsupported` | 依赖类型拼写错误 | 使用 `marketplace`、`package` 或 `github` |
