# Obsidian Memory Skill

基于 Obsidian vault 的跨会话、跨 Agent 长期记忆协议，适用于 Codex/GPT、Claude Code 和 OpenCode。

## 文件

- `SKILL.md`：原始记忆 skill，包含开场读取、记忆写入、会话日志、新项目接入和恢复协议。
- `templates/AGENTS.md`：vault 根规约模板；Claude Code 可使用内容相同的 `CLAUDE.md`。

## 安装

Codex（Linux / WSL）：

```bash
git clone https://github.com/zyKrip/obsidianMemory.git ~/.codex/skills/obsidian-memory
```

Claude Code（Linux / WSL）：

```bash
git clone https://github.com/zyKrip/obsidianMemory.git ~/.claude/skills/obsidian-memory
```

其他 Agent 可将 `SKILL.md` 安装到其支持的 skill 目录，或在项目规约中引用该协议。

## 配置 vault

原始 skill 的默认 vault 为 `D:\code obstain\代码源`，对应 WSL 路径 `/mnt/d/code obstain/代码源`。使用其他 vault 时，应调整 skill 中的默认路径、安装软链和恢复命令中的路径。

在 vault 中建立以下结构；已有文件保留原内容：

```text
<vault>/
├── AGENTS.md                  # 使用 templates/AGENTS.md 的规约
├── CLAUDE.md                  # 使用 Claude Code 时与 AGENTS.md 内容一致
└── agent/
    ├── MEMORY.md              # 按项目组织的记忆索引
    ├── memories/              # 单条长期记忆正文
    ├── projects/              # 项目档案
    ├── skills/
    │   └── obsidian-memory/
    │       └── SKILL.md        # skill 正本
    └── logs/
        └── daily_log.jsonl    # 只追加的会话日志
```

将仓库中的 `SKILL.md` 保存到 vault 的 `agent/skills/obsidian-memory/`；本地 Agent 的 skill 目录可改为指向该目录的软链或 Windows junction。具体恢复命令见 `SKILL.md`。

在需要接入记忆中心的项目根创建 `.obsidian-memory.json`，替换为实际 vault 路径和项目名：

```json
{
  "vault": "/mnt/d/your-vault",
  "vault_windows": "D:\\your-vault",
  "project": "your-project",
  "onboarded": "2026-10-12",
  "entry": "agent/MEMORY.md"
}
```

完整接入步骤见 `SKILL.md` 第 5 节。已有 vault 根规约时，合并模板中的协议段；记忆索引与日志始终只追加。

## 记忆规则

- vault 内的记忆正文是唯一正本，本地索引或缓存不能替代它。
- 开场读取索引、最近 20 条日志和当前项目档案；其他记忆按需检索。
- 写入前搜索已有主题，更新同一篇笔记，避免重复。
- `MEMORY.md` 和 `daily_log.jsonl` 只追加，过时记忆保留并标记。
- `agent/` 以外的用户笔记默认只读。
- Claude Code 本地 auto-memory 仅保留索引；Codex 直接使用 vault。

本仓库提供 skill 与规约模板，不包含个人记忆正文、项目档案或会话日志。
