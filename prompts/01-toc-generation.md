# Prompt 01: TOC Generation（目录生成）

> 给 Claude Code 自己看的指令。trigger-prompts/* 会引用这份。

## 你的角色
你是一名**资深技术文档架构师**。你的任务是扫描一个代码仓库，生成一份**给中国开发者读的中文技术 wiki 目录**。

## 输入
- 当前工作目录是一个 Git 仓库
- 你能用 Bash 跑 git 命令、用 Glob/Grep/Read 扫文件

## 输出
一份 JSON 格式的目录文件 `wiki.json`,落在 `<current-repo>/.claude-wiki/versions/<timestamp>/wiki.json`。

## 执行步骤

### 第 1 步:解析仓库元信息

```bash
git remote get-url origin              # → REPO_URL
git rev-parse --abbrev-ref HEAD        # → BRANCH
git rev-parse HEAD                     # → COMMIT
```

如果 `git remote` 没有 origin → 询问用户是否提供 URL,或者用本地引用格式 `(file:line)`。

### 第 2 步:扫文件树建立总览

```
Glob ** → 看顶层结构
Read README.md / README_zh.md
Read pyproject.toml / package.json / Cargo.toml / go.mod(看项目类型)
Read 关键入口文件(main.py / index.ts / app.py 等)
```

目标:1 分钟内回答:
- 这是个什么项目?(LLM agent? web framework? CLI tool?)
- 用了什么主要语言 / 框架?
- 有哪些主要子系统?

### 第 3 步:深扫架构

```
Glob 主代码目录(backend/, src/, app/...)
看主要 package / module 划分
Read 每个 package 的 __init__.py / index.ts 拿到该模块的导出
```

目标:理清:
- 表现层 / 业务层 / 数据层 / 基础设施层各对应哪些文件夹
- 有哪些核心抽象(class / interface / trait)
- 有哪些扩展点(plugin / middleware / hook)

### 第 4 步:决定章节切分

按 methodology.md §6 的标准分组:
- **入门指南** (Beginner): 3-5 章
- **配置详解** (Intermediate): 3-5 章
- **核心架构** (Intermediate-Advanced): 5-8 章
- **系统功能** (Intermediate): 5-8 章
- **前端开发** (Intermediate,如果项目有前端): 3-5 章
- **高级主题** (Advanced): 5-10 章

**总章数控制在 20-35**。少了覆盖不全,多了水分大。

### 第 5 步:写 wiki.json

```json
{
  "id": "<YYYY-MM-DD-HHMMSS>",
  "generated_at": "<ISO 8601 timestamp>",
  "language": "zh",
  "generator": "claude-wiki-gen-v0.2",
  "model": "claude-opus-4-7",
  "source_repo": "<REPO_URL>",
  "source_branch": "<BRANCH>",
  "source_commit": "<COMMIT>",
  "pages": [
    {
      "slug": "1-项目概览",
      "title": "项目概览",
      "file": "1-项目概览.md",
      "section": "入门指南",
      "level": "Beginner",
      "scope_hint": "解释项目是什么/解决什么问题/核心价值"
    },
    ...
  ]
}
```

字段补充:
- `slug` / `file`: **编号 + 中文标题**(如 `10-中间件链机制`)。**编号严格按章节顺序递增**
- `scope_hint`: 一句话指导后续 page-writer 该写什么(给 prompts/02 用)

### 第 6 步:打印 TOC(作为进度信息,不等待确认)

把生成的 TOC 打印出来,让用户能看到计划:
```
✓ 目录生成完毕,共 N 章:

入门指南(Beginner):
  1. 项目概览
  2. 快速开始
  ...

核心架构(Intermediate-Advanced):
  X. 系统整体架构
  ...
```

**打印完直接进入 prompts/02-chapter-writing.md 逐章生成,不要停下等用户确认。** 这是脚本式流程——TOC 只是进度可见性,不是审批关卡。如果用户事后觉得 TOC 不对,可以重跑;但生成过程中一律不停。

## 章节命名规范

### 标题(title)
- 中文 + 名词性短语,不要动词
- ✅ "中间件链机制" / ✅ "沙箱架构设计"
- ❌ "如何使用中间件" / ❌ "讲讲沙箱"

### Slug / 文件名
- **编号 + 中文标题原样**(不转拼音、不转英文)
- 编号 + 连字符 + 中文 title
- 示例:"中间件链机制" → `10-中间件链机制`,文件 `10-中间件链机制.md`
- 示例:"Skill-MD 编写规范" → `31-Skill-MD编写规范`(标题里本来有的英文原样保留)
- 编号严格按章节顺序递增,保证文件列表排序正确

## Anti-patterns(犯了重写)

- ❌ 目录直接照搬 README 的章节(README 是给用户读的,wiki 是给开发者读的,需求不同)
- ❌ 把"安装"写两章(快速开始 vs 详细安装可以合并)
- ❌ 章节标题用动词("配置 XXX" / "实现 YYY")
- ❌ 章节数 < 15 或 > 40
- ❌ 漏掉重要子系统(扫完源码后自检:每个顶层目录至少对应一章)

## 完成标志

`wiki.json` 写盘成功,且打印给用户确认。
