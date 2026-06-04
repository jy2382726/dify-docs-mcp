# 变量引用语法规则

## 两种变量格式

### 工作流变量引用（含 #）
格式：`{{#node_id.field#}}`
用途：引用上游节点的输出值
示例：`{{#llm_1.text#}}`、`{{#start_1.query#}}`

### Prompt 占位符（不含 #）
格式：`{{variable_name}}`
用途：Agent Prompt 模板中的变量占位
示例：`{{sys.query}}`、`{{user_name}}`

## 变量前缀

| 前缀 | 说明 | 示例 |
|------|------|------|
| `sys` | 系统变量（16 个） | `sys.query`、`sys.files`、`sys.user_id` |
| `env` | 环境变量 | `env.API_KEY`、`env.DB_URL` |
| `conversation` | 对话变量 | `conversation.history` |
| `rag` | RAG 检索上下文 | `rag.top_k`、`rag.score_threshold` |

## value_selector 和 variable_selector

- `value_selector` 和 `variable_selector` 是数组格式：`["node_id", "field"]`
- 示例：`["llm_1", "text"]` 表示引用 llm_1 节点的 text 输出

## 系统变量完整列表（16 个）

| 变量名 | 类型 | 说明 |
|--------|------|------|
| `sys.query` | string | 用户查询文本 |
| `sys.files` | array[File] | 用户上传的文件 |
| `sys.user_id` | string | 用户 ID |
| `sys.conversation_id` | string | 会话 ID |
| `sys.dialogue_count` | number | 对话轮次 |
| `sys.chat_history` | string | 聊天历史 |
| `sys.current_timestamp` | number | 当前时间戳 |
| `sys.current_date` | string | 当前日期 |
| `sys.current_time` | string | 当前时间 |
| `sys.timezone` | string | 时区 |
| `sys.language` | string | 语言 |
| `sys.app_name` | string | 应用名称 |
| `sys.app_description` | string | 应用描述 |
| `sys.recommended_questions` | array[string] | 推荐问题 |
| `time_diff` | string | 时间差 |
| `upload_file` | File | 上传文件 |

## 常见错误

1. 混用 `{{#...#}}` 和 `{{...}}` — 前者用于节点输出引用，后者用于 Prompt 模板
2. 忘记 `#` 号 — `{{node_id.field}}` 不会解析为节点引用
3. 引用不存在的节点 — 会导致导入后运行时错误
