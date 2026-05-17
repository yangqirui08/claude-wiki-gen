# 触发指令:部分运行(Partial Run)

> **用法**:已经跑过 full-run 但被中断 / 只想补几章 / 迭代调 prompt 后想重跑某几章 时用。

---你的任务---

你是资深技术文档工程师。任务是为**当前工作目录的代码仓库**生成 **指定的几个章节** 的中文 wiki。

## 必读资料(按顺序)

1. `D:\Projects\源码分析工具\methodology.md`
2. `D:\Projects\源码分析工具\prompts\02-chapter-writing.md`(主要)
3. `D:\Projects\源码分析工具\prompts\03-quality-check.md`

(不需要读 01-toc,目录已经存在;不需要读 04-cross-ref,partial 不动 Related Pages 全局,只补本批章节内部的)

## 必填变量(我会告诉你)

- `TARGET_CHAPTERS`: 要生成的章节 slug 列表,例:`["10-zhong-jian-jian-lian-ji-zhi", "14-sha-xiang-jia-gou-yu-shi-xian", "19-chang-qi-ji-yi-ji-zhi"]`
- `WIKI_DIR`: 已有 wiki 目录路径,例:`./.claude-wiki/versions/2026-05-15-153000/`
- `REPO_URL`: 源仓库 URL
- `BRANCH`: 源仓库分支
- `OVERWRITE`: `true` / `false`(默认 false,已存在的章节跳过,除非用户要求重写)

## 执行流程

```
1. 读 WIKI_DIR/wiki.json,验证 TARGET_CHAPTERS 都在 pages[] 里
2. 对每个 chapter:
   a. 找到对应 pages[] 条目,拿到 title / scope_hint
   b. 检查 WIKI_DIR/<slug>.md 是否已存在
      - 存在 + OVERWRITE=false → 跳过,报告 "⊘ <slug> 已存在,跳过"
      - 存在 + OVERWRITE=true  → 备份为 <slug>.md.bak,继续
      - 不存在 → 继续
   c. 按 prompts/02 跑该章
   d. 按 prompts/03 自查
   e. 保存,报告 "✓ <slug> 完成 (X.X KB, N mermaid, M refs)"
3. 报告本批所有章节状态
```

## 重要约束

- **不动 wiki.json**(目录已确定)
- **不动其他章节的 Related Pages**(不属于本批 scope)
- 本批章节之间的 Related Pages 互引可以补

## 启动(脚本式,不等确认)

读完 methodology + prompts/02 + prompts/03 后,**直接开始执行**,不要回复确认、不要等待。第一条输出可简短说明"准备生成 X 章到 <WIKI_DIR>",随后立即开干。中途不停,跑完报告结果。

---结束---
