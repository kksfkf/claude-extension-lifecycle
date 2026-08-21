# claude-extension-lifecycle

**AI 工具扩展生命周期管理规范 / AI Tool Extension Lifecycle Specification**

扩展是租客，用户是房东。租客搬走，房间要清空——像从未存在过。
The tenant leaves, the room must be empty — as if they were never there.

一套通用的清理准则，覆盖 Claude Code、Cursor、Windsurf、继续等 AI 编程助手的扩展安装与卸载全流程。
A universal cleanup specification covering extension install and uninstall flows for Claude Code, Cursor, Windsurf, 继续, and other AI programming assistants.

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](./LICENSE)
[![Claude Code](https://img.shields.io/badge/Claude_Code-compatible-blue)](https://claude.ai/code)

## 覆盖范围 / Coverage

| 扩展类型 / Extension Type | 清理层次 / Cleanup Layers |
|---|---|
| MCP Server | 配置注册 + 数据目录 + 状态计数 |
| Skill | 技能目录 + 使用计数 + 文档引用 |
| Hook | 钩子脚本 + 配置文件条目 |
| Plugin | 插件目录 + 自带 skill/hook + 状态文件 |

## 使用方式 / Usage

### 方式一：直接安装到 Claude Code / Install directly to Claude Code

```bash
mkdir -p ~/.claude/skills/extension-lifecycle
curl -sL https://raw.githubusercontent.com/kksfkf/claude-extension-lifecycle/main/skills/extension-lifecycle/SKILL.md \
  -o ~/.claude/skills/extension-lifecycle/SKILL.md
```

安装后在会话中输入 `/extension-lifecycle` 即可触发。
After install, type `/extension-lifecycle` in chat to trigger.

### 方式二：加入项目 CLAUDE.md / Add to project CLAUDE.md

将 SKILL.md 内容合并到项目的 `.claude/CLAUDE.md` 中，每次新会话自动生效。
Merge SKILL.md content into `.claude/CLAUDE.md` for automatic loading on every new session.

### 方式三：手动执行清理 / Manual execution

打开对应的 skill 文件，按照八步流程逐条执行。
Open the skill file and follow the 8-step flow manually.

## 八步流程 / 8-Step Flow

```
① 卸载包   ② 清配置   ③ 清数据   ④ 清状态
      ↓                                                              ↓
⑧ 验证   ←  ⑦ 清引用   ←  ⑥ 清备份   ←  ⑤ 清副本
```

详见 [SKILL.md](skills/extension-lifecycle/SKILL.md)（含完整中英文说明）。
See [SKILL.md](skills/extension-lifecycle/SKILL.md) for the full bilingual specification.

## 设计理念 / Design Principles

- **不问用户** — 清理是确定性的，不需要确认 / No user confirmation — cleanup is deterministic
- **只删扩展的痕迹** — 用户自己的 memory / research 文档不在清理范围内 / Only delete extension traces — user's own memory/research stays
- **八步闭环** — 从卸载到验证，每步有明确的检查点 / 8-step closed loop — each step has a clear verification point
- **通用性** — 适用于任何具有多层架构的 AI 编程助手 / Universal — works for any AI assistant with multi-layer architecture

## 来源 / Origin

本规范由用户在实际清理 Claude Code 扩展（ruflo、claude-flow、camofox-browser、weixin-platform-setup、herdr 等）过程中逐步提炼而成，经过多轮审阅和实地验证。
Refined from real-world cleanup of Claude Code extensions (ruflo, claude-flow, camofox-browser, weixin-platform-setup, herdr, etc.), verified through multiple review cycles.

## License

MIT
