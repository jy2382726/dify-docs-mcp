# iteration 与 loop 节点对比

Dify 工作流提供两种容器节点：iteration（迭代）和 loop（循环）。它们的本质差异决定了使用场景。

## 核心差异

| 维度 | iteration | loop |
|------|-----------|------|
| 数据来源 | 外部数组（iterator_selector） | 内部状态（loop_variables） |
| 每轮输入 | 通过 `item` 取当前元素 | 通过 `loop_variables` 读取状态 |
| 状态管理 | 无状态，每轮独立 | 有状态，跨轮次传递 |
| 退出条件 | 数组遍历完 | break_conditions 满足或达到 loop_count |
| 写回机制 | 不需要 | 必须用 assigner 节点写回 |

## iteration 适用场景

- 批量处理一组同类数据（如多个文档摘要、多维度分析）
- 每个元素的处理相互独立
- 输出是数组（如 `array[string]`）

**典型示例**：遍历用户上传的多个文件，逐个提取关键信息。

## loop 适用场景

- 需要根据上一轮结果决定下一轮行为
- 有明确的退出条件（如质量达标、收敛）
- 状态需要跨轮次维护

**典型示例**：迭代优化报告，每轮根据评分反馈改进，直到质量分 ≥8。

## 变量引用差异

### iteration
```text
当前元素：{{#iteration_1.item#}}
当前索引：{{#iteration_1.index#}}
``` {data-source-line="225"}

### loop
```text
循环变量：{{#loop_1.variable_label#}}  （不是 item！）
``` {data-source-line="230"}

## 常见陷阱

1. **loop 内忘记 assigner**：仅靠代码节点 return 不会更新 loop_variables，下一轮读到的是旧值
2. **loop 内写 `["loop_id", "item"]`**：这是 iteration 的语法，loop 没有 item
3. **iteration 内子节点有状态依赖**：每轮独立执行，前一轮的状态不会保留
4. **break_conditions 的 value 是变量**：必须加 `numberVarType: variable`，否则按常量处理