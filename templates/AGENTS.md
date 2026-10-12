<!-- obsidian-memory:begin -->
# 本 Vault 是 Agent 记忆中心

任何在此 vault 中工作的 AI Agent（Claude Code / Codex / OpenCode / Cursor / 其他）都必须遵守本规约。本文件以 AGENTS.md 与 CLAUDE.md 两个名字存在于 vault 根部，内容一致。

## 模块地图

| 路径 | 职责 | 写入规则 |
|---|---|---|
| `agent/MEMORY.md` | 记忆索引（按项目分组），会话开场必读 | 索引行追加到对应项目小节末尾；行只增不删 |
| `agent/memories/` | 单条记忆笔记，一事一档，**唯一正本** | 新建文件；同主题已有则更新那一篇；过时标 `status: outdated`，不删除 |
| `agent/logs/daily_log.jsonl` | 会话日志 | **只追加，绝不覆盖**（最重要的护栏） |
| `agent/projects/` | 各接入项目的档案 | 新建或更新 |
| `agent/.backup-*/` | 重构前快照 | 只读 |
| 其余目录 | 用户的笔记 | 未经任务明确要求，不读不写 |

## 记忆笔记规范（agent/memories/*.md）

- **文件名 = frontmatter `name` = kebab-case**，前缀标类型：`project-` / `feedback-` / `reference-` / `user-`
- frontmatter 统一为：

```markdown
---
name: <kebab-case, 与文件名一致>
description: <一句话摘要, 供索引判断相关性, 避免使用冒号>
type: user | feedback | project | reference
project: <来源项目名>
date: <YYYY-MM-DD, 最后一次重大更新>
---

<事实本身。feedback 类要写"为什么"和"下次怎么用"。相关记忆用 [[name]] 互链。>
```

- type 语义：`user`=用户偏好/身份；`feedback`=纠正过的做法与踩坑结论；`project`=项目进展/架构决策；`reference`=环境/工具/外部资源指引。
- JSONL 日志 `type` 词表：`session` | `decision` | `learning` | `bugfix`。

## 双记忆协议（2026-07-07 起）

**本 vault 是唯一正本。** Claude Code 的本地 auto-memory（`~/.claude/projects/*/memory/`）只保留 `MEMORY.md` 索引行（每行一个钩子，指向本 vault 的笔记名），**不再存正文文件**。其他 Agent 无本地层，直接读写 vault。

## 行为规则

1. JSONL 与 MEMORY.md 只追加，绝不重写或删行。
2. 写入记忆前先搜 `memories/` 是否已有同主题笔记——有就更新那一篇（并刷新其 `date`），不造重复。
3. 用户笔记（agent/ 之外）一律视为只读，除非用户当次明确要求修改。
<!-- obsidian-memory:end -->
