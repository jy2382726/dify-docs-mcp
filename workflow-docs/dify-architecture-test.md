# Dify 工作流引擎架构概述

## 核心组件

Dify 工作流引擎基于 ReactFlow 实现 DAG 编排，主要包含以下模块：

1. **节点系统**：支持 30+ 种内置节点类型，包括 LLM、Code、HTTP、Tool、Iteration 等
2. **变量系统**：通过 `{{#node_id.field#}}` 语法实现节点间数据传递
3. **执行引擎**：负责拓扑排序、并行调度、错误处理
4. **状态管理**：通过 conversation_variables 和 environment_variables 维护状态

## 关键特性

- 支持迭代（iteration）和循环（loop）两种容器节点
- 提供 if-else、question-classifier 等多种分支机制
- 集成主流模型供应商和插件生态