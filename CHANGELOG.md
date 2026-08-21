# Changelog

## [v1.2.0] - 2026-08-22

### Added
- Step 7: Embedded skill content blocks in CLAUDE.md must be deleted as a unit (trigger map rows + full content block), not just the map rows
- Step 8: New verification rows — valid state entries, projects sync, CLAUDE.md cleanliness
- Health check: orphaned project directory detection (disk has dir but .claude.json projects has no entry)

### Fixed
- CLAUDE.md cleanup now correctly removes full embedded skill blocks (unlazy, adversarial-verify) instead of only the trigger map entries
- Health check detects orphaned project directories on disk that are missing from .claude.json

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
