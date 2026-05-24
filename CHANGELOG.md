# Changelog

All notable changes to Ferrite will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/), and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added
- Initial project setup and architecture documentation
- Core crate structure for all 7 layers
- README and contributing guidelines
- Architecture Decision Records (ADRs) for key design decisions
- Specification references for HTML, CSS, DOM, and JavaScript

### Planned for Phase 1
- HTML5 parser integration via `html5ever`
- DOM tree implementation with arena allocator
- CSS cascade and selector matching engine
- Terminal layout engine (block and inline formatting)
- JavaScript engine integration via `boa_engine`
- Basic terminal rendering with ANSI colors

## [0.0.1] - 2026-05-24

### Added
- Project foundation and architecture documentation
- License files (MIT and Apache 2.0)
- Code of Conduct
- Contributing guidelines
- Architecture Decision Records (ADRs):
  - ADR-0001: Arena allocation for DOM tree
  - ADR-0002: Pure Rust dependency policy
  - ADR-0003: Layered architecture with no up-dependencies
- Specification references for HTML5, CSS, and DOM

### Status
**Alpha** — Pre-release, architecture only. No functionality yet.

---

## Guidelines for Updating This File

### When to Add Entries

- **Added** — New features
- **Changed** — Changes to existing functionality
- **Deprecated** — Features that will be removed
- **Removed** — Removed features
- **Fixed** — Bug fixes
- **Security** — Security vulnerability fixes

### Example Entry

```markdown
## [1.2.0] - 2026-06-15

### Added
- CSS Grid layout support
- `fetch()` API for HTTP requests from JavaScript

### Changed
- HTML parser now validates DOCTYPE more strictly
- Improved color mapping to ANSI palette

### Fixed
- Text wrapping at word boundaries (fixes #142)
- Memory leak in style cache invalidation

### Deprecated
- `--no-colors` flag (use `--colors none` instead)

### Removed
- Workaround for old terminal emulator compatibility (dropped support for xterm < 200)

### Security
- Fixed XSS vulnerability in innerHTML setter (CVE-2026-XXXXX)
```

### Version Format

- **Major** — Breaking changes, major new features
- **Minor** — New features, non-breaking changes
- **Patch** — Bug fixes, documentation

### Date Format

Use ISO 8601: YYYY-MM-DD

---

## Release Schedule

Ferrite follows a phased development approach:

- **Phase 1** (Q3-Q4 2026) — Core browser functionality
- **Phase 2** (Q1-Q2 2027) — Event handling and interactivity
- **Phase 3** (Q3 2027) — Advanced features
- **Phase 4** (2028+) — Next-generation features

See [ARCHITECTURE.md](ARCHITECTURE.md#15-phased-roadmap) for details.

---

## Comparing Versions

- [Unreleased changes](https://github.com/yourusername/ferrite/compare/0.0.1...HEAD)
- [0.0.1 release](https://github.com/yourusername/ferrite/releases/tag/0.0.1)
