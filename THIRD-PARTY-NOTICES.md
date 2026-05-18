# 第三方来源与许可声明(Third-Party Notices)

本项目「源码分析工具 / Claude-Wiki-Gen」是一个**衍生作品(derivative work)**:其方法论与 prompt 架构改编自下列第三方项目。本文件按各上游许可要求保留其版权与许可声明。本项目自身的许可见根目录 [`LICENSE`](LICENSE)(MIT,© 2026 Qirui),仅覆盖本项目的原创部分。

---

## 1. 主要来源:Microsoft deep-wiki(直接改编自此)

- 项目:`microsoft/skills` 仓库内的 deep-wiki 插件
- 来源:https://github.com/microsoft/skills/tree/main/.github/plugins/deep-wiki
- 许可:**MIT License**(2026-05-18 核实:`microsoft/skills` 仓库 LICENSE 为 MIT,版权人 Microsoft Corporation;插件 README 亦自述 MIT)
- 改编范围(详见 [`methodology-sources/microsoft-deep-wiki-notes.md`](methodology-sources/microsoft-deep-wiki-notes.md)):10 条核心原则、行号引用格式、Mermaid 规范、章节结构、5 条 NEGOTIABLE 深度铁律等,经中文化、Zread schema 对齐、Claude Code 适配后改编使用。

依据 MIT 许可,以下为微软原始版权与许可声明,原文保留:

```
MIT License

Copyright (c) Microsoft Corporation.

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```

## 2. 间接来源(致谢)

Microsoft deep-wiki 插件 README 自述其 prompt 架构 "Distilled from" 以下两个前辈项目。本项目并未直接复制此二者代码,但其方法论思想经微软蒸馏后间接影响本项目,在此一并致谢:

- OpenDeepWiki — https://github.com/AIDotNet/OpenDeepWiki
- deepwiki-open — https://github.com/AsyncFuncAI/deepwiki-open

(此二项目各自有其独立许可,如需直接复用其代码请另行查阅其 LICENSE。)

---

> 维护提示:若上游 `microsoft/skills` 许可或来源发生变化,需同步更新本文件。核实方法:`curl https://raw.githubusercontent.com/microsoft/skills/main/LICENSE`。
