---
name: obsidian-memory
description: Obsidian 记忆中心协议。适用于 GPT/Codex、Claude Code、OpenCode 等 Agent 在本 vault 中读写长期记忆、追加会话日志、新项目接入记忆中心。凡项目根存在 .obsidian-memory.json 或当前工作区就是本 vault 即适用。
---

# Obsidian 记忆中心 skill

跨会话、跨 Agent 的长期记忆**唯一正本**是本 Obsidian vault：`D:\code obstain\代码源`（WSL 路径：`/mnt/d/code obstain/代码源`）。
权威地址优先读项目根 `.obsidian-memory.json` 的 `vault` / `vault_windows` 字段；若不存在，则使用当前 vault 根。完整规约见 vault 根 `AGENTS.md`（与 `CLAUDE.md` 同文）——本 skill 是其可执行摘要，两者冲突以 vault 根规约为准。

## 0. GPT/Codex 适配原则

1. runtime 名统一写实际执行环境：Codex/GPT 写 `codex`，Claude Code 写 `claude-code`，OpenCode 写 `opencode`。
2. Codex 没有 Claude 本地 auto-memory 层；正文与索引都只写本 vault。只有在 **Claude Code** 中才同步写 `~/.claude/projects/*/memory/MEMORY.md` 的索引行。
3. Windows 下优先使用 `D:\code obstain\代码源\...` 路径；WSL/Linux 下使用 `/mnt/d/code obstain/代码源/...`。路径包含空格，命令必须加引号或用 LiteralPath。
4. 若沙箱或权限限制导致无法追加日志/记忆，先按当前 Agent 的权限审批流程请求授权；不要改写为覆盖写入，也不要绕开 vault 正本。
5. 对用户笔记保持克制：`agent/` 之外默认只读，除非用户当次明确要求修改。

## 1. 开场协议（做任何任务之前）

1. 读 `<vault>/agent/MEMORY.md` 全文；
2. 读 `<vault>/agent/logs/daily_log.jsonl` 末尾 20 行；
3. 若当前项目根有 `.obsidian-memory.json`，读取其中 `project` 字段，并读 `<vault>/agent/projects/<项目名>.md`；
4. 若当前工作区就是本 vault 且没有 `.obsidian-memory.json`，按任务涉及对象选择项目档案；只改记忆系统本身时，可不强行读取业务项目档案。

## 2. 过程中：写记忆

触发条件：用户偏好、架构决策、踩过的坑、被纠正的做法、环境/工具恢复方式、跨 Agent 需要复用的流程——任何值得跨会话记住的事，**立即**写，不等收尾。

步骤：

1. 先搜 `<vault>/agent/memories/` 是否已有同主题笔记——有就更新那一篇并刷新其 `date`，不造重复；
2. 新建/更新笔记，**文件名 = frontmatter `name` = kebab-case**，前缀标类型 `project-` / `feedback-` / `reference-` / `user-`：

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

3. 在 vault `agent/MEMORY.md` 对应项目小节末尾追加一行索引：
   `- [标题](memories/文件名.md) — 一句话钩子 (日期)`
4. **仅 Claude Code**：在本地 auto-memory 的 `MEMORY.md`（`~/.claude/projects/*/memory/MEMORY.md`）追加同一行索引（注明 vault 笔记名）。**正文只写 vault，本地不建正文文件。**

## 3. GPT/Codex 文件操作建议

1. 查找优先用 `rg` / `rg --files`；Windows PowerShell 读中文文件时显式指定 UTF-8。
2. JSONL 与 MEMORY.md 只能 append。Codex 可用 `apply_patch` 修改单条记忆正文；追加 JSONL/MEMORY.md 时要确保是新增行，不重排旧行。
3. 写 JSONL 前先构造单行合法 JSON，`summary` 简短写事实与结论，`refs` 写相关记忆笔记名数组；不要写 Markdown、换行或尾逗号。
4. 需要记录“本次改了 skill/协议”时，优先更新已有 `reference-obsidian-memory-skill.md`，不要新建重复主题记忆。

## 4. 收尾协议（结束回合前）

向 `<vault>/agent/logs/daily_log.jsonl` 追加一行（`agent` 字段写自己的 runtime 名，如 `codex` / `claude-code` / `opencode`）：

```json
{"ts":"<ISO时间>","project":"<项目名>","agent":"<runtime名>","type":"session","summary":"<本次做了什么、结论>","refs":["<相关记忆笔记名>"]}
```

- `type` 词表：`session` | `decision` | `learning` | `bugfix`。
- 只改本 vault/记忆系统时，`project` 可写 `agent-memory`。

## 5. 新项目接入

1. 在项目根写 `.obsidian-memory.json`：`{"vault": "/mnt/d/code obstain/代码源", "project": "<项目名>", "onboarded": "<YYYY-MM-DD>", "entry": "agent/MEMORY.md", "vault_windows": "D:\\code obstain\\代码源"}`；
2. 在 `<vault>/agent/projects/` 新建 `<项目名>.md` 项目档案（路径/技术栈/架构要点/大事记）；
3. 在 vault `agent/MEMORY.md` 新建该项目小节，写一条 `project-<项目名>-onboarded` 记忆；
4. 在项目 `AGENTS.md` / `CLAUDE.md` 写入记忆协议段。Claude Code 项目保留 `CLAUDE.md`；GPT/Codex 项目优先读 `AGENTS.md`。

## 铁律

1. JSONL 与 MEMORY.md **只追加，绝不覆盖/重写/删行**；
2. 过时记忆在原笔记 frontmatter 标 `status: outdated`，索引行保留，不删除；
3. 用户笔记（`agent/` 之外的目录）一律只读，除非用户当次明确要求修改；
4. `agent/.backup-*/` 为重构前快照，只读；
5. 本 vault 是唯一正本。任何 Agent 的本地缓存、软链、摘要都不能替代 vault 内正文。

## 本 skill 的容灾

本文件正本在 vault `agent/skills/obsidian-memory/SKILL.md`。Claude Code 可把 `~/.claude/skills/obsidian-memory` 设为指向它的软链；GPT/Codex 不依赖该软链。

若 Claude 软链丢失（`~/.claude` 被清），重新执行：
`ln -s "/mnt/d/code obstain/代码源/agent/skills/obsidian-memory" ~/.claude/skills/obsidian-memory`
