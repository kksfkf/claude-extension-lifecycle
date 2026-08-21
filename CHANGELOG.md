# Change Log

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/),
and this project adheres to [Semantic Versioning](https://semver.org/).

---

## [1.0.0] - 2026-08-21

### Added
- 8-step cleanup flow specification for MCP Server, Skill, Hook, Plugin
- Bilingual support (English + Chinese) across all documents
- Extension trace map: config / data / state layers for each extension type
- Cross-tool copy cleanup rules (cc-switch, hermes, Cursor, Windsurf, etc.)
- Memory frontmatter-safe deletion rules (整条删除，不拆 YAML)
- Verification checklist with 7 validation points
- Installation guide (3 methods: direct install, CLAUDE.md merge, manual)
- `.claude-plugin/` manifests for marketplace distribution
- `marketplace/skill.json` for mcpmarket-style publishing

### Origin
Refined from real-world cleanup of extensions: ruflo, claude-flow, camofox-browser, weixin-platform-setup, herdr across Claude Code, cc-switch, and hermes.

### Scope
Covers all four extension types for any AI programming assistant with multi-layer architecture.

---

## [Unreleased]

### Planned
- [ ] Add path mappings for Cursor / Windsurf / 继续 after practical installation
- [ ] Add automated cleanup script (bash) as optional tool
- [ ] Add integration tests against common extension patterns
