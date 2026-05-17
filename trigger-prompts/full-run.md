# 触发指令:完整运行(Full Run)

> **用法**:打开新 Claude Code 会话(或在现有会话 `/clear`),把下面整段(从"---你的任务---"开始到末尾)复制粘贴进对话框,回车。
>
> **这是脚本式触发**:粘贴后全自动跑到底,**不需要中途确认、不需要握手**。读完资料直接开干,跑完(或上下文吃紧)再报告结果。

---你的任务---

你是一名资深技术文档工程师。你的任务是为**当前工作目录的代码仓库**生成一份高质量中文技术 wiki。

**这是全自动任务:不要在中途停下来问"是否继续 / 是否确认 TOC"——读完资料后直接执行到底,跑完或上下文吃紧才停下报告。**

## 必读资料(按顺序读完再开干)

1. `D:\Projects\源码分析工具\methodology.md` — 完整 SOP,10 条原则 + 工作流 + 输出 schema + 引用格式 + Mermaid 规范 + 章节切分 + 检索策略 + 质量自检 + 时间预算 + lessons learned
2. `D:\Projects\源码分析工具\prompts\01-toc-generation.md` — 阶段 B 怎么生成目录
3. `D:\Projects\源码分析工具\prompts\02-chapter-writing.md` — 阶段 C 怎么写单章
4. `D:\Projects\源码分析工具\prompts\03-quality-check.md` — 每章保存前怎么自检
5. `D:\Projects\源码分析工具\prompts\04-cross-reference.md` — 全部完成后怎么补章节互引

## 工作流(读完资料直接按此执行,无中途确认)

```
阶段 A: 准备(< 5 min)
  - git remote / branch / commit
  - 扫文件树 + 读 README
  - 决定 REPO_URL + BRANCH

阶段 B: TOC 生成(~10 min)
  - 按 prompts/01 跑
  - 输出 wiki.json
  - 把 TOC 打印出来作为进度信息,但【不等待确认】,直接进阶段 C

阶段 C: 逐章生成(主要时间)
  - 按 prompts/02 一章一章跑(Beginner → Advanced 顺序)
  - 每写完一章按 prompts/03 自查
  - 每章保存后报告一行: "✓ 5. 模型配置 (3.2 KB, 4 mermaid, 7 文件引用)"

收尾:
  - 按 prompts/04 补 Related Pages
  - 更新 输出目录的 current 指针
  - 给最终总结
```

## 输出路径

默认 `<current-working-dir>/.claude-wiki/versions/<YYYY-MM-DD-HHMMSS>/`。
**若触发消息里另给了输出目录,以那个为准。**

```
<输出目录>/versions/<YYYY-MM-DD-HHMMSS>/
├── wiki.json
├── 1-<中文标题>.md
├── 2-<中文标题>.md
├── ...
<输出目录>/current  ← 内容为 versions/<最新时间戳>
```

## 关键约束(违反任一直接 fail)

1. **绝不编造**:你没读过的代码,写"未深入分析,建议补读 X" 或不写。**不要**写"应该 / 大概 / 可能"
2. **行号引用**:每个非微小断言带 `[file:line](github URL)`
3. **3-5 Mermaid 每章 + 2+ 类型**
4. **章节顺序**:Overview(WHY) → Architecture → Components → Data Flow → Implementation → References → Related Pages
5. **中文自然不机翻**
6. **文件名 = 编号 + 中文标题**(如 `10-中间件链机制.md`)

## 中途不停的原则

- TOC 生成完**不等确认**,直接写章节
- 单章自查不过**自己改**,不问用户
- 遇到某章源码太散/难判断 → 写"⚠️ 本次扫描发现 X,需补读 Y",**继续下一章**,不停下
- **唯一允许的停止点**:上下文吃紧,跑不动了 —— 此时优雅结束,报告"已完成 X/N 章,剩余 [章节列表] 请新开 session 用 partial-run"

## 时间预算

- 中等仓库(deer-flow 量级 5-50k LOC): 4-8 小时
- 单 session 大概率跑不完全部章节,按上面"中途不停原则"处理

## 启动

读完 5 份必读资料后,**直接从阶段 A 开始执行**,不要回复确认、不要等待。第一条输出可以是简短的"已读完 prompts,开始为 <REPO> 生成 wiki",随后立即进入阶段 A。

---结束---
