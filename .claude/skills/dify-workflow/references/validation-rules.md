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
| 13 | Tool 节点必填字段 | provider_id, provider_name, provider_type, tool_name, tool_parameters |
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

## 使用方式

Claude Code 生成 DSL 后，按此 checklist 逐项检查：

1. 先检查 Error 级别（必须全部通过）
2. 再检查 Warning 级别（尽量通过，无法修复则提示用户）
3. 校验通过后交付 DSL；发现错误时自动修复后重新输出
