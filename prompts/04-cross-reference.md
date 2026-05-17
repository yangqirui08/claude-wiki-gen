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

## 链接格式规范

### 同目录(都在 .claude-wiki/versions/<X>/ 里)
```markdown
[中间件链机制](./10-zhong-jian-jian-lian-ji-zhi.md)
```

### 锚到具体小节
```markdown
[中间件链机制 §before/after hook](./10-zhong-jian-jian-lian-ji-zhi.md#beforeafter-hook)
```

### 不要写空标题
❌ `[10-zhong-jian-jian-lian-ji-zhi.md](./10-...)`
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
