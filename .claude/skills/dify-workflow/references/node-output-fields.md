# 节点输出字段参考

用于正确生成变量引用 `{{#node_id.field#}}` 中的 field 部分。

## start
无输出字段（变量通过 start 节点的 variables 定义）

## llm
| 字段 | 类型 | 说明 |
|------|------|------|
| `text` | string | LLM 生成的文本 |
| `usage` | object | Token 用量（prompt_tokens, completion_tokens, total_tokens） |
| `finish_reason` | string | 结束原因（stop, length 等） |

## code
| 字段 | 类型 | 说明 |
|------|------|------|
| `result` | any | 代码执行结果（对应 outputs 中定义的字段） |

## if-else
无输出字段（通过 sourceHandle 分流）

## knowledge-retrieval
| 字段 | 类型 | 说明 |
|------|------|------|
| `result` | array[object] | 检索结果列表 |
| `result[].content` | string | 文档片段内容 |
| `result[].title` | string | 文档标题 |
| `result[].score` | number | 相似度分数 |

## tool
| 字段 | 类型 | 说明 |
|------|------|------|
| `text` | string | 工具返回的文本结果 |
| `files` | array[File] | 工具返回的文件 |
| `json` | object | 工具返回的 JSON 结果 |

## end
无输出字段（定义 outputs 数组指定返回值）

## answer
无输出字段（answer 字段定义回复内容）

## iteration
| 字段 | 类型 | 说明 |
|------|------|------|
| `output` | array | 迭代输出列表 |

## loop
| 字段 | 类型 | 说明 |
|------|------|------|
| `output` | array | 循环输出列表 |

## parameter-extractor
| 字段 | 类型 | 说明 |
|------|------|------|
| `extracted_parameters` | object | 提取的参数值 |

## http-request
| 字段 | 类型 | 说明 |
|------|------|------|
| `response` | object | HTTP 响应体 |
| `status_code` | number | HTTP 状态码 |
| `headers` | object | 响应头 |

## variable-aggregator
| 字段 | 类型 | 说明 |
|------|------|------|
| `output` | any | 聚合后的变量值 |

## document-extractor
| 字段 | 类型 | 说明 |
|------|------|------|
| `text` | string | 提取的文本内容 |

## agent
| 字段 | 类型 | 说明 |
|------|------|------|
| `text` | string | Agent 生成的文本 |
| `usage` | object | Token 用量 |
| `finish_reason` | string | 结束原因 |

## template-transform
| 字段 | 类型 | 说明 |
|------|------|------|
| `output` | string | 模板渲染后的文本 |

## question-classifier
无输出字段（通过 class ID 分流）

## list-operator
| 字段 | 类型 | 说明 |
|------|------|------|
| `result` | array | 列表操作结果 |
| `first_record` | any | 第一条记录 |

## assigner
无输出字段（写入 conversation/environment 变量）
