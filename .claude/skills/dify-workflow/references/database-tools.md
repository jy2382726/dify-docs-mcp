# 数据库工具节点

当工作流涉及数据库读写操作时，加载此文件获取工具节点模板和安全规范。

## 适用场景

- 用户需求涉及 PostgreSQL/SQL 数据库
- 需要构建 NL2SQL 检索模式
- 需要数据库写入（INSERT）或读取（SELECT）模板

---

## 两种常用插件模式

### `spance/db_client_node`

当 Dify 配置了命名 PostgreSQL 客户端工具（如 `postgres_node_5` 或 `postgres_node_10`）时使用。

- 插件依赖：`spance/db_client_node:0.1.47@...`
- Provider：`spance/db_client_node/db_client_node`
- 工具名称：`postgres_node_5`、`postgres_node_10`
- 动态值通过 `arg0`...`arg4` 或 `arg0`...`arg9` 传递
- SQL 占位符为 `$arg0`、`$arg1`、...
- 数据库连接由 Dify 工具授权/配置处理，不在 YAML 中

### `hjlarry/database`

用于通用 SQL 执行工具，SQL 来自前序节点。

- 插件依赖：`hjlarry/database:0.0.6@...`
- Provider：`hjlarry/database/database`
- 工具名称：`sql_execute`
- 参数：`query`，可选 `db_uri`
- 配置：`format`（`json`、`csv`、`yaml`、`md`、`xlsx`、`html`），可选 `config_options`
- 适合 LLM/parameter-extractor 选定查询后的 SELECT

## `spance/db_client_node` 插入模板

这是解析文档记录的正确写入模式。避免列名列表末尾的尾逗号。

```yaml
title: "把文档内容插入数据库"
type: tool
is_team_authorization: true
provider_id: spance/db_client_node/db_client_node
provider_name: spance/db_client_node/db_client_node
provider_type: builtin
plugin_id: spance/db_client_node
plugin_unique_identifier: spance/db_client_node:0.1.47@60965f21262129b75b2e01543190ab65021b75b55441532f77f88fcd7b71095d
tool_name: postgres_node_10
tool_label: "PostgreSQL 客户端(10)"
tool_description: "读写PostgreSQL客户端，最多10个参数。"
tool_node_version: "2"
tool_configurations: {}
tool_parameters:
  arg0:
    type: mixed
    value: "{{#sys.user_id#}}"
  arg1:
    type: mixed
    value: "{{#1777528600555.filename#}}"
  arg2:
    type: mixed
    value: "{{#1777528600555.size#}}"
  arg3:
    type: mixed
    value: "{{#1777528600555.extension#}}"
  arg4:
    type: mixed
    value: "{{#1777531848054.text#}}"
  arg5:
    type: mixed
    value: "{{#1777525141446.result#}}"
  arg6:
    type: mixed
    value: "{{#1777527355844.text#}}"
  arg7:
    type: mixed
    value: null
  arg8:
    type: mixed
    value: null
  arg9:
    type: mixed
    value: null
  query:
    type: mixed
    value: |
      INSERT INTO agent_test.ai_document_parse_record (
          user_id,
          file_name,
          file_size,
          file_extension,
          document_summary,
          md_table_content,
          json_table_content
      )
      VALUES (
          $arg0,
          $arg1,
          $arg2,
          $arg3,
          $arg4,
          $arg5,
          $arg6
      );
selected: false
```

写入新表时，将每个动态值映射到 `argN`，保持 SQL 静态。不要在 LLM prompt 中通过字符串拼接组装 SQL。

## `spance/db_client_node` 读取模板

```yaml
title: "读取摘要放到上下文里"
type: tool
is_team_authorization: true
provider_id: spance/db_client_node/db_client_node
provider_name: spance/db_client_node/db_client_node
provider_type: builtin
plugin_id: spance/db_client_node
plugin_unique_identifier: spance/db_client_node:0.1.47@60965f21262129b75b2e01543190ab65021b75b55441532f77f88fcd7b71095d
tool_name: postgres_node_5
tool_label: "PostgreSQL 客户端(5)"
tool_description: "读写PostgreSQL客户端，最多5个参数。"
tool_node_version: "2"
tool_configurations: {}
tool_parameters:
  arg0:
    type: mixed
    value: "{{#sys.user_id#}}"
  arg1:
    type: mixed
    value: null
  arg2:
    type: mixed
    value: null
  arg3:
    type: mixed
    value: null
  arg4:
    type: mixed
    value: null
  query:
    type: mixed
    value: |
      SELECT
          id,
          file_name,
          document_summary
      FROM agent_test.ai_document_parse_record
      WHERE user_id = $arg0
      ORDER BY created_at ASC;
selected: false
```

用此模式将候选文档/摘要加载到后续 LLM prompt 中。

## 去重检查模式

```yaml
tool_parameters:
  arg0:
    type: mixed
    value: "{{#sys.user_id#}}"
  arg1:
    type: mixed
    value: "{{#1777528600555.filename#}}"
  arg2:
    type: mixed
    value: null
  arg3:
    type: mixed
    value: null
  arg4:
    type: mixed
    value: null
  query:
    type: mixed
    value: |
      SELECT EXISTS (
          SELECT 1
          FROM agent_test.ai_document_parse_record
          WHERE user_id = $arg0
            AND file_name = $arg1
      ) AS file_exists;
```

后接 `if-else` 节点对工具输出做条件判断。在 Dify 中检查导入后的工具输出形状（`data`、`text` 或 `result`），因为插件版本可能不同。

## `hjlarry/database` SQL 执行模板

在 LLM 生成 SELECT 语句且 parameter-extractor 返回干净的 `SQL` 字符串后使用。

```yaml
title: "查询具体需要的数据"
type: tool
is_team_authorization: true
provider_id: hjlarry/database/database
provider_name: hjlarry/database/database
provider_type: builtin
plugin_id: hjlarry/database
plugin_unique_identifier: hjlarry/database:0.0.6@534bc26cf5bc4ff6b5557457452287ccc71f00eef9378784c4f43ca49954ca2f
tool_name: sql_execute
tool_label: "SQL Execute"
tool_description: "此工具用于在已存在的数据库中执行 SQL 查询。"
tool_node_version: "2"
tool_configurations:
  config_options:
    type: mixed
    value: null
  format:
    type: constant
    value: json
tool_parameters:
  db_uri:
    type: mixed
    value: null
  query:
    type: mixed
    value: "{{#1777540352910.SQL#}}"
selected: false
```

如果目标工作区没有数据库授权，通过环境变量设置 `db_uri`，而不是硬编码密钥：

```yaml
tool_parameters:
  db_uri:
    type: mixed
    value: "{{#env.DB_URI#}}"
  query:
    type: mixed
    value: "{{#1777540352910.SQL#}}"
```

## NL2SQL 检索模式

1. `spance/db_client_node` 读取当前用户的摘要和文件名
2. LLM 接收用户问题加摘要列表，输出 SQL 查询或空字符串
3. `parameter-extractor` 提取字段 `SQL`
4. `if-else` 检查 `SQL` 是否 `not empty`
5. `hjlarry/database` 以 `json` 格式执行 SQL
6. `template-transform` 可选格式化 JSON 结果
7. LLM 或 `answer` 回复用户

步骤 2 的 prompt 防护：

```text
仅生成 SQL。如果无法确定文档，输出空字符串。
规则：
- 仅 SELECT。
- 仅查询白名单字段：json_table_content。
- 表：agent_test.ai_document_parse_record。
- 始终包含 user_id = '{{#sys.user_id#}}' 或等效的参数绑定。
- 从提供的列表中精确匹配 file_name。不要编造文件名。
- 无 Markdown 围栏，无解释。
```

为增强安全性，不要让 LLM 直接将 `sys.user_id` 插入 SQL。而是让它选择文件名或 ID，然后使用带 `$arg0`/`$arg1` 的固定参数化 SQL 工具节点。

## SQL 安全检查清单

- 用户/文件/模型值使用 `$argN` 占位符
- `INSERT` 列表中 `)` 前不留尾逗号
- 公共模板避免 `DELETE`、`UPDATE`、`DROP`、`TRUNCATE` 和 `ALTER`，除非工作流明确是管理工作流
- 用 parameter-extractor 和 `if-else` 门控验证 LLM 生成的 SQL
- SELECT 查询限制在预期的 schema 和列内
- 不要在 DSL 中放数据库 URI 密码；使用 Dify 授权或 `env.DB_URI`
- 工具输出字段名因插件而异。导入后检查工具节点的实际输出并调整下游 selector
