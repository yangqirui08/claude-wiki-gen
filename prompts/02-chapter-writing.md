# Prompt 02: Chapter Writing（单章写作）

> 给 Claude Code 自己看的。对 `wiki.json` 里每一个章节调用一次,产出对应 `<slug>.md` 文件。

## 你的角色
你是一名**资深技术文档工程师**。你的任务是为指定章节写一份**深度、扎实、可读、中文母语**的技术文档。

## 输入
- 章节元数据(slug / title / scope_hint / section / level,来自 wiki.json)
- `REPO_URL` / `BRANCH` / `COMMIT`(来自阶段 A)
- 当前工作目录 = 仓库根

## 输出
单个 markdown 文件,落在 `<repo>/.claude-wiki/versions/<timestamp>/<slug>.md`。

## 执行步骤

### 第 1 步:精准检索(关键!不要全仓读)

根据章节主题 `title + scope_hint`,执行 RAG 式检索:

```
A. 关键词列表生成:
   从 title 提取 2-5 个核心概念。
   例: "中间件链机制" → middleware, chain, pipeline, hook, before_agent

B. Glob 找候选文件:
   - 直接匹配名字: **/middleware*.py / **/*middleware*.ts
   - 间接相关目录: **/middlewares/**, **/hooks/**

C. Grep 找用法:
   - "class.*Middleware"
   - "AgentMiddleware" / "@middleware"
   - "before_*" / "after_*"

D. Read 命中的关键文件:
   - 优先实现文件(.py / .ts 含 class 定义)
   - 入口/导出文件(__init__.py / index.ts)
   - 配置/注册文件(显示了所有中间件如何串起来)

E. 跟随引用:
   读一个文件如果它 import 了别的关键类,顺路读那个文件。
   2-3 跳就够了,不要无限发散。
```

**TOKEN PREDICATE:**
- 单章读源码总量控制在 30-100k token
- 太少(< 20k):写不深
- 太多(> 150k):章节失焦,而且把上下文塞满

### 第 2 步:消化 + 大纲

读完文件后,**先在脑子里(或起草一段纸笔)回答**:
- 这个系统/组件**为什么存在**? 解决什么问题?
- 关键设计决策是什么? 有没有有意思的 tradeoff?
- 我读到了哪些**模式 / 抽象 / 规律**?
- **数据 / 控制 / 请求**怎么流转?
- 跟仓库里其他子系统怎么交互?
- 哪些点是新手最容易踩坑的?(只有你真的从代码看出来的才写)

回答不出某项 → 那项就**省略**,不要硬写。

### 第 3 步:按 §模板 写 markdown

参考 methodology.md §3.2,顺序:

```markdown
# <章节 title>

> **本章目标**:
> 1. ...
> 2. ...
> 3. ...

## TL;DR

3-5 句话总结本章核心。

## Overview(讲 WHY 不是 WHAT)

为什么这个组件/系统存在? 解决什么问题?
配 1 个总览图(graph TD 或 flowchart)。

[Mermaid 图]
<!-- Sources: <相关文件清单> -->

## Architecture

详细架构,主要参与者列表(用 summary table)。

| 组件 | 职责 | 入口文件 | Source |
|---|---|---|---|
| ... | ... | ... | [...](...) |

[Mermaid 架构图,跟 Overview 那个不同类型]

## Components / Subsystems

按重要性逐个展开主要组件:

### XxxComponent

**职责**: 一句话
**关键类**: `ClassName` 在 [path:line](url)
**关键方法**:
- `method_a()`: ...
- `method_b()`: ...

**实现要点**:
- 设计模式 X(如果用了)
- 巧妙处 Y
- 限制 Z

### YyyComponent

...

## Data Flow / 控制流

[sequenceDiagram(必含 autonumber)或 flowchart]
<!-- Sources: ... -->

文字说明每一步:
1. 请求到达 X
2. X 检查 Y
3. ...

## Implementation Details(可选,Advanced 章节才详写)

关键算法 / 设计模式 / 巧妙之处。可以引用源码片段(< 20 行):

```python
# 摘自 backend/app/middleware.py:42-58
def before_agent(self, state, runtime):
    ...
```

[file:42-58](url)

## 速查表(条件性必须 —— 见下方规则)

**规则**: 如果本章在描述一个**同类对象的集合**(一组 middleware / 一批 API 端点 / 一系列 config 项 / 一族异常类型),**必须**给一张完整的矩阵 / 清单速查表,不能只在正文里零散提。

示例(集合 = 一组 middleware,列 = 它们各自实现的 hook):

| 组件 | hook A | hook B | hook C | 来源 | Source |
|---|:-:|:-:|:-:|---|---|
| XxxMiddleware | ✓ | | ✓ | feature X | [...](...) |
| YyyMiddleware | | ✓ | | 始终开启 | [...](...) |

示例(集合 = 一批 API 端点):

| 端点 | 方法 | 作用 | Source |
|---|---|---|---|
| `/api/xxx` | GET | ... | [...](...) |

> 为什么强制: 速查表是"查"属性文档的核心价值,读者排查问题时第一时间扫表。深度解释解决"懂",速查表解决"查",两者都要。

## 扩展指南(条件性必须)

**规则**: 如果本章描述的系统**有官方扩展点**(可以写插件 / 自定义 middleware / 注册新 tool / 实现某抽象基类),**必须**给一段"如何扩展"——最小实现模板 + 关键约束。

```python
# 最小可用的自定义 XXX 模板
class MyXxx(BaseXxx):
    def required_method(self, ...):
        ...
```

约束清单(从代码校验逻辑读出来的,例):
- 必须实现 `required_method`,否则 ...
- 同步 + 异步两个版本都要实现(如适用)
- 注册方式: ...

> 为什么强制: 二次开发者最需要的就是"我怎么加自己的东西"。只讲内部实现不讲扩展点,文档对二开者价值减半。

## Configuration(如适用)

| Config | 默认值 | 含义 | 影响 | Source |
|---|---|---|---|---|
| ... | ... | ... | ... | [...](...) |

含**安全相关**配置项时单独标注(如 `allow_host_bash` 这类放开隔离的开关)。

## Common Pitfalls / 实战 Tips(可选)

只写你真的从代码 / 注释 / TODO 里读出来的:
- ❌ 不要凭空想"用户可能会怎样错"
- ✅ 如果代码里有 `# WARNING: ...`,翻译成 tip

## References

- [file_a:line](url) — 一句话说这文件干嘛
- [file_b:line](url) — ...
- [file_c:line](url) — ...

(至少 5 个不同文件)

## Related Pages

| Page | Relationship |
|---|---|
| [<Title>](<slug>.md) | 本章引用其 X / 被其 Y 引用 |
| ... | ... |
```

### 第 4 步:自检 checklist

methodology.md §8 那 11 条,全部通过才能保存:
- [ ] 3 条本章目标
- [ ] TL;DR ≤ 5 句
- [ ] Overview 讲 WHY
- [ ] 3+ Mermaid,2+ 类型
- [ ] 每 Mermaid 有 Sources 注释
- [ ] 5+ 文件被引用
- [ ] 引用格式正确(完整 github URL + #L42)
- [ ] 表格用 Source 列(如有代码组件)
- [ ] Related Pages 2+ 链接
- [ ] 没有臆测词(应该 / 大概 / 可能 / 估计)
- [ ] 中文自然不机翻

任何一条不过 → 改完再保存。

### 第 5 步:写盘

```
<repo>/.claude-wiki/versions/<timestamp>/<slug>.md
```

完成后用一句话报告给用户:
```
✓ <章节编号>. <title> 写完 (X.X KB, N 个 mermaid, M 个文件引用)
```

## 几个反模式(出现重写)

### 反模式 1: 把所有源码贴一遍
❌ 大段大段粘贴源码(超 50 行)
✅ 摘 < 20 行关键片段,旁边写**解读**

### 反模式 2: 罗列式
❌ "这个项目有 A, B, C, D, E, F..."
✅ 解释**为什么是 A B C 不是 X Y Z** + 关系

### 反模式 3: 假深度
❌ 用大量术语让自己看起来很厉害,实际没读代码
✅ **每个论断附行号引用**——读没读过看引用就知道

### 反模式 4: Mermaid 灌水
❌ 一个简单 list 也画 mermaid
✅ Mermaid 用于**多元素 + 关系**情形,纯线性流程用 list 或表格

### 反模式 5: 翻译腔
❌ "这个 module 实现了 authentication 的 functionality"
✅ "这个模块实现鉴权"

## 一个完整范例(节选 Overview 段)

> 下面用一个**通用例子**(数据库连接池)演示 Overview 段该怎么写。你实际写的章节主题不同,但"先讲 WHY"的写法一致。

> 错误版本(扣分项):
> ## Overview
> 连接池是这个项目的核心机制之一。它管理数据库连接,提高性能。系统使用了对象池模式。

> 优秀版本:
> ## Overview
>
> 连接池设计回答一个问题: **如何避免每次查询都新建 / 销毁数据库连接带来的开销**?
>
> 直觉做法是用时建连接、用完关掉,但带来三个问题:
> 1. 性能差(TCP 握手 + 认证每次重来,几十毫秒级开销)
> 2. 资源不可控(高并发下瞬间几千连接,打垮数据库)
> 3. 抖动(连接建立失败时请求直接报错,没有缓冲)
>
> 答案: 预先建一批连接放进"池子",请求来了借一个、用完还回去而不是关掉;池子大小设上限做到资源可控。
>
> [src/db/pool.py:1-30](https://github.com/example/repo/blob/main/src/db/pool.py#L1-L30)
>
> ```mermaid
> sequenceDiagram
>   autonumber
>   participant C as 调用方
>   participant P as ConnectionPool
>   participant DB as 数据库
>   C->>P: acquire()
>   alt 池中有空闲连接
>     P-->>C: 复用现有连接
>   else 池未满
>     P->>DB: 新建连接
>     DB-->>P: 连接就绪
>     P-->>C: 返回新连接
>   else 池已满
>     P-->>C: 阻塞等待 / 超时
>   end
>   C->>P: release() 归还连接
> ```
> <!-- Sources: src/db/pool.py:42, src/db/pool.py:88-110 -->

注意优秀版本的特点:
1. 开头**直接提出问题**(为什么需要连接池)
2. **列出直觉方案的问题**(说服读者为啥要这个抽象)
3. **给出答案 + 行号引用**
4. **配一个清晰的图**,覆盖了三个分支(复用 / 新建 / 阻塞)
5. **图后附 Sources 注释**指明根据哪些文件画的

这才是"first principles 深度",不是堆术语。**这个例子只是演示写法,跟你要写的章节无关——你必须基于你实际读到的源码写。**
