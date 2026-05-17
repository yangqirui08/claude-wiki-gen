# Microsoft deep-wiki 学习笔记

> 来源: https://github.com/microsoft/skills/tree/main/.github/plugins/deep-wiki
> 阅读时间: 2026-05-15
> 关键文件: README.md + skills/wiki-architect/SKILL.md + skills/wiki-page-writer/SKILL.md

## 为啥选这个作蓝本

1. **"Distilled from OpenDeepWiki and deepwiki-open"** — 不是另起炉灶,是吸收两个前辈的精华
2. **Microsoft 工程质量** — 不会犯低级 bug
3. **格式就是 Claude skill** — SKILL.md + frontmatter,可以直接当 Claude Code prompt 用
4. **拆分清晰** — architect(TOC) + page-writer(content) + 10 个辅助 skill,职责单一

## 核心原则 10 条(直接挪用)

1. Source-linked citations
2. Structure-first(TOC before content)
3. Evidence-based(no speculation)
4. Diagram-rich(3-5 mermaid per page, 2+ types)
5. Table-driven(tables > paragraphs for structured info)
6. Progressive disclosure(TL;DR → details)
7. Hierarchical depth(max 4 levels)
8. Systems thinking(architecture → subsystems → components → methods)
9. Never invent
10. Dark-mode native

## 我吸收的关键点

### A. 工作流拆分(三阶段)
- 阶段 A: Source Repository Resolution(git remote / branch)
- 阶段 B: Catalogue(TOC as JSON)
- 阶段 C: Per-chapter content writing

我的版本同样三阶段,但加了"用户 TOC 确认"作为 B → C 的关卡。

### B. 引用格式
微软原版: `[file:line](REPO_URL/blob/BRANCH/file#Lline)`
我的版本: 完全一致(便于 Zread 对比)

### C. Mermaid 规范
微软原版规定的:
- 暗色配色(节点 `#2d333b` + stroke `#6d5dfc` + text `#e6edf3`)
- subgraph 背景 `#161b22`
- 行内 style 用 `color:#e6edf3`
- 禁用 `<br/>` (用 `<br>`)
- hex 色必 6 位
- sequenceDiagram 必带 autonumber

完全照搬。

### D. Citations 要求
微软原版:
- 每页至少 5 个不同文件被引用
- Mermaid 后必有 `<!-- Sources: ... -->` 注释
- 表格里有代码组件必加 "Source" 列
- 引用没找到时写 `(Unknown – verify in path/to/check)`

我的版本:基本照搬,把"Unknown"改成中文:`⚠️ 本次扫描未深入分析,建议补读 X 验证`。

### E. 章节结构(必有)
微软原版:
```
Overview → Architecture → Components → Data Flow → Implementation → References → Related Pages
```

我的版本:
- 加了"章节目标"(3 条 学完能干啥)放最顶
- 加了 TL;DR(3-5 句话)放 Overview 之前
- 把 Implementation 标记为"可选,Advanced 章才详写"
- 加了 Configuration 和 Common Pitfalls 作可选段

### F. 四种 audience 的 Onboarding(微软独有)
微软强调生成 4 份 onboarding:
1. Contributor Guide(新贡献者)
2. Staff Engineer Guide(staff/principal IC)
3. Executive Guide(VP/director)
4. Product Manager Guide(PM)

**我没采用**——原因:
- 我们的 wiki 目标是开发者读,不是给多角色看
- 加 4 个 onboarding 章节增加复杂度,且大部分内容会跟"项目概览" / "快速开始"重叠
- Zread 输出也没这种分层,加了反而不便对比

如果将来要支持多角色 onboarding,可作 v0.2 增项。

## 我加的(微软没有)

1. **章节切分指南**(methodology.md §6): 给中文项目的标准分组(入门 / 配置 / 核心架构 / 系统功能 / 前端 / 高级主题)
2. **检索策略**(methodology.md §7): 明确"Glob → Grep → Read → 跟随引用"的 RAG 流程,因为 Claude Code 没有真 RAG 系统
3. **Token predicate**(prompts/02 §第 1 步): 单章读源码 30-100k token 控制
4. **Zread schema 对齐**(methodology.md §3): wiki.json 字段、kebab-case 拼音 slug、文件结构
5. **反模式列表**(prompts/02 末尾): 5 种常见错误产物
6. **质量自检 checklist**(prompts/03): 11 条单章 + 5 条整体

## 没采用的微软设计

| 微软的 | 我没用 | 原因 |
|---|---|---|
| VitePress frontmatter | ❌ | 输出对齐 Zread,Zread 没用 frontmatter |
| AGENTS.md 生成 | ❌ | 是给 agent 用的,不是给人读的 wiki 一部分 |
| llms.txt 生成 | ❌ | 同上,不是 wiki 内容 |
| Azure DevOps 转换 | ❌ | 我们不用 ADO |
| changelog skill | ❌ | wiki 跟 changelog 是两件事 |
| ado / build / deploy 等 13 个 slash command | ❌ | Claude Code 不用 slash command 这种形态 |
| 4 audience onboarding | ❌ | 见上文 §F |

## 关键 quote(原文挪)

来自 wiki-page-writer 的 NEGOTIABLE 不可妥协深度要求:

> 1. TRACE ACTUAL CODE PATHS — Do not guess from file names. Read the implementation.
> 2. EVERY CLAIM NEEDS A SOURCE — File path + function/class name.
> 3. DISTINGUISH FACT FROM INFERENCE — If you read the code, say so. If inferring, mark it.
> 4. FIRST PRINCIPLES — Explain WHY something exists before WHAT it does.
> 5. NO HAND-WAVING — Don't say "this likely handles..." — read the code.

我把这 5 条翻译进 methodology.md §1 第 1-9 项原则。

## 我会持续跟进的

微软的 plugin 会更新,如果他们加了我没考虑到的设计,值得回看。但**不照搬,只学习**——他们工具用法跟我们不同(他们是 Copilot CLI plugin,我们是 Claude Code agent 直接执行)。

下次审查时机:遇到某个 wiki 章节写出来质量不好 → 回头看微软最新版是不是改了什么 → 选择性吸收。
