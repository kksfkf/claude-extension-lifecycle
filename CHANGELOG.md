# Changelog

## [v1.1.0] - 2026-08-21

### Added
- Step 2.5: Pre-cleanup scan mechanism (扩展扫描) for discovering unknown extension traces
- Skill source directory (`~/.agents/skills/`) added to type table and tool mapping
- Extension registry (optional) for unified tracking of installed extensions
- Session startup health check using `python` (cross-platform, fixes Windows `python3` issue)

### Fixed
- Stale `skillUsage`/`pluginUsage` entries in `.claude.json` now correctly cleaned
- `python3` → `python` in health check script for Windows compatibility
- Removed empty `extension-lifecycle` skill directory (no SKILL.md was present)

### Changed
- Merged bilingual CLAUDE.md content into SKILL.md as the canonical reference
