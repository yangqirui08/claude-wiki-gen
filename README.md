# 源码分析工具 (Claude-Wiki-Gen)

> **一句话**：用 Claude Code 给任意代码仓库生成一份高质量中文技术 wiki，对标 Zread / DeepWiki 输出格式。

## 这是什么

一套**方法论 + prompt 模板 + 触发指令**的组合系统，让 Claude Code（Opus 4.7）在一次会话内为指定仓库生成 25-30 章的结构化中文技术文档。

**不是**一个可执行程序——是一份给 Claude Code 看的"做 wiki 的 SOP"。Claude 读它就知道该怎么做。

## 怎么用

```
1. cd 到任意你想分析的仓库（Git repo,有 README 更好）
2. 打开新 Claude Code 会话(或 /clear 现有会话)
3. 复制 trigger-prompts/full-run.md 内容 → 粘贴进对话框 → 回车
4. 等 1-3 小时(看仓库规模),Claude 自动跑完全部章节
5. 输出落在 <你的仓库>/.claude-wiki/versions/<时间戳>/ 下
```

**模式速查**：

| 触发文件 | 用途 |
|---|---|
| `trigger-prompts/full-run.md` | 完整跑全部章节（首次/大改） |
| `trigger-prompts/partial-run.md` | 只跑指定几章（迭代/补章） |
| `trigger-prompts/quality-compare.md` | 跟已有 wiki 做质量对比 |

## 输出格式

采用同类 wiki 工具的通用 schema：

```
.claude-wiki/versions/<YYYY-MM-DD-HHMMSS>/
├── wiki.json                    ← 章节索引 + 元数据
├── 1-项目概览.md                 ← 单章 markdown (编号 + 中文标题命名)
├── 2-快速开始.md
├── ...
└── current                      ← 指针文件,内容是 versions/<最新时间戳>
```

每章 markdown 包含：
- **章节目标**（学完能干啥）
- **Overview（讲 WHY 不是 WHAT）** → **Architecture** → **Components** → **Data Flow** → **Implementation** → **References** → **Related Pages**
- **3-5 个 Mermaid 图**（mix 不同类型）
- **行号级源码引用**（`[file:line](github.com/.../#L42)`格式）
- **结构化表格**（summary table / comparison table）

## 文件夹结构

```
源码分析工具/
├── README.md                       ← 你正在读
├── methodology.md                  ← 核心 SOP(Claude 必读)
├── prompts/
│   ├── 01-toc-generation.md       ← 目录生成 prompt
│   ├── 02-chapter-writing.md      ← 单章写作 prompt
│   ├── 03-quality-check.md        ← 自查 prompt
│   └── 04-cross-reference.md      ← 章节互引 prompt
├── trigger-prompts/
│   ├── full-run.md                ← 完整运行触发
│   ├── partial-run.md             ← 部分章节触发
│   └── quality-compare.md         ← 对比评估触发
├── methodology-sources/
│   └── microsoft-deep-wiki-notes.md ← 蓝本笔记
├── examples/
│   └── deer-flow-output/          ← 验证时生成的样例产物(deer-flow 3 章)
└── validation/
    ├── 对照报告-Kimi盲评.md         ← v0.2 定版依据:两份独立盲评
    └── 对照报告-Claude盲评.md
```

## 设计理念（基于 Microsoft deep-wiki 10 原则改编）

1. **源码追溯**：每个非微小断言带行号引用，文件名 + 行号必须可点
2. **结构优先**：先生成 TOC，再写章节内容
3. **证据驱动**：每个论断引具体代码位置——不臆测
4. **图表丰富**：每章 3-5 个 Mermaid，混合 2+ 类型
5. **表格化**：结构化信息优先用表格不用段落
6. **渐进展开**：从大图到细节，每节开头先 TL;DR
7. **层级控制**：组件级最多 4 层深度
8. **系统思维**：架构 → 子系统 → 组件 → 方法
9. **绝不编造**：所有内容来自实际代码追溯
10. **中文母语**：内容写给中国开发者读，自然中文不生硬翻译

## 限制 / 不适合的场景

- ❌ 私有 / 商业敏感 repo（你不想代码被 Claude 处理就别用）
- ❌ 仓库大到无法在 Claude 1M context 内完整扫描（数百万行的 monorepo）
- ❌ 非代码项目（纯文档库 / 数据集 / 多媒体）
- ❌ 强时效性要求（生成完上游再改你的 wiki 就过期了）

## 跟现有工具的关系

| 工具 | 模型 | 优势 | 不足 |
|---|---|---|---|
| Zread | GLM 系列 | 国内访问顺、便宜 | 国产模型代码理解略弱 |
| DeepWiki | Cognition 自家 | 英文质量高 | 国内访问不稳 |
| **本系统 + Claude Opus 4.7** | Claude | **代码理解最强 + 长上下文** | 烧 Claude 配额 |

**这套系统是"用 Claude 强能力 + 别人验证过的方法论"** 的产物，不是另起炉灶。

## 版本

- **v0.2 (2026-05-15)** — 经 deer-flow 3 章对照验证后定版。改进:slug 改中文标题命名;`§6.1` 章节示例标注按项目类型派生;`prompts/02` 新增"速查表"+"扩展指南"两个条件性必须段。
- v0.1 (2026-05-15) — 初版，基于 Microsoft deep-wiki 改编。

### 验证结果(v0.1 → v0.2)

在 deer-flow 第 10/14/19 章做盲评对照(基线: Zread 同章),两个独立裁判:

| 裁判 | 本系统 | 基线 |
|---|---|---|
| Kimi | 70 / 80 | 48.5 / 80 |
| Claude | 71.5 / 80 | 57.5 / 80 |

本系统在深度 / 引用精度 / 图表 / 准确性上明显领先;吸收了基线在"速查表 / 扩展指南"上的优点。验证产物存 `validation/`。

## 维护

- **本质是 prompt 工程**，模型升级 / 上游 deep-wiki 更新时跟着调
- 如果某次产出质量不行，先改 `prompts/`，不要改 trigger-prompts（trigger 是稳定接口）
- 学到的新教训写进 `methodology.md` 的"Lessons Learned"区
