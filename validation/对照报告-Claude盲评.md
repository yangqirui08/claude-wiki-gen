# WIKI 质量对比报告

> 盲评说明:全程以「文档A / 文档B」称呼,不推断生成工具/模型。元数据表中的 `model`/`generator` 字段为 wiki.json 文件中的客观内容,如实记录,**不影响评分**。
>
> - 文档A:`D:\Projects\源码分析工具\examples\deer-flow-output\versions\2026-05-15-162457\`
> - 文档B:`D:\我的仓库\deerflow研究\st-二开指导\`
> - 对比章节:第 10、14、19 章
> - 源码验证仓库:`D:\我的仓库\deerflow研究\deer-flow`(HEAD `6e8e6a9…`,2026-05-13)
> - 评审日期:2026-05-15

## 基础元数据

| | 文档A | 文档B |
|---|---|---|
| 本次对比章节 | 第 10、14、19 章 | 第 10、14、19 章 |
| wiki 总章数 | **3 章**(wiki.json 仅列 3 页) | **37 章**(完整导览) |
| 目标 3 章平均大小 | **23.6 KB** | **13.9 KB** |
| 3 章 mermaid 总数 | **15**(graph/classDiagram/stateDiagram/sequenceDiagram/flowchart/erDiagram 6 型) | **11**(flowchart/sequenceDiagram/classDiagram/erDiagram/stateDiagram 5 型) |
| 引用粒度 | 行内 `[文件:行](url)`,**精确 5–30 行区间** | 段末 `Sources: [文件](url)`,**多为整文件/宽区间**(1-373、1-228、1-141) |
| 生成日期 | 2026-05-15 | 2026-04-03 |
| source_commit | `6e8e6a9…`(= **当前仓库 HEAD**) | wiki.json 未记录(早 HEAD 约 6 周) |
| wiki.json model 字段 | `claude-opus-4-7` | 无 |

> **公允性前提**:文档A 生成于源仓库当前 HEAD,行号天然对得齐;文档B 早 6 周,部分行号漂移**不计为 B 的过错**。但 B 选择「整文件区间」是与漂移无关的引用风格选择;B 的概念性错误亦与漂移无关。下文严格区分这两类。

## 维度评分

| 维度 | 文档A | 文档B | 差异说明 |
|---|---|---|---|
| 1. 覆盖度 | **8** / 10 | **8** / 10 | 平。B 更广(逐中间件细节、中间件开发指南、ThreadState、`/api/memory` 端点、未来路线、异常体系、双后端、Subagent 4 件子集);A 更全于核心机制(生产链 `_build_middlewares` 完整列出 LLMError/SandboxAudit/DynamicContext/Token/DeferredTool 等 5 个 B 表格遗漏的中间件) |
| 2. 深度 | **9** / 10 | **6.5** / 10 | A 每章以「Overview(为什么)」给出朴素做法+3 个具体痛点+设计回答;讲透洋葱语义、fail-fast、prefix 缓存动机、同步 invoke 避开 async 连接池的动机。B 多为 WHAT 优先 |
| 3. 准确性 | **9.5** / 10 | **5.5** / 10 | 5+ 抽样:A 全部 ✅;B 多处 ❌(见下)。B 的 `middleware-execution-flow.md` 数据源本身含错,B 继承之 |
| 4. 结构清晰 | **9** / 10 | **8** / 10 | A 三章骨架完全一致(目标→TL;DR→Overview→Arch→Components→Pitfalls→Ref→Related);B 也清晰但略随章浮动 |
| 5. 可读性 | **9** / 10 | **8.5** / 10 | 均流畅、术语一致。A 行文更有带入感;B 更简洁偏教科书 |
| 6. 图表质量 | **9** / 10 | **7.5** / 10 | A 图更多更杂、每图带 `<!-- Sources -->`;B 图干净易读但 §10 流程图含事实错误、类型略少 |
| 7. 引用质量 | **9.5** / 10 | **5.5** / 10 | A 行内链接+精确小区间+逐论断引;B 整文件宽区间,且 `_insert_extra` 实为 306-379、B 引 290-373 错位 |
| 8. 互引网络 | **8.5** / 10 | **8** / 10 | A 三章互引成完整三角、带「Relationship」说明、双向;B 链入 37 节点大网但多为单向 plain list |
| **总分** | **71.5 / 80** | **57.5 / 80** | |

## 引用准确性抽查

### 文档A 抽样(均在当前 HEAD 逐行核对)

1. `factory.py:155-188` — ✅ 精确。该区间正是列出「0-2 沙箱…13 Clarification」的 14 段链 docstring,A 称「14 段」属实。
2. `factory.py:218-223` — ✅ 精确。正是 `summarization` 块的 `raise ValueError("summarization=True requires…")`。
3. `factory.py:306-379` — ✅ 精确。`_insert_extra` 函数 def 在 306、止于 379。
4. `features.py:14-34` — ✅ 精确。`RuntimeFeatures` 数据类,**含** `loop_detection=True`(第 34 行)。
5. `dangling_tool_call_middleware.py:160-169` — ✅ 精确。正是 `wrap_model_call`(源码 docstring 明写 "Uses wrap_model_call instead of before_model")。
6. `summarization_middleware.py:120` — ✅ 精确。第 120 行正是 `def before_model(...)`。
7. `sandbox.py:18-93` — ✅ 精确。7 个 `@abstractmethod`,A 逐一列名(execute_command/read_file/list_dir/write_file/glob/grep/update_file)全对。
8. `aio_sandbox.py:78-82` — ✅ 精确。A 引的 ErrorObservation 重试代码块逐行一致。
9. `dynamic_context_middleware.py:84-89 / 104-119` — ✅ 精确。docstring 明写「注入首条 HumanMessage、冻结以命中 prefix 缓存」。
10. `memory_config.py:9/27/31/37/41/47/53/57` — ✅ **8/8 单行号全部命中**。

> A 唯一可挑的瑕疵:第 10 章称依赖注释「在 `make_lead_agent` 顶部」,实际该注释紧贴 `_build_middlewares`(由 `make_lead_agent` 调用),属极轻微措辞松动,行号 230-239 仍精确。

### 文档B 抽样

1. `factory.py:290-373`(标 `_insert_extra`)— ❌ **区间错位**。函数实为 306-379,B 起点早 16 行(落在上一函数内)、终点早 6 行。
2. `sandbox.py:1-73` — ❌ **内容欠数**。B 称「五大能力」,源码实有 **7 个**抽象方法(漏 `glob`、`grep`,且这两者就在 B 所引区间内)。
3. `features.py:1-63` — ⚠️ B 引此处的 `RuntimeFeatures` 代码块**漏掉 `loop_detection` 字段**(源码第 34 行)。
4. `loop_detection_middleware.py:1-228` — ❌ B 引的 `after_model` 实在第 420 行,**不在所引区间内**(可部分归因 4→5 月代码增长)。
5. `middleware-execution-flow.md` — ⚠️ 该文档确存在;但**文档本身把 DanglingToolCall、Summarization 标为 `after_model`(错)**,B 继承了这些错。
6. `memory_config.py:1-83` — ✅ 内容准确(整文件区间,文件实 84 行)。
7. `prompt.py:62-95`(fact 数据模型)— ❌ 区间有效,但 B 正文列「5 种 category」,源码第 71-76 行(在区间内)明列 **6 种**(漏 `correction`)。
8. B「记忆注入系统提示词」— ❌ 源码 `dynamic_context_middleware.py` 实为注入**首条用户消息的 `<system-reminder>`**,设计动机正是「保 system prompt 静态」。B 该章**完全未引** `dynamic_context_middleware.py`。

## 抽样章节深度对比

### 第 10 章:中间件链机制

**文档A 优势**
- 正确区分**两类 hook**:点状 hook(before/after,线性管道)vs 包裹 hook(`wrap_*`,真洋葱);并给出 ToolError→Clarification 嵌套的代码+时序图。
- 正确区分**两条装配路径**:`create_deerflow_agent`/`_assemble_from_features`(RuntimeFeatures 驱动)vs `make_lead_agent`/`_build_middlewares`(AppConfig 驱动),且把生产链每个中间件逐一列全。
- `@Next/@Prev` 的 `_insert_extra` 四步迭代算法、循环依赖检测讲到位。

**文档B 优势**
- 一张「中间件 × 钩子」对照表 + 主 Agent/Subagent 列,速查性好;补充了 ThreadState 状态合并、中间件开发模板(异步/测试)——A 未涉及。
- 给出 LoopDetection 默认阈值(警告 3 / 硬限 5 / 窗口 20)、SubagentLimit 并发上限等具体数字。

**裁决:A 胜(明显)**。B 的核心表格有**两处事实错误**(DanglingToolCall 应为 `wrap_model_call`、Summarization 应为 `before_model`),流程图同错;并把 `make_lead_agent` 误说成「`RuntimeFeatures` 组装」(混淆两条路径);「并非洋葱模型」是对一份本身不精确的仓库文档的忠实转述,但落地为误导性结论。A 三项均正确。

### 第 14 章:沙箱架构与实现

**文档A 优势**
- 路径映射讲到**安全本质**:正向/反向解析、`..` 越界 `PermissionError`、只读挂载 `EROFS`,并配源码摘录;`Sandbox` 接口 **7 个方法**列名全对。
- 懒获取链路、`thread_id→沙箱` 复用、warm pool 状态机、两 provider 释放策略差异清晰。
- Pitfall「`LocalSandbox` 不是真隔离——无路径前缀的 `rm -rf ~` 不被拦」一针见血。

**文档B 优势**
- 覆盖更广:`LocalContainerBackend` vs `RemoteSandboxBackend` 双后端、`SandboxError` 异常体系、`allow_host_bash` 能力门控、线程目录 `0o777` 与 `thread_id` 正则校验——均 A 未涉及。
- 配置三模式对照表实用。

**裁决:A 胜(中等)**。B 覆盖面更宽且无致命错,但「五大能力」漏 `glob/grep`、类图把 `AioSandbox` 隐入「SB」略含糊;A 在隔离机制与生命周期的深度、引用精度上明显更强。

### 第 19 章:长期记忆机制

**文档A 优势**
- 注入机制**完全正确**:记忆进首条用户消息的 `<system-reminder>`、首轮冻结、为 prefix 缓存服务——逐字符合 `dynamic_context_middleware.py` docstring。
- 防抖队列按 thread 去重、`correction/reinforcement` 信号 or 合并、同步 `invoke` 避开 async httpx 连接池、原子写盘、三道留存闸门——均讲到 WHY。
- fact 的 6 种 category(含 `correction`)正确。

**文档B 优势**
- 补充 `/api/memory` 全套 REST 端点、`MEMORY_IMPROVEMENTS.md` 的 TF-IDF 未来路线(明确标注「未合并」)——A 未覆盖,是真实的广度增量。

**裁决:A 胜(明显)**。B 的「记忆注入系统提示词」是实质性错误(且 B 漏引最关键的注入中间件),category 漏 `correction`;A 在该章既准又深。

## 总体结论

- **推荐使用:文档A**(71.5 vs 57.5 / 80)。

- **各自适用场景**
  - **文档A 适合**:需要**可信地照着改代码**的深度读者——引用精确到行、每个论断可回源验证、讲透设计动机。是「二次开发指导」性质的首选。
  - **文档B 适合**:需要**快速建立全局图景**的读者——37 章覆盖前端/IM/测试/部署等 A 没有的领域,单章更短易扫读,适合入门期纵览。但**任何要落到代码的论断都应回源码复核**。

- **给文档A 的改进建议**
  1. 第 14 章对 `LocalContainerBackend`/`RemoteSandboxBackend` 双后端、异常体系着墨偏少,可补一节。
  2. 补全广度:仅 3 章,缺少 B 覆盖的逐中间件细节、Subagent 子集、管理 API 等;若要做完整 wiki 需扩章。
  3. 「`make_lead_agent` 顶部注释」措辞可精确为「`_build_middlewares` 上方注释」。

- **给文档B 的改进建议**
  1. **准确性是首要短板**:不要把仓库内 `docs/*.md` 当一等数据源——`middleware-execution-flow.md` 自身就把 DanglingToolCall/Summarization 的钩子标错;应直接读中间件源码核对。
  2. 修正三处实质错误:DanglingToolCall=`wrap_model_call`、Summarization=`before_model`、记忆注入=首条用户消息的 `<system-reminder>`(非 system prompt)。
  3. 引用应细化到**函数级行区间**而非整文件(`1-373`/`1-228` 这类对读者定位帮助有限);`_insert_extra` 这类区间需对准函数边界。
  4. 区分 `create_deerflow_agent`(RuntimeFeatures)与 `make_lead_agent`(AppConfig)两条装配路径;补 `glob/grep` 能力与 `correction` 分类。
  5. 若条件允许,基于当前 HEAD 重新生成以消除 6 周漂移。
