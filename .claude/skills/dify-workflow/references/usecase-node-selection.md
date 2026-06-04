# 用例与节点选择

收到用户的业务需求时，加载此文件确定工作流模式、触发方式和节点组合。

## 适用场景

- 用户描述业务需求，需要确定工作流模式（workflow vs advanced-chat）
- 需要选择触发方式（手动、聊天、定时、Webhook、插件事件）
- 需要参考常见业务模式和节点选择启发式

---

## 模式选择

| 用户需求 | 推荐模式 | 原因 |
| --- | --- | --- |
| 运行一次，处理输入，返回结构化结果 | `workflow` | 有明确的 start 变量和 end 输出 |
| 批量处理文件/表格/文档 | `workflow` | iteration、文档提取、code 和多 end 分支更易控制 |
| 定时任务、Webhook、插件事件或集成自动化 | `workflow` | 触发器节点是 workflow 原生的；终端动作可能是工具副作用 |
| 被其他 Dify 工作流调用的子流程 | `workflow` | 导出示例中常用 workflow 作为工具提供者 |
| 用户多轮对话 | `advanced-chat` | 需要 memory 和 answer 节点 |
| 聊天输入、上传文件、即时回复 | `advanced-chat` | 使用 `sys.query`、`sys.files` 和可选的 conversation 变量 |
| FAQ/客服机器人，带检索和多轮追问 | `advanced-chat` | 检索 + LLM + answer + memory |

### 模式决策规则

如果用户未指定模式，说明："我将默认使用 `workflow` 模式构建；如果您需要 Chatflow 行为，我会切换为 `advanced-chat`。" 然后继续，除非模式歧义会影响导入关键的输入或终端节点。

- **workflow**：用于一次性运行、批量处理、触发器、集成和副作用自动化
- **advanced-chat**：用于 Chatflow、`sys.query`、`sys.files`、memory 和 `answer` 节点

## 触发方式选择

| 触发需求 | 节点模式 | 说明 |
| --- | --- | --- |
| 用户手动运行带表单字段的应用 | `start -> ... -> end` | 在 start 变量中定义 `required`、`type` 和 label |
| 聊天消息启动流程 | `advanced-chat` 中 `start -> ... -> answer` | 通常使用 `{{#sys.query#}}` 而非 start 变量 |
| 定时摘要或同步 | `trigger-schedule -> agent/code/tool` | 公共示例中，当任务仅为副作用时，通常以 Slack/飞书/邮件工具结束 |
| 外部系统事件 | `trigger-webhook -> code -> if-else -> tool` | 在 code 中解析 payload；导出的 webhook URL 保持为空 |
| 插件事件 | `trigger-plugin -> classifier/agent/tool` | 保留导出的插件触发器字段；不要编造事件 schema |
| 其他工作流调用此流程 | `provider_type: workflow` 的 Tool 节点 | Provider ID 是工作区特定的；从导出中复制 |

## 常见业务模式

| 需求 | 推荐图结构 | 考虑使用的节点 |
| --- | --- | --- |
| 总结或改写文本 | `start -> llm -> end` | `llm`、`template-transform` |
| 生成邮件或报告草稿 | `start -> llm -> template-transform -> end` | LLM 生成内容，template 固定输出格式 |
| 分类意图然后路由 | `start -> question-classifier/if-else -> branch -> answer/end` | `question-classifier` 做语义分类，`if-else` 做确定性字段判断 |
| 从非结构化文本提取严格字段 | `start -> llm -> parameter-extractor/code -> end` | 模型提取够用时用 `parameter-extractor`；JSON 清洗用 `code` |
| 验证表单或业务规则 | `document-extractor/code -> if-else -> end/email` | Code 最适合确定性验证；分支用于缺陷提示 |
| 处理上传的 PDF/Excel/ZIP | `start(file) -> document-extractor/list-operator/tool -> code -> end` | 原生提取器不够时使用文件转换或提取工具 |
| 遍历行、文件或块 | `start -> code/list-operator -> iteration -> template-transform -> end` | 嵌套分支涉及的迭代内部结构从导出中复制 |
| 用当前数据搜索/研究 | `start/trigger -> agent -> tool -> end/tool` | 工具选择动态时用 Agent；始终显式建模时效性检查 |
| RAG 知识库问答 | `start -> knowledge-retrieval -> llm -> answer/end` | 启用 LLM `context` 并设置检索 top-k/阈值 |
| 更新知识库元数据或外部记录 | `start/trigger -> http-request/tool -> code -> if-else -> end` | 数据集/文档 ID 和 token 作为变量或占位符 |
| 发送 Slack/飞书/邮件通知 | `trigger/code/llm -> tool` | 输出可能仅为副作用；保留工具 `paramSchemas` 和授权字段 |
| 同步 Dify DSL 到 GitHub | `trigger-schedule/start -> code -> iteration/code -> end` | 有状态 API/文件逻辑用一个 code 节点比多个图节点更安全 |
| 审计或漏洞检查 DSL | `start(file/text) -> code/http -> llm -> end` | 先解析 YAML，再让 LLM 对标准化事实推理 |
| 从 DSL 构建流程图 | `start -> code/http -> llm/template -> end` | 图表生成前优先做结构化解析 |

## 节点选择启发式

- **code**：确定性解析、标准化、验证、去重、批处理和 API 响应整形
- **llm**：自然语言推理、总结、改写、容忍模型判断的提取和最终文案
- **parameter-extractor**：下游分支/工具需要从模型输出中提取少量类型化字段时
- **question-classifier**：语义意图类别驱动路由时
- **if-else**：条件明确时——空/非空、等于、包含、数字比较、验证结果或文件数量
- **template-transform**：数据已就绪后，用于稳定的 Markdown、类 JSON 文本、邮件正文或报告组装
- **variable-aggregator**：合并互斥分支输出到共享的 `end` 或 `answer` 前
- **assigner**：仅用于 conversation/environment 变量写入，主要在 Chatflow memory/状态场景
- **agent**：工具选择或多步研究动态时。固定工具调用用普通 tool 节点更可预测
- **http-request**：无 Dify 插件时用于普通 REST API；有插件提供授权、schema 和稳定输出时用 tool 节点

## 文件和文档工作流

- `workflow` 中单文件输入：start 变量定义 `type: file`
- 多文件：用 `type: file-list`，然后 `list-operator` 或 `iteration`
- 除非复制的导出另有说明，否则保持 `features.file_upload.enabled: false`
- 文档密集型工作流通常需要防护分支：缺失文件、转换失败、提取文本为空和内容超限
- Excel 提取：公共示例通常组合文件转换工具、`document-extractor`、code 清洗，仅在表格文本稳定后才用 LLM

## 集成工作流

- Slack、飞书、邮件等通知工具：保留导出的 `paramSchemas`、`params`、`tool_configurations`、`is_team_authorization`、`plugin_unique_identifier` 和 dependency 条目
- Webhook 工作流：先用 `code` 解析原始 payload，再做分支
- 定时工作流：包含时区和 cron/visual 配置，但预期目标工作区在导入后重新配置调度
- Workflow-provider 工具：provider ID 通常是工作区 UUID。除非用户提供目标工作区的导出，否则视为不可移植

## 可靠性规则

- 罕见插件、触发器、Agent 策略或 workflow-provider 工具：以导出节点为事实来源
- 逻辑主要是状态管理、API 分页、diff 或文件打包时：少量健壮 code 节点优于庞大图
- 业务工作流添加显式失败输出：无效输入、无数据、上游 API 失败、未授权或部分成功
- 面向用户的生成文件或消息：将推理/提取与格式化分离——先 LLM 或 code，再 `template-transform` 或 tool
