---
name: extension-lifecycle
description: Clean up any AI tool extension (MCP Server, Skill, Hook, Plugin) when uninstalling. Use the 8-step cleanup flow to remove all traces — config entries, runtime data, state records, cross-tool copies, backups, and stale references. Runs fully automatically without asking the user. / 卸载任何 AI 工具扩展（MCP Server、Skill、Hook、Plugin）时执行清理。通过八步流程移除所有痕迹——配置项、运行时数据、状态记录、跨工具副本、备份和过期引用。全自动执行，不询问用户。
---

# Extension Lifecycle Cleanup / 扩展生命周期清理规范

**Core principle / 核心理念：** The tenant leaves, the room must be empty — as if they were never there.
**核心原则：** 扩展是租客，用户是房东。租客搬走，房间要清空——像从未存在过。

---

## What each extension type leaves behind / 各类型扩展留下的痕迹

| Type / 类型 | Config layer / 配置层 | Data layer / 数据层 | State layer / 状态层 |
|---|---|---|---|
| **MCP Server** | `settings.json` → `mcpServers.<name>` | `~/<server-name>/`, `~/.cache/<server-name>/` | `.claude.json` → `mcpServers` |
| **Skill** | None / 无（不写配置） | `~/.claude/skills/<name>/` | `.claude.json` → `skillUsage.<name>` |
| **Skill (source dir)** | None / 无（不写配置） | `~/.agents/skills/<name>/`, other skill source dirs | None / 无（不写状态） |
| **Hook** | `settings.json` → `hooks.*` / `statusLine` | `~/.claude/hooks/<name>.sh`, `.claude/hooks/` | Stale script paths in config / 配置中引用的脚本路径失效 |
| **Plugin** | `settings.json` + `.claude.json` | `~/.claude/plugins/<name>/`, bundled skill/hook dirs | `.claude.json` → `pluginUsage.<name>` |

Installation is always three steps: **place files → write config → record state**. Cleanup is the exact reverse.
安装时系统只做三件事：**放文件 → 写配置 → 记状态**。清理就是这三个操作的精确逆向。

---

## The 8-Step Cleanup Flow / 清理八步流程

```
① Uninstall   ② Clear Config   ③ Clear Data   ④ Clear State
      ↓                                                              ↓
⑧ Verify   ←  ⑦ Clear References   ←  ⑥ Clear Backups   ←  ⑤ Clear Cross-Tool Copies
```

Each step does one thing. Fully automatic. No user confirmation required.
每一步只做一件事。全自动执行，不询问用户。

---

### Step 1: Uninstall Package / 卸载包

Use the official uninstall command. Also clean the npx cache (`~/.npm/_npx/` or `%APPDATA%\npm-cache\_npx\`).
使用官方命令：`npm uninstall -g <pkg>` / `pip uninstall <pkg>` / `code --uninstall-extension <id>` / `/plugin uninstall <name>`。同时清理 npx 缓存（`~/.npm/_npx/` 中属于该包的部分）。

**If the extension is a pure skill (no package), skip this step and proceed from Step 2.**
**纯 skill（无 npm/pip 包）时**，跳过此步，直接从②开始。

---

### Step 2: Clear Configuration / 清配置

Delete only the target fields. Do not touch anything else.
只删目标字段，不动其他内容。

- **MCP Server** → Delete `settings.json` → `mcpServers.<name>` entirely / 删除 `settings.json` → `mcpServers.<name>` 整条字段
- **Skill / Hook** → Delete matching entries from `hooks.*` array; keep the field if it becomes `[]`, keep the file if only `env` remains / 从 `hooks.*` 数组中删除匹配该扩展的条目；hooks 字段变为空数组 `[]` 则保留字段，整个文件因此只剩 `env` 字段则保留文件
- **Plugin** → Same as above, also clear `statusLine` / 同上，同时清理 `statusLine`

**Risk:** If a field value accidentally contains the extension name as a substring, verify before deleting to avoid collateral damage.
字段值中巧合包含扩展名的情况需先验证再删，避免误伤其他配置。

---

### Step 2.5: Pre-Cleanup Scan / 扩展扫描（预清理发现）

Before beginning the cleanup, run a full scan to identify all traces of the extension.
在正式清理之前，先执行一次全量扫描，识别所有与该扩展相关的痕迹。
这是解决"未来扩展写到未知位置"的关键步骤——规范无法穷举所有路径，扫描兜底。

**Scan commands / 扫描命令：**

```bash
find ~ -maxdepth 3 -name "<ext-name>" 2>/dev/null
find ~ -maxdepth 3 -name "*<ext-name>*" 2>/dev/null
find ~/.claude -maxdepth 3 -name "*<ext-name>*" 2>/dev/null
find ~ -maxdepth 2 -name ".<ext-name>*" 2>/dev/null
```

**Checklist / 检查列表（逐项确认）：**

| Check item / 检查项 | Method / 方法 | Note / 注意 |
|---|---|---|
| Official plugin dir / 官方插件目录 | `ls ~/.claude/plugins/<name>/` | Covered by spec |
| Skill directory / Skill 目录 | `ls ~/.claude/skills/<name>/` | Covered by spec |
| **Skill source dir / Skill 源目录** | `ls ~/.agents/skills/<name>/` | New: symlink targets may reside here |
| Hook scripts / Hook 脚本 | `ls ~/.claude/hooks/<name>.sh` | Covered by spec |
| Helper scripts / 辅助脚本 | `ls ~/.claude/scripts/<name>.*` | Covered by spec |
| Dot-prefix dirs / 点前缀目录 | `ls ~/.<name>*` | Covered by spec |
| Sibling dirs / 平级目录 | `ls ~ | grep <name>` | Covered by spec |
| **Extension registry** | Check `~/.claude/extension-registry.json` | Optional: unified tracking |
| State file / 状态文件 | `~/.claude.json` → `skillUsage` / `pluginUsage` | Covered by spec |

**Scan principles / 扫描原则：**
- Scanning is not guessing — for each discovered path, verify it actually belongs to the extension
- Criteria: extension name appears in file/dir name, or the extension install created the path
- Cannot confirm ownership → do not delete, mark for review
- Newly discovered paths → sync update to the tool mapping table in this spec

---

### Step 3: Clear Data / 清数据

Delete named-matching entries in the following locations:
删除以下位置中命名匹配扩展名的条目：

```
~/<pkg-name>/                              # User-directory extension data
~/.<pkg-name>*/                            # Dot-prefix naming convention
project-root/.<pkg-name>*/                 # Project-level runtime data (e.g. .swarm/, .claude-flow/)
~/.claude/scripts/<name>.{sh,py,js}        # Extension helper scripts
~/.claude/plugins/<name>/                  # Official plugin install dir
~/.claude/hooks/<name>.sh                  # Hook scripts
~/.claude/helpers/                         # Legacy extension helper dir
```

If no clear ownership can be established → do not delete.
找不到明确归属的 → 不删。

---

### Step 4: Clear State / 清状态

Delete invalid entries in the state file (`.claude.json` or equivalent).
精确实删除状态文件（`.claude.json` 或其等效文件）中以下字段里的无效条目：

- `projects` — entries for non-existent disk paths
- `mcpServers` — expired MCP registrations
- `pluginUsage` / `skillUsage` — entries pointing to uninstalled extensions
- `enabledMcpjsonServers` / `disabledMcpjsonServers` — deleted servers

**Backup the state file before editing. Validate JSON format after.**
编辑前备份原文件，编辑后验证 JSON 格式合法。

---

### Step 5: Clear Cross-Tool Copies / 清副本

Search and delete the extension's files from other AI tools using the same naming rule. Do not touch other extensions' data.
对其他 AI 工具（cc-switch / hermes / Cursor / Windsurf 等），用相同命名规则搜索并清除该扩展的文件。其他扩展的数据不动。

---

### Step 6: Clear Backups / 清备份

Delete files known to be generated by the cleanup process (`.bak`, `.backup.*`, `.old`). Mark unknown-origin files for retention; never delete silently.
删除能确定是清理工具产生的 `.bak` / `.backup.*` / `.old` 文件。无法确认来源的标注保留，不静默删除。

---

### Step 7: Clear References / 清引用

Search for the extension name in documentation and delete matching entries or paragraphs.
在文档中搜索扩展名，删除匹配的条目或段落：

```
~/.claude/memory/                          # Global memory
~/.claude/projects/<proj>/memory/          # Project-level memory
~/.claude/projects/<proj>/CLAUDE.md        # Project-level rules
~/.claude/CLAUDE.md                        # Global rules
~/.claude/plans/                           # Completed plans no longer referenced
```

**Processing rules / 处理规则：**
- Memory entries → delete the entire entry (do not break frontmatter, which would corrupt YAML structure)
- CLAUDE.md → delete paragraphs explicitly about the extension, keep the rest
- User research notes → delete mentions of the extension, keep research conclusions
- This spec document itself → keep (self-reference)
- Plans still in use → keep
- **Extension-derived behavior rules → delete the entire memory entry, even if the extension name is not mentioned in the text**
- **Embedded skill content blocks in CLAUDE.md → delete the entire block** (example: unlazy, adversarial-verify skill instruction texts were directly embedded in CLAUDE.md as installation traces; removal requires deleting both the trigger map rows and the full embedded content block, not just the map rows).

---

### Step 8: Verify / 验证

| Check / 检查项 | Method / 方法 | Expected / 期望 |
|---|---|---|
| Package uninstalled / 包已卸载 | `npm list -g | grep <pkg>` | No output / 无输出 |
| npx cache cleaned / npx 缓存已清理 | `find ~/.npm/_npx/ -name "*<pkg>*"` | No match / 无匹配 |
| Config has no references / 配置无引用 | `grep -r <pkg> ~/.claude/settings.json` | No match / 无匹配 |
| Data directory gone / 数据目录无残留 | `ls ~/<pkg-name>/ 2>&1` | Does not exist / 不存在 |
| State file valid / 状态文件合法 | JSON parse check | No error / 无报错 |
| Plugin dir gone / 插件目录无残留 | `ls ~/.claude/plugins/<pkg-name>/ 2>&1` | Does not exist / 不存在 |
| Valid state entries / 状态条目有效 | Check all pluginUsage/skillUsage entries point to existing dirs | No stale entries / 无 stale 条目 |
| Projects synced / 项目目录同步 | `ls ~/.claude/projects/` vs `.claude.json` → `projects` keys | No orphaned directories / 无孤儿目录 |
| CLAUDE.md clean / CLAUDE.md 干净 | `grep -n "<ext-name>" ~/.claude/CLAUDE.md` | No remaining references (including embedded blocks) / 无残留引用（含内嵌内容块） |
| Clean startup / 启动无报错 | Restart tool | No hook/MCP errors / 无 hook/MCP 报错 |

All checks pass → cleanup complete.
全部通过 → 清理完成。

---

## Trigger Conditions / 触发条件

| Signal / 触发信号 | Description / 说明 | Action / 执行动作 |
|---|---|---|
| User says "uninstall/remove XX" / 用户说"卸载/移除 XX" | Explicit instruction / 明确指令 | Full 8-step flow + Step 2.5 scan |
| Extension install complete / 扩展安装完成 | AI agent wrote config/directories | Update extension registry |
| Session startup anomaly / 会话启动发现异常 | Health check detects orphaned entries | Report first, ask before cleaning |
| Tool startup error / 工具启动报错 | Hook/MCP failure, path not found | Locate extension via error, execute cleanup |

**Do NOT proactively scan in these cases / 禁止主动扫描的情况：**
- User has not mentioned any extension name → do not run Step 2.5 full scan
- User has not requested cleanup → do not delete any extension files
- Health check only reports, does not handle → let user confirm before executing

---

## Health Check (Session Startup) / 健康检查（会话启动自动运行）

Run this lightweight check on every new session. Reports only; never deletes.
每次新会话启动时，自动执行以下轻量检查（不删除，只报告）：

```bash
python -c "
import json, os
path = os.path.expanduser('~/.claude.json')
with open(path) as f:
    data = json.load(f)
issues = []
for field in ['skillUsage', 'pluginUsage']:
    for key, val in data.get(field, {}).items():
        name = key.split('@')[0].split(':')[0]
        paths = [
            os.path.expanduser(f'~/.claude/skills/{name}'),
            os.path.expanduser(f'~/.claude/plugins/{name}'),
            os.path.expanduser(f'~/.agents/skills/{name}'),
        ]
        if not any(os.path.exists(p) for p in paths):
            issues.append(f'{field}.{key} 指向的目录不存在')
# Also check projects: disk dirs with no corresponding state entry
projects_dir = os.path.expanduser('~/.claude/projects/')
if os.path.isdir(projects_dir):
    disk_projects = set(os.listdir(projects_dir))
    state_projects = set(data.get('projects', {}).keys())
    for dp in disk_projects:
        dp_norm = os.path.join(projects_dir, dp)
        if not any(dp_norm in sp or sp in dp_norm for sp in state_projects):
            issues.append(f'project {dp}: directory exists on disk but not in .claude.json projects')
if issues:
    print('ISSUES:', '; '.join(issues))
else:
    print('OK')
"
```

**Output interpretation / 输出解读：**
- `OK` → No anomalies, proceed normally / 无异常，正常进入会话
- `ISSUES: ...` → Report to user, let them decide whether to clean / 报告问题，由用户决定是否清理，不自动删除

---

## Extension Registry (Optional) / 扩展注册表（推荐机制）

**Registry path / 注册表路径：** `~/.claude/extension-registry.json`

**Format / 格式：**

```json
{
  "registeredAt": "2026-08-21T10:00:00.000Z",
  "extensions": {
    "<ext-name>": {
      "type": "skill | mcp | hook | plugin",
      "installedAt": "2026-08-21T10:00:00.000Z",
      "paths": {
        "skills": ["~/.claude/skills/<ext-name>"],
        "source": ["~/.agents/skills/<ext-name>"],
        "hooks": [],
        "scripts": [],
        "data": []
      },
      "manifest": "https://github.com/user/<ext-name>"
    }
  }
}
```

**Registration timing / 注册时机：**
- On extension install → the install script or AI agent writes the entry
- On manual install → register after configuration is complete

**On cleanup / 清理时：**
- Read the registry, verify each registered path still exists
- Paths in the registry → must be cleaned
- Paths found by scan but not in registry → verify via Step 2.5, then handle
- After cleanup completes → remove the extension entry from the registry

---

## What is NOT part of extension installation / 什么不属于扩展安装痕迹

- Official tool baseline config (env, permissions defaults)
- Other extensions' independent data

---

## Tool Mapping / 工具映射

| Concept / 概念 | Claude Code |
|---|---|
| Config directory / 配置目录 | `~/.claude/` |
| State file / 状态文件 | `~/.claude.json` |
| Global settings / 全局设置 | `~/.claude/settings.json` |
| Skill directory / Skill 目录 | `~/.claude/skills/<name>/` |
| **Skill source dir / Skill 源目录** | **`~/.agents/skills/<name>/`** |
| Plugin directory / Plugin 目录 | `~/.claude/plugins/<name>/` |
| Hook script / Hook 脚本 | `~/.claude/hooks/<name>.sh` |
| Helper scripts / 辅助脚本 | `~/.claude/scripts/` |
| Extension registry / 扩展注册表 | `~/.claude/extension-registry.json` |
| Metadata / 元数据 | `.claude.json` → `skillUsage` / `pluginUsage` / `mcpServers` |
| Cross-tool copies / 跨工具副本 | `~/.cc-switch/`, `~/.hermes/`, etc. |

Other tools (Cursor / Windsurf / 继续): paths to be added after practical installation.
其他工具（Cursor / Windsurf / 继续）路径待实际安装后补充。

---

## Scope / 适用范围

**Applies to:** MCP server, Skill, Hook, Plugin — four extension types; multi-tool concurrent scenarios.
**适用：** MCP server、Skill、Hook、Plugin 四类扩展；多工具并发使用场景。

**Does not apply to:** Pure CLI stateless tools; one-shot containerized Agents.
**不适用：** 纯 CLI 无状态工具；一次性容器 Agent。

**Local installation sync:** If you installed this spec locally (into CLAUDE.md or as a skill), check for updates periodically and re-apply when changed. The spec is the single source of truth — local copies must stay in sync.
**本地安装同步：** 若将本规范安装到本地（写入 CLAUDE.md 或作为 skill），请定期检查更新并在有变更时重新应用。规范是单一事实来源，本地副本需保持同步。

**Relationship with official mechanisms:** Official handles automation; this spec fills gaps. They run in parallel, no conflict.
**与官方机制的关系：** 官方负责自动化部分，本规范做补充和兜底。两者并行不悖。

**Extension registry mechanism:** Applies to manually installed or registry-supported extensions; existing extensions are covered by the Step 2.5 scan mechanism. The path mapping table will continue to expand as more tools are validated.
**扩展注册表机制：** 适用于手动安装或支持注册的扩展；现有扩展通过 ②.5 扫描机制覆盖。随着更多工具验证，路径映射表将持续扩展。
