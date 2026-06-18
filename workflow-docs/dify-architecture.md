# Dify 架构与设计原理

## 整体架构

Dify 是一个开源的 LLM 应用开发平台，核心架构分为以下层次：

1. **应用层**：提供 Chatflow、Workflow、Agent 等多种应用模式
2. **编排层**：基于 ReactFlow 的 DAG 图编辑器，支持节点拖拽和连线
3. **执行引擎**：负责拓扑排序、并行调度、错误处理、状态管理
4. **集成层**：对接主流模型供应商（OpenAI、Anthropic、通义千问等）和插件生态
5. **存储层**：使用 PostgreSQL 存储应用配置，Redis 缓存运行时状态，Weaviate/Qdrant 做向量检索

## 工作流执行引擎

### 执行流程
1. 从 start 节点开始，按边的拓扑顺序执行
2. 遇到 if-else/question-classifier 时根据条件路由
3. 遇到 iteration 时按数组长度循环内部子图
4. 遇到 loop 时按条件循环，通过 assigner 更新状态
5. 到达 end 节点时输出最终结果

### 并行调度
iteration 节点支持 `is_parallel: true` 并行执行，通过 `parallel_nums` 控制并发数（默认 10）。

### 错误处理
容器节点支持 `error_handle_mode`：
- `terminated`：出错时终止整个工作流
- `remove-abnormally`：跳过异常项继续执行

## 变量系统

### 变量引用语法
- `{{#node_id.field#}}`：引用上游节点输出
- `{{#sys.query#}}`：系统变量（用户查询、文件、用户 ID 等）
- `{{#env.API_KEY#}}`：环境变量
- `{{#conversation.Memory#}}`：对话变量（chatflow）

### 数据类型
支持 string、number、boolean、object、array[string]、array[number]、array[object]、array[file]、file 等。

## 插件生态

Dify 通过插件机制扩展能力：
- 模型供应商插件（如 langgenius/tongyi）
- 工具插件（如 langgenius/tavily 提供搜索能力）
- 数据源插件（如 Notion、GitHub 集成）
- MCP 协议支持

插件通过 marketplace、package、github 三种方式分发。