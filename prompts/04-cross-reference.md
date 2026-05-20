# Prompt 04: Cross-Reference（章节互引补全）

> 全部章节生成完后跑一次。补全 Related Pages,让 wiki 形成网状。

## 执行步骤

### 第 1 步:建立章节关系图(在脑中)

读完 `wiki.json` 的 pages[],画出概念关系:

```
项目概览 ←─── 所有章节都引用
系统架构 ←─── 中间件链 / 沙箱 / Agent / 网关
中间件链 ←──→ 鉴权 / 内存注入 / 沙箱预备(它们是 middleware 子类)
沙箱架构 ←──→ 本地沙箱 / Docker 沙箱 / 路径映射(实现/扩展)
                       ↑
Lead Agent ←──── 子代理任务委派 ←─── 工具系统
                       ↑
长期记忆 ←──→ 记忆提取与注入
```

### 第 2 步:为每章补 Related Pages

每章末尾的 Related Pages 表必须满足:

| 关系类型 | 何时用 | 示例 |
|---|---|---|
| 「使用」 | A 章描述的组件用了 B 章的能力 | 中间件链 "使用" Agent 接口(see Agent 章) |
| 「被使用」 | A 章描述的组件被 B 章引用 | Sandbox 章被 Agent / 工具开发等多章引用 |
| 「子集」 | A 是 B 的特例 | LocalSandbox 章 < Sandbox 架构章 |
| 「兄弟」 | A B 是平行实现 | LocalSandbox / DockerSandbox |
| 「前置」 | 读 B 前最好先读 A | "项目概览" 是几乎所有 Advanced 章的前置 |
| 「延伸」 | 想深入可去 B | "测试策略" 延伸自 "贡献指南" |

### 第 3 步:**双向**链接

A 引用 B → B 也必须引用 A。
- 如果 A 章 Related Pages 列了 B,反过来去看 B 章 Related Pages,**必须列 A**
- 不强求关系类型对称(A 「使用」 B,B 可以 「被使用」 A)

### 第 4 步:孤岛检查

Grep 整个 .claude-wiki/versions/<timestamp>/ 找:
- 没有任何其他章节链接到它的章节 = 孤岛
- 链接 < 2 个的章节 = 半孤岛

孤岛 / 半孤岛章节:
- 看看是不是该章过于独立(可能可以合并)
- 或者其他章节漏链了

### 第 5 步:写盘

为每章追加/重写 Related Pages 段(如果原有的不全)。

### 第 6 步:独立 subagent 终审(强烈建议)

主 agent 自己 grep 容易漏自己写过的章节(盲点)。**派 1 个 subagent 用 fresh ctx 做独立终审**,产出一份"问题清单",主 agent 拿来修。

为什么用 subagent:本步骤的工作性质天然适合"没有写作上下文"的视角——校验员不该是作者本人。

派单模板(主 agent 拷贝调整后发 Agent 工具,设 `run_in_background: true`):

```
你是技术文档独立审计员。任务:对 <WIKI_DIR> 下的 wiki 做最终独立校验,产出问题清单。

请按以下顺序检查并返回 4 段报告:

## 1. 坏链
对 WIKI_DIR/*.md 文件 grep 所有 ./N-xxx.md 形式的链接,验证目标文件实际存在。
列出每个坏链: "<src>.md 第 X 行 → <broken-target>.md (不存在)"

## 2. 孤岛/半孤岛
统计每章入度(被多少其他章节链接)。
- 入度 = 0 → 孤岛(必须补)
- 入度 = 1 → 半孤岛(建议补)
列出每个: "<slug>.md 入度=X"

## 3. 行号引用抽查(从 5+ 不同章节随机抽 8 处)
对每处 [path:line](url) 引用:
- Read 实际源码该行
- 跟章节里描述的内容是否对得上
- 标记 ✅ 准确 / ⚠️ 行号偏 ±3 / ❌ 完全不符

## 4. 术语一致性
对 wiki.json 里的核心术语,grep 出在不同章节的翻译是否一致(例: middleware 在某章叫"中间件"另一章叫"拦截器" → 报)。

## 输出
返回 4 段报告,每段列具体位置(文件:行号)。不修文件,只产报告。

WIKI_DIR: <填具体路径>
REPO_URL: <填 wiki.json 里的 source_repo>
```

主 agent 拿到报告后:
- 坏链 → 直接 Edit 改
- 孤岛 → 给相关章节补反向引用
- 引用 ❌ → 改行号或重写描述
- 术语不一致 → 统一翻译

## 链接格式规范

### 同目录(都在 .claude-wiki/versions/<X>/ 里)
```markdown
[中间件链机制](./10-中间件链机制.md)
```

> ⚠️ slug 用中文标题原样(v0.2 起规则,见 [01-toc-generation §章节命名](01-toc-generation.md))。**不要转拼音、不要转英文**。

### 锚到具体小节
```markdown
[中间件链机制 §before/after hook](./10-中间件链机制.md#beforeafter-hook)
```

### 不要写空标题
❌ `[10-中间件链机制.md](./10-...)`
✅ `[中间件链机制](./10-...)`

## 反模式

### 反模式 1: 列了一堆但没说关系
❌
```
## Related Pages
- [Auth](./auth.md)
- [Memory](./memory.md)
```

✅
```
## Related Pages

| Page | Relationship |
|---|---|
| [Auth](./auth.md) | 本章描述的 AuthMiddleware 是 §3 提到的实现 |
| [Memory](./memory.md) | MemoryMiddleware 跟 AuthMiddleware 串在同一条 chain |
```

### 反模式 2: 全部章节都被链
"项目概览" 可以被很多章引(它是入门入口),但**不是每个章节都链所有 Beginner 章**。只链**真有概念依赖**的。

### 反模式 3: 单向链接
A 章引 B,但 B 章 Related Pages 没列 A → 必须补。

## 完成标志

- 每章 Related Pages 至少 2 个链接
- 整个 wiki 没有孤岛章
- 所有链接是双向的
