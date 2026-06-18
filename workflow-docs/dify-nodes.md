# Dify 工作流节点详解

Dify 工作流引擎提供 30+ 种内置节点类型，覆盖从输入解析到 LLM 推理、外部调用、容器迭代等场景。本文详细介绍核心节点的功能与使用方式。

## 基础流程节点

### start 节点
工作流的入口节点，定义用户输入变量。支持多种变量类型：
- text-input / paragraph：文本输入
- number：数值
- select：下拉选择
- file / file-list：文件上传
- json：结构化数据

### end 节点
workflow 模式的出口，定义输出变量。每个输出变量通过 `value_selector` 引用上游节点的字段。

### answer 节点
chatflow 模式（advanced-chat）的出口，直接生成对话回复。

## 推理与转换节点

### llm 节点
大语言模型调用节点，支持 system/user 角色提示模板，可通过 `context` 字段注入知识库检索结果。支持 chat/completion 两种模式。

### code 节点
Python3 代码执行节点，必须定义 `main` 函数，返回 dict。常用于 JSON 解析、数据转换、字符串处理。

### template-transform 节点
Jinja2 模板渲染节点，运行在沙箱中。注意不能调用 dict 的 `.items()` 等方法，需要先用 code 节点转换为固定字段结构。

## 分支与提取节点

### if-else 节点
条件分支节点，Dify 限制为最多两个分支（true/false）。需要超过两个分支时，需串联多个 if-else。

### question-classifier 节点
基于 LLM 的语义分类节点，支持多个类别，分支 handle 即类别 ID。

### parameter-extractor 节点
结构化参数提取节点，根据定义的参数 schema 从文本中提取字段值。

## 容器节点

### iteration 节点
数组遍历容器，每轮通过 `["iteration_id", "item"]` 取当前元素，子节点无状态。

### loop 节点
条件循环容器，通过 `loop_variables` 维护状态，需要 `assigner` 节点写回更新值。支持 `break_conditions` 提前退出。