# WIKI 质量对比报告

## 基础元数据

> 注：因本次仅对比目标章节（第 10、14、19 章），Wiki A 目录中仅有这 3 章；Wiki B 为完整 37 章 wiki。下表"目标章节"统计基于双方这 3 章的对比数据。

| 指标 | 文档A | 文档B |
|---|---|---|
| 总章数 | 3（目标章节） | 37 |
| 目标章节总大小 | 76 KB | 48 KB |
| 目标章节平均大小 | 25.3 KB / 281 行 | 16.0 KB / 272 行 |
| 目标章节总 mermaid 数 | 16 | 7 |
| 目标章节平均引用/章 | 41.3 | 12.0 |
| 模型（wiki.json） | claude-opus-4-7 | 未标注 |
| 生成日期 | 2026-05-15 | 2026-04-03 |

---

## 维度评分

| 维度 | 文档A | 文档B | 差异说明 |
|---|---|---|---|
| **1. 覆盖度** | 8 / 10 | 8.5 / 10 | B 在 14、19 章覆盖了安全配置/错误处理/管理 API/未来演进等 A 未涉及的主题；A 在核心机制上没有遗漏 |
| **2. 深度** | 9 / 10 | 5 / 10 | A 每章均解释了 WHY（设计决策的根因），如"为什么顺序写死""为什么注入到用户消息而非 system prompt"；B 以 WHAT 罗列为主，缺少设计 rationale |
| **3. 准确性** | 9 / 10 | 5 / 10 | A 5 抽 5 通过；B 5 抽中 2 处准确、1 处行号偏移约 60 行、1 处偏移约 300 行、1 处范围过大（1-373 行）难以核验 |
| **4. 结构清晰** | 9 / 10 | 7 / 10 | A 采用统一的 Overview/Architecture/Components/Implementation/Pitfalls/References 模板；B 各章结构不一致，部分内容像 API 手册拼接 |
| **5. 可读性** | 9 / 10 | 7 / 10 | A 有 TL;DR 和"为什么"引导，新手友好；B 配置参数罗列较多，对新手有门槛 |
| **6. 图表质量** | 9 / 10 | 5 / 10 | A 3 章共 16 个 mermaid，涵盖 graph/classDiagram/stateDiagram/sequenceDiagram/flowchart/erDiagram 6 种类型；B 仅 7 个，以 flowchart 为主，信息密度低 |
| **7. 引用质量** | 9 / 10 | 4 / 10 | A 引用精确到具体函数/代码段，行号范围通常 5-30 行；B 引用范围大（常见 50-300 行），且部分引用未精准对应描述内容 |
| **8. 互引网络** | 8 / 10 | 7 / 10 | A 每章末尾有 Related Pages 双向链接表；B 有相关文档导航，但 10 章与 19 章的互引格式不统一 |
| **总分** | **70 / 80** | **48.5 / 80** | |

---

## 引用准确性抽查

### 文档A 抽样

| # | 引用 | 验证结果 |
|---|---|---|
| 1 | `factory.py:155-188` — feature 驱动的 14 段固定链 | 155 行起为 `_assemble_from_features` 函数，docstring 中明确列出 0-13 号位顺序，与描述完全一致 |
| 2 | `tool_error_handling_middleware.py:39-67` — 异常转 error ToolMessage | 39 行起为 `wrap_tool_call`，try/except GraphBubbleUp/except Exception 结构与 A 引用的代码片段完全吻合 |
| 3 | `sandbox.py:6-93` — Sandbox 抽象接口 | 6 行起为 `class Sandbox(ABC)`，含 execute_command/read_file/list_dir/write_file/glob/grep/update_file 共 7 个抽象方法，与描述一致 |
| 4 | `memory_middleware.py:52-110` — after_agent 触发记忆写入 | 52 行起为 `after_agent` 方法，含过滤消息、检测 correction/reinforcement、门槛检查、排队四步，与描述一致 |
| 5 | `storage.py:160-189` — 原子写入 memory.json | 160 行起为 `save` 方法，使用 `.tmp` 临时文件 + `temp_path.replace(file_path)` 原子替换，与描述一致 |

**文档A 准确率：5/5** 

### 文档B 抽样

| # | 引用 | 验证结果 |
|---|---|---|
| 1 | `factory.py:1-373` — 工厂方法与中间件链 | 1-11 行 docstring 确认是 SDK 层入口，但 1-373 范围过大，无法核验整段描述是否精确对应 |
| 2 | `loop_detection_middleware.py:1-228` — 滑动窗口哈希检测 | 34-40 行确认默认配置（warn=3/hard=5/window=20/max_threads=100），与描述一致 |
| 3 | `local_sandbox.py:44-92` — 最长前缀优先路径解析 | 44-92 行主要为 shell 检测与路径映射初始化；"最长前缀优先"的 `_find_path_mapping` 实际在 107-121 行，行号偏移约 60 行 |
| 4 | `aio_sandbox_provider.py:155-200` — 温池机制 | 155-200 行为配置加载与孤儿容器清理；温池核心逻辑（`_acquire_internal` 中的 warm pool reclaim）在 466-476 行，偏移约 300 行，描述与引用严重错位 |
| 5 | `prompt.py:182-341` — 精确词元预算控制 | 182 行起跨越 `_coerce_confidence` 与 `format_memory_for_injection`，确实包含 tiktoken 计量与预算截断逻辑，与描述一致 |

**文档B 准确率：2/5 精确命中，1 处大范围，1 处偏移约 60 行，1 处偏移约 300 行**

---

## 抽样章节深度对比

### 章节 10：中间件链机制

**文档A 优势**：
- 从"为什么要有中间件链"出发，解释了主循环爆炸、无法选择性开关、无法扩展三个致命问题，再引出 middleware 方案
- 深入剖析了"洋葱模型"的执行语义，用 `wrap_tool_call` 的代码片段演示了三种用法（改 request、短路、异常兜底）
- 详细解释了三套装配机制（feature 固定链、配置生产链、@Next/@Prev 锚点插入），并给出锚点插入算法的流程图
- 6 个 mermaid 图表类型多样（graph/classDiagram/stateDiagram/sequenceDiagram/flowchart）

**文档B 优势**：
- 给出了 14 个中间件的完整矩阵表（before_agent/before_model/after_model/after_agent/wrap_tool_call 各列标注）
- 包含中间件开发指南（最小实现模板、异步支持、测试模式），对二次开发者更实用
- 状态管理一节解释了 `ThreadState` 如何合并各中间件状态

**裁决**：文档A 胜。A 在 WHY 和 HOW 的深度上明显领先，对洋葱模型和装配算法的解释是 B 缺失的；B 的开发指南是补充价值，但核心机制深度不足。

---

### 章节 14：沙箱架构与实现

**文档A 优势**：
- 用流程图展示了 `LocalSandbox` 路径映射的完整链路（容器路径 → 正向解析 → 越界检查 → subprocess 执行 → 反向解析）
- 解释了为什么 `AioSandbox` 不需要路径映射（路径真实存在于容器内），对比清晰
- 深入分析了 `AioSandboxProvider` 的三层获取（进程缓存 → warm pool → 后端发现）和 LRU 驱逐策略
- 从代码注释中提炼出 4 条实战 Tips（LocalSandbox 不是真隔离、read_file 反向解析范围、AIO 串行、保留路径冲突）

**文档B 优势**：
- 覆盖了 A 未涉及的安全配置（`allow_host_bash`）、路径遍历防护三层检查、线程数据隔离目录结构
- 包含错误处理异常层次图（SandboxError 及其子类）
- 给出了三种部署模式的配置对比表（本地/AIO本地容器/AIO Provisioner）

**裁决**：文档A 胜。A 对路径映射和 warm pool 生命周期的深度分析是核心差异；B 的覆盖更广但停留在概述层面，且 `aio_sandbox_provider.py:155-200` 引用与温池描述严重错位。

---

### 章节 19：长期记忆机制

**文档A 优势**：
- 深入解释了三个设计决策的 WHY：为什么防抖（用户连续追问）、为什么同步调用 LLM（避开 async 连接池 bug）、为什么注入到首条用户消息（prefix 缓存稳定）
- 用流程图展示了 fact 留存的三道闸门（confidence 阈值、去重、数量上限）
- 记忆数据结构的 ER 图清晰展示了 version/user/history/facts 的关系
- 52 个引用，密度极高，几乎每个论断都有源码支撑

**文档B 优势**：
- 覆盖了 A 未涉及的管理 API（`/api/memory` 系列 CRUD 端点）
- 包含 6 条最佳实践（置信度标注、上传过滤、防抖调优、词元预算、自定义存储）
- 提及未来演进路线（TF-IDF 上下文感知召回）

**裁决**：文档A 胜。A 对记忆注入策略和 fact 留存机制的深度分析远超 B；B 的管理 API 和最佳实践是实用补充，但核心技术深度差距明显。

---

## 总体结论

- **推荐使用：文档A**

- **各自适用场景**：
  - **文档A 适合**：需要深入理解系统内部机制、进行源码级二次开发、排查复杂问题的工程师；其精确的源码引用和 WHY 层面的解释使读者能建立准确的心智模型
  - **文档B 适合**：需要快速查阅配置参数、API 接口、部署选项的运维人员或产品经理；其管理 API 文档和配置表格更便于日常参考

- **改进建议给 文档A**：
  1. 补充开发指南（如 B 中的最小中间件实现模板、异步钩子示例）
  2. 增加管理 API 和配置参数的速查表，方便运维场景
  3. 第 14 章可补充错误处理异常层次和安全配置选项

- **改进建议给 文档B**：
  1. **最关键：大幅提高引用精度**，避免 300+ 行的大范围引用，应精确到函数/方法级别
  2. 增加 WHY 层面的解释（如"为什么用防抖""为什么注入到用户消息"），减少纯参数罗列
  3. 增加 mermaid 图表的数量和类型，尤其是 sequenceDiagram 和 stateDiagram 能帮助理解异步流程
  4. 纠正第 14 章中温池机制引用的错位问题
