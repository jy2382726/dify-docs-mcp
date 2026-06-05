# 真实 YAML 文件研究

需要参考真实 Dify 工作流模式时加载此文件。基于 262 个公开 DSL 文件的分析结果。

## 适用场景

- 生成的工作流需要与真实世界对齐
- 需要了解实际导出的工具节点格式
- 需要参考触发器和集成工作流模式
- 需要了解公共 DSL 的版本分布

---

## 目录

- 检查的语料库
- 2026-05-11 附加语料库
- 详细样本
- 附加详细样本
- 观察到的版本和模式现实
- 工具节点现实
- 触发器和集成现实
- 业务工作流现实
- 依赖现实
- 画布和辅助节点
- 生成器项目经验
- 本技能的规则修正

## 检查的语料库

检查的仓库：

- `BannyLon/DifyAIA`：38 个 YAML 文件，全部 38 个解析为 Dify 应用
- `svcvit/Awesome-Dify-Workflow`：46 个 YAML 文件，45 个解析为 Dify 应用
- `wwwzhouhui/dify-for-dsl`：92 个 YAML 文件，89 个解析为 Dify 应用

详细样本：39 个代表性 DSL 文件，按最近提交日期和工作流多样性选择，重点侧重 `Awesome-Dify-Workflow`。

2026-05-07 版本扫描：

- `BannyLon/DifyAIA`：`0.1.2`-`0.3.1`，无 `0.6.0`
- `svcvit/Awesome-Dify-Workflow`：`0.1.0`-`0.3.0`，无 `0.6.0`
- `wwwzhouhui/dify-for-dsl`：`0.1.2`-`0.5.0`，无 `0.6.0`

结论：这些样本是宝贵的现实世界 DSL 证据，但仅作为遗留/导入形状和工作流设计素材。新生成应以 `references/official-0.6-target.md` 为目标。

## 2026-05-11 附加语料库

检查的仓库：

- `g-krishna0/dify-export-test`：`dsl/` 下 87 个 YAML 文件，全部 87 个解析为 Dify 应用。全部使用 `version: 0.3.1`；模式为 50 个 `workflow` 和 37 个 `advanced-chat`
- `Petrus-Han/dify-usecase-playground`：`usecases/` 下 3 个 YAML 文件，全部 3 个解析为 Dify 应用。两个使用 `version: 0.5.0`；一个使用 `version: 0.6.0`；全部为 `workflow`
- `TheOneWithChair/Dify-DSL-generator`：不是样本 DSL 语料库，但作为竞品生成设计有参考价值。它使用 Streamlit 应用和 Gemini API 包装生成过程，采用 UI 输入收集工作流类型、复杂度、所需工具、预期输入/输出、验证、细化和节点文档库。可借鉴的经验是更好的输入收集和参考组织

本次扫描后的聚合公共语料库：262 个解析的 Dify 应用 DSL 文件（172 个早期文件 + 90 个新文件）。大多数仍是遗留导出。新的 `dify-usecase-playground` 样本证明公共 `0.6.0` 导出存在，但样本量很小；官方 Dify 源码仍是新 DSL 的权威。

新语料库统计：

- `dify-export-test`：节点数 3 到 190，中位数 15。节点类型排名：`code` 744、`answer` 561、`if-else` 507、`assigner` 416、`template-transform` 259、`llm` 202、`http-request` 133、`tool` 90、`end` 100
- `dify-export-test` 工具 provider：`workflow` 69 和 `builtin` 21。这是导出的 Dify 工作流常被组合为可复用子工作流工具的有力证据
- `dify-usecase-playground`：节点数 3、8 和 14。节点类型包括 `trigger-schedule`、`trigger-webhook`、`trigger-plugin`、`agent`、`tool`、`llm`、`question-classifier`、`if-else` 和 `code`
- 新的依赖证据仍以 marketplace 为主：`dify-export-test` 有 72 个 marketplace dependency 条目；`dify-usecase-playground` 有 4 个

## 详细样本

### BannyLon/DifyAIA

- `文粹 AI——批量文档总结神器.yml`：advanced-chat, `0.3.1`, document extractor + iteration
- `票录精灵.yml`：advanced-chat, `0.3.1`, 飞书多维表格添加记录工具
- `架构魔法师.yml`：advanced-chat, `0.3.1`, document extractor + Mermaid 转换工具
- `智票通 - 批量发票智能解析 (1).yml`：advanced-chat, `0.3.1`, 发票解析、分支、飞书表格工具
- `PDF 翻译 Agent.yml`：agent-chat, `0.3.1`, `model_config.agent_mode` 带 MCP 工具，无 workflow graph
- `Zapier MCP test.yml`：advanced-chat, `0.3.0`, MCP tool 节点
- `实时热点新闻聚合引擎（每日简报版）.yml`：advanced-chat, `0.3.0`, 多个 RSS 工具、code 节点、SMTP 工具、聚合器
- `文生视频.yml`：advanced-chat, `0.3.0`, 文生视频工具
- `文思泉涌.yml`：workflow, `0.3.0`, iteration 和 code
- `智能合同卫士.yml`：workflow, `0.2.0`, 文档提取和 Markdown 导出工具
- `知识图解（KnowGraph）.yml`：workflow, `0.1.5`, 嵌套 iteration、注释、Jina 工具、多个 end

### svcvit/Awesome-Dify-Workflow

- `小支付-DEMO.yml`：advanced-chat, `0.3.0`, 支付工具、assigner、分支
- `Artifact.yml`：advanced-chat, `0.2.0`, 最简 LLM answer
- `MCP-amap.yml`：advanced-chat, `0.1.5`, 带 MCP server 参数的 agent 节点
- `图文知识库.yml`：advanced-chat, `0.1.5`, knowledge retrieval + LLM
- `Demo-tod_agent.yml`：advanced-chat, `0.1.5`, agent + 条件 answer
- `记忆测试.yml`：advanced-chat, `0.1.2`, 多个 assigner 和 conversation memory 模式
- `根据用户的意图进行回复.yml`：workflow, `0.1.0`, question classifier + knowledge retrieval + aggregator
- `文章仿写-单图_多图自动搭配.yml`：workflow, `0.1.0`, workflow provider 工具、parameter extractor、iteration
- `搜索大师.yml`：advanced-chat, `0.1.0`, HTTP、搜索工具、iteration
- `simple-kimi.yml`：advanced-chat, `0.1.2`, list-operator、document extractor、多个工具分支
- `json_translate.yml`：workflow, `0.1.3`, code + iteration + 翻译工具
- `runLLMCode.yml`：workflow, `0.1.4`, HTTP request + code 执行模式
- `Text to Card Iteration.yml`：workflow, `0.1.0`, parameter extractor + template
- `全书翻译.yml`：workflow, `0.1.2`, iteration 加画布注释
- `旅行Demo.yml`：advanced-chat, `0.1.5`, agent + assigner + template

### wwwzhouhui/dify-for-dsl

- `51-dify案例分享-...财报分析...HTML 可视化.yml`：advanced-chat, `0.5.0`, MinerU 解析文件 + LLM + code
- `88-dify案例分享-...Nano Banana2AI画图.yml`：advanced-chat, `0.4.0`, package dependency 和私有图像工具
- `85-dify案例分享-...Sora2...yml`：advanced-chat, `0.4.0`, package dependency 和视频工具
- `86-dify案例分享-Qwen3-VL+Dify...yml`：advanced-chat, `0.4.0`, HTTP + code + if-else 多模态流程
- `84-dify案例分享-...文生图+图生图插件...yml`：advanced-chat, `0.4.0`, 两个图像工具和分支
- `83-dify案例分享-...即梦 4.0 多图生成...yml`：advanced-chat, `0.4.0`, HTTP/code 多 answer 流程
- `79-dify案例分享-...MCP工具...yml`：advanced-chat, `0.3.0`, LLM + code + agent
- `76-dify案例分享-...通用票据识别...yml`：advanced-chat, `0.3.0`, 多个 HTTP/code/LLM 分支
- `74-dify案例分享-...秘塔搜索...yml`：workflow, `0.3.0`, 26 节点搜索工作流，多个 code/end 分支
- `73-dify案例分享-...发票申请预览...yml`：advanced-chat, `0.3.0`, Excel 工具 + LLM + code
- `69-dify案例分享-数学公式识别工作流.yml`：advanced-chat, `0.3.0`, PDF 处理工具 + aggregator
- `58-dify案例分享-中小学数学错题本-生成同类型题.yml`：advanced-chat, `0.3.0`, 数据库、时间、Markdown 导出、iteration、question classifier
- `57-dify案例分享-中小学数学错题本-错题收集篇.yml`：advanced-chat, `0.3.0`, 数据库 + PDF 处理 + iteration

## 附加详细样本

### g-krishna0/dify-export-test

- `Dify_Dsl_Github_Sync_Cron_OPTIMISED__v30.yml`：workflow, `0.3.1`, 定时或手动运行的 DSL 同步模式。使用 code 密集的状态/diff/导出逻辑和 iteration 辅助。经验：当逻辑是 API 分页、状态文件、GitHub 提交和标准化 diff 时，少量健壮的 code 节点比大量小节点的庞大图更清晰
- `Auto_KB_Metadata_Extractor_v4.1.yml`：workflow, `0.3.1`, Dify 知识库元数据更新。使用 HTTP request、code 解析、if-else 验证、LLM 提取、iteration 和多个错误 end 路径。经验：外部 Dify API 自动化需要显式失败分支处理缺失 ID、空段、API 错误和部分成功
- `Graph_Data_Extractor.yml`：workflow, `0.3.1`, 文件上传图表数据提取。使用多次 LLM 处理加 code/template。经验：视觉或文档提取工作流受益于将检测、提取、反幻觉验证和最终表格格式化分离
- `Table_Data_Extractor__PDF_v6.yml`：workflow, `0.3.1`, PDF 表格提取。使用 code 密集的 pdfplumber 风格提取、LLM 清洗、iteration 和 template 输出。经验：大文档在 LLM 清洗前做确定性提取可节省 token 并减少幻觉
- `Travel_Expenses_Form3_Validator.yml`：workflow, `0.3.1`, 表单验证和缺陷邮件生成。使用 document extraction、knowledge retrieval、variable aggregation、template transform、code 和 end。经验：验证工作流应用 code 做规则检查，用 template 做稳定消息
- `SharePoint_Connector_Graph_API.yml`：workflow, `0.3.1`, 无 Dify 插件的 Microsoft Graph 连接器。使用 HTTP request、code、if-else 和 variable aggregator。经验：无插件时 REST + code 可模拟连接器，但密钥必须是环境变量/占位符
- `D11QAEmail.yml`：advanced-chat, `0.3.1`, PC 检查支持机器人，调用可复用工作流做双语处理、文件验证、Excel 提取、FAQ 查找和邮件生成。经验：当用户在支持对话中且可复用工作流工具承担主要工作时，Chatflow 是合适的选择
- `The_Smart_Voyager.yml`：advanced-chat, `0.3.1`, 旅行规划器，将经验、签证、物流和行程步骤委托给 workflow-provider 工具。经验：复杂助手可以是 Chatflow 外壳加较小的 Workflow 工具
- 重复的 Excel 提取器文件：workflow, `0.3.1`, 文件上传加 `document-extractor`、`list-operator`、`excel_2_csv` 工具、code 清洗、LLM 和多个 end 分支。经验：文件转换和确定性清洗应在让 LLM 推断字段之前完成

此仓库常见工具名称：

- Workflow-provider 工具：`comBilingualWorkflow` 19、`excel_extractor_v4` 17、`comFileValidator` 16、`email_gen_v1` 6、`PC_inspection` 4
- 内置工具：`md_to_pdf` 11、`excel_2_csv` 7、`word_2_pdf` 2、`zip` 1

### Petrus-Han/dify-usecase-playground

- `daily-news-slack/workflow.yml`：workflow, `0.5.0`, schedule trigger → agent 带 Yahoo/current-time 工具 → Slack webhook。经验：定时摘要用 Workflow 而非 Chatflow；输出可以是副作用工具调用
- `slack-news-researcher/workflow.yml`：workflow, `0.5.0`, plugin trigger → question classifier → agent/LLM 分支 → Slack webhook。经验：消息事件工作流可分类是否使用研究工具或生成普通回复后发回 Slack
- `confluence-to-feishu/workflow.yml`：workflow, `0.6.0`, webhook trigger → code payload 解析 → if-else 事件路由 → 飞书消息工具。经验：webhook 自动化应先在 code 中标准化 payload，再按事件类型分支并构建确定性消息卡片

## 观察到的版本和模式现实

- 公共 DSL 常用 `0.1.0` 到 `0.5.0`；不是每个可运行的工作流都使用最新官方 DSL 版本
- 前三个检查的仓库没有解析到 `0.6.0` 导出。后续 2026-05-11 扫描在 `Petrus-Han/dify-usecase-playground` 中发现了一个公共 `0.6.0` 工作流，所以公共 `0.6.0` 证据存在但仍稀少
- `advanced-chat` 在近期示例中占主导；`workflow` 在批量运行和多端点自动化中仍常见
- 新样本中的触发式集成自动化使用 `workflow`，即使它们响应到 Slack/飞书而非返回普通 `end` 输出
- `agent-chat` 示例常使用顶层 `model_config`，而非 `workflow.graph.nodes`
- 部分 YAML 文件写 `version: 0.3.0` 不带引号。YAML 解析器通常将多点版本保持为字符串，但生成的 DSL 仍应引号包裹版本以确保安全

## 工具节点现实

真实工具节点比干净 schema 变化更大：

- `provider_type` 出现为 `builtin`、`api`、`workflow` 和 `mcp`
- 许多有效工具节点在节点内不包含 `plugin_id` 或 `plugin_unique_identifier`，即使应用有顶层 dependency
- `tool_node_version` 有用但不总是存在
- `paramSchemas`、`params` 和 `is_team_authorization` 在较新导出中频繁出现，从真实 DSL 复制时应保留
- MCP 工具可作为普通 workflow tool 节点出现，也可在 `model_config.agent_mode.tools` 内出现
- Workflow-provider 工具可引用本地/自定义工作流，provider ID 为 UUID 形式且无 marketplace dependency
- 在 `dify-export-test` 中，workflow-provider 工具占工具节点的多数。将它们视为强大但工作区特定的；永远不要编造 provider UUID

## 触发器和集成现实

- 真实触发器工作流可能以 `trigger-schedule`、`trigger-webhook` 或 `trigger-plugin` 而非 `start` 节点开始
- 公共触发器样本可能省略 `end` 节点，当任务目的是副作用（如 Slack/飞书通知）时。对于调用者需要返回值的用户运行工作流，仍需包含 `end`
- 定时导出包含 `cron_expression` 和 visual config/timezone 字段；导入后可能需要重新配置
- Webhook 导出保持 `webhook_url` 和 `webhook_debug_url` 为空。不要用假 URL 填充
- 插件触发器 schema 是插件特定的。从当前导出复制，而不是外推

## 业务工作流现实

- 文件和办公文档工作流常组合文件上传、文档提取/转换工具、code 清洗、LLM 解释、if-else 错误分支和模板化输出
- 企业审批、检查、差旅报销和 QA 工作流在转换或验证记录时通常是 `workflow`，但人在继续追问时是 `advanced-chat`
- 集成工作流常需要密钥和端点 ID 作为环境变量。公共示例有时嵌入占位符或静态描述；生成的 DSL 不应硬编码真实 token
- 复杂助手产品可设计为 Chatflow 外壳，将确定性子任务委托给暴露为 workflow-provider 工具的较小 Workflow 应用
- 有状态同步任务中，code 节点可合理承担大部分逻辑，因为 Dify 图节点在 API 分页、diff、打包和提交状态方面不太符合人体工程学

## 生成器项目经验

`TheOneWithChair/Dify-DSL-generator` 目标类似但用 Streamlit 应用和 Gemini API 包装生成。可借鉴的思路：

- 尽早询问或推断工作流类型：Chatflow 映射到 Dify `advanced-chat`；普通自动化映射到 `workflow`
- 写 YAML 前收集预期输入、预期输出、所需工具和复杂度
- 节点文档放独立文件，仅加载相关节点参考
- 验证和后处理生成的 YAML，而不是信任原始模型输出
- 支持从自然语言反馈细化现有 DSL

本技能的差异：

- 不需要 `GEMINI_API_KEY`、`DIFY_API_KEY` 或 Streamlit UI。使用此技能的 Agent 就是生成器
- 新 DSL 以官方 `version: "0.6.0"` 为目标，旧样本仅作为兼容性/工作流设计证据
- 优先使用导出的插件/工具节点确保可靠性，而不是仅从名称合成精确的插件 schema

## 依赖现实

依赖不仅仅是 marketplace 条目。

依赖条目格式（marketplace / package / github）的完整 YAML 示例见 `dsl-structure.md` 的 Dependencies 章节。

旧版或自定义工作流可能在使用内置、MCP、API 或 workflow 工具时仍有 `dependencies: []`。

## 画布和辅助节点

- `custom-note` 节点在复杂公共 DSL 中常见。它们可能有空的 `data.type`，应视为画布注释而非可执行节点
- Iteration 导出包含 `iteration` 加 `iteration-start`（wrapper `custom-iteration-start`）
- 部分旧 iteration 子边缺少所有较新的 loop/iteration 标记；从真实导出复制比手写嵌套图元数据更安全

### Iteration 节点完整结构（基于真实导出）

迭代节点是 Dify DSL 中最复杂的结构之一。以下是基于真实导出（搜索大师3.yml、json_translate.yml）的完整字段清单：

**迭代容器节点**：
- `data.type: iteration` — 必须
- `data.iterator_selector: ["源节点ID", "变量名"]` — 必须
- `data.output_selector: ["输出节点ID", "输出字段"]` — 必须
- `data.output_type: array[string]` — 必须
- `data.start_node_id: "iteration-start节点ID"` — 必须
- `data.startNodeType: agent/tool/code/llm` — 必须，指定迭代内起始节点类型
- `data.error_handle_mode: terminated | remove-abnormally` — 必须
- `data.width: 数字` — 必须，需要足够大以容纳内部子节点（建议 >= 500）

**iteration-start 辅助节点**：
- `data.type: iteration-start` — 注意：不是 `start`
- `data.isInIteration: true` — 必须
- `draggable: false` — 必须
- `selectable: false` — 必须
- `parentId: "迭代容器ID"` — 必须
- `position: { x: 24, y: 68 }` — 相对坐标（相对于容器左上角）

**迭代内子节点**：
- `data.isInIteration: true` — 必须
- `data.iteration_id: "迭代容器ID"` — 必须
- `data.isIterationStart: true` — 第一个节点必须有
- `parentId: "迭代容器ID"` — 必须
- `extent: parent` — 必须，坐标相对于父节点
- `position: { x: 数字, y: 数字 }` — 相对坐标
- `zIndex: 1001` — 第一个节点

**迭代内边**：
- `data.isInIteration: true` — 必须
- `data.iteration_id: "迭代容器ID"` — 必须
- `data.sourceType: iteration-start` — 从 iteration-start 出发的边
- `zIndex: 1002` — 迭代内边的 zIndex

**重要说明**：
1. **迭代内不需要显式的 iteration-end 节点**：迭代内的最后一个节点就是终点
2. **迭代项引用方式**：用 `["迭代容器ID", "item"]`，例如 `{{#iteration_1.item#}}`
3. **不要使用 `{{#sys.current_item#}}`**：这是错误的引用方式

## 本技能的规则修正

- 不要求每个工具节点都有 `plugin_id` 或 `tool_node_version`
- 可执行 workflow tool 节点确实要求 `provider_id`、`provider_name`、`provider_type`、`tool_name`、`tool_label` 和 `tool_parameters`
- 从真实节点复制时保留 `paramSchemas` 和 `params`
- 支持 dependency `type: package` 和 `type: github` 以及 `type: marketplace`
- 将 `custom-note` 视为有效的非可执行元数据
- 将 `agent-chat`、`chat` 和 `completion` 视为 model-config 应用，而非图工作流，除非存在图
- 生成的应用默认 `workflow`；需要 Chatflow、memory、`sys.query`、`sys.files` 和 `answer` 节点时切换为 `advanced-chat`
- 将触发器驱动的 Slack/飞书/webhook/定时自动化视为 `workflow`
- 新工作优先生成 `workflow` 或 `advanced-chat`，但理解遗留/公共 DSL 用于审查或适配
- 新生成 DSL 使用官方 `version: 0.6.0`（不带引号），仅从这些旧公共样本借鉴图模式
