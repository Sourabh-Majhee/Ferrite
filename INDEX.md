# Project Documentation Index

Complete index of all markdown documentation files for Ferrite.

## Root-Level Documentation Files

### Core Project Documentation

| File | Purpose | Audience |
|------|---------|----------|
| [README.md](README.md) | Project overview, features, quick start | Everyone |
| [ARCHITECTURE.md](ARCHITECTURE.md) | Complete technical architecture | Developers |
| [CONTRIBUTING.md](CONTRIBUTING.md) | Contribution guidelines and workflow | Contributors |
| [CODE_OF_CONDUCT.md](CODE_OF_CONDUCT.md) | Community standards and behavior | Everyone |

### Project Information

| File | Purpose | Audience |
|------|---------|----------|
| [CHANGELOG.md](CHANGELOG.md) | Version history and release notes | Everyone |
| [GOVERNANCE.md](GOVERNANCE.md) | Project governance and decision-making | Project leaders |
| [SECURITY.md](SECURITY.md) | Security policy and vulnerability reporting | Everyone |
| [FAQ.md](FAQ.md) | Frequently asked questions | Everyone |
| [DOCUMENTATION.md](DOCUMENTATION.md) | Documentation overview | Everyone |

### License Files

| File | License |
|------|---------|
| [LICENSE-MIT](LICENSE-MIT) | MIT License |
| [LICENSE-APACHE](LICENSE-APACHE) | Apache License 2.0 |

---

## docs/decisions/ — Architecture Decision Records

### ADR Index

| ADR | Title | Status | Link |
|-----|-------|--------|------|
| [README](docs/decisions/README.md) | How to write ADRs | — | [View](docs/decisions/README.md) |
| 0001 | Arena Allocation for DOM Tree | Accepted | [View](docs/decisions/0001-arena-allocation.md) |
| 0002 | Pure Rust Dependency Policy | Accepted | [View](docs/decisions/0002-pure-rust-deps.md) |
| 0003 | Layered Architecture with No Up-Dependencies | Accepted | [View](docs/decisions/0003-layered-architecture.md) |

### ADR Details

#### [ADR-0001: Use Arena Allocation for DOM Tree](docs/decisions/0001-arena-allocation.md)
- **Status:** Accepted
- **Context:** Rust ownership issues with bidirectional graph structures
- **Decision:** Use typed indices (NodeId) into a single Vec arena
- **Key Consequences:** O(1) allocation, no ownership issues, cache-friendly

#### [ADR-0002: Pure Rust Dependency Policy](docs/decisions/0002-pure-rust-deps.md)
- **Status:** Accepted
- **Context:** Balancing "pure Rust" philosophy with practical development
- **Decision:** Phased approach: use battle-tested Rust crates now, replace with first-party implementations over time
- **Key Consequences:** Faster MVP, long-term goal of complete Rust implementation

#### [ADR-0003: Layer-Based Architecture with No Up-Dependencies](docs/decisions/0003-layered-architecture.md)
- **Status:** Accepted
- **Context:** Need for clear module organization and independent testing
- **Decision:** 7-layer architecture with strict downward-only dependencies
- **Key Consequences:** Maintainable code, clear separation of concerns, safe refactoring

---

## docs/specs/ — Specification References

### Specification Index

| Spec | Topic | Standards | Link |
|------|-------|-----------|------|
| [README](docs/specs/README.md) | Spec reference guide | — | [View](docs/specs/README.md) |
| HTML5 Parsing | HTML tokenization and tree construction | WHATWG | [View](docs/specs/html5-parsing.md) |
| CSS Cascade | CSS cascade algorithm, specificity, inheritance | W3C | [View](docs/specs/css-cascade.md) |
| CSS Selectors | CSS selector matching | W3C | [View](docs/specs/css-selectors.md) |
| DOM API | DOM node types, element interface, events | WHATWG | [View](docs/specs/dom-api.md) |

### Specification Details

#### [HTML5 Parsing](docs/specs/html5-parsing.md)
- **Coverage:** Tokenization, tree construction, error handling, encoding detection
- **Implementation:** Via `html5ever` (WHATWG-compliant)
- **Status:** ✅ Fully implemented

#### [CSS Cascade](docs/specs/css-cascade.md)
- **Coverage:** Cascade algorithm, specificity, inheritance, computed values
- **Implementation:** `ferrite-css` crate
- **Status:** ✅ Fully implemented

#### [CSS Selectors](docs/specs/css-selectors.md)
- **Coverage:** Type, class, ID, attribute, pseudo-class, pseudo-element selectors
- **Implementation:** `ferrite-css` crate
- **Status:** 🟡 Mostly implemented (some pseudo-classes deferred)

#### [DOM API](docs/specs/dom-api.md)
- **Coverage:** Node interface, Element interface, Document, events, collections
- **Implementation:** `ferrite-dom` crate
- **Status:** 🟡 Partially implemented (core features working)

---

## Quick Navigation by Purpose

### 🎯 Getting Started
1. [README.md](README.md) — Project overview
2. [CONTRIBUTING.md](CONTRIBUTING.md) — How to contribute
3. [docs/decisions/README.md](docs/decisions/README.md) — Architecture decisions

### 📚 Learning the Architecture
1. [ARCHITECTURE.md](ARCHITECTURE.md) — Complete architecture overview
2. [docs/decisions/](docs/decisions/) — Design rationale
3. [docs/specs/](docs/specs/) — Implementation standards

### 👥 Community Participation
1. [CODE_OF_CONDUCT.md](CODE_OF_CONDUCT.md) — Community standards
2. [CONTRIBUTING.md](CONTRIBUTING.md) — Contribution process
3. [GOVERNANCE.md](GOVERNANCE.md) — Project governance

### 🔒 Security and Safety
1. [SECURITY.md](SECURITY.md) — Security policy
2. [CODE_OF_CONDUCT.md](CODE_OF_CONDUCT.md) — Safe community
3. [GOVERNANCE.md](GOVERNANCE.md) — Conflict resolution

### ❓ Questions and Answers
1. [FAQ.md](FAQ.md) — Frequently asked questions
2. [CONTRIBUTING.md](CONTRIBUTING.md) — Contribution FAQ
3. [GOVERNANCE.md](GOVERNANCE.md) — Governance FAQ

### 📋 Tracking Progress
1. [CHANGELOG.md](CHANGELOG.md) — Version history
2. [ARCHITECTURE.md#15-phased-roadmap](ARCHITECTURE.md#15-phased-roadmap) — Roadmap
3. [docs/decisions/](docs/decisions/) — Design decisions

---

## Search Reference

### By Topic

**HTML Parsing**
- [ARCHITECTURE.md#7-layer-1--html5-parser](ARCHITECTURE.md#7-layer-1--html5-parser)
- [docs/specs/html5-parsing.md](docs/specs/html5-parsing.md)

**CSS**
- [ARCHITECTURE.md#9-layer-3--css-engine](ARCHITECTURE.md#9-layer-3--css-engine)
- [docs/specs/css-cascade.md](docs/specs/css-cascade.md)
- [docs/specs/css-selectors.md](docs/specs/css-selectors.md)

**DOM**
- [ARCHITECTURE.md#8-layer-2--dom-tree](ARCHITECTURE.md#8-layer-2--dom-tree)
- [docs/specs/dom-api.md](docs/specs/dom-api.md)

**Layout**
- [ARCHITECTURE.md#10-layer-4--terminal-layout-engine](ARCHITECTURE.md#10-layer-4--terminal-layout-engine)

**JavaScript**
- [ARCHITECTURE.md#11-layer-5--javascript-engine](ARCHITECTURE.md#11-layer-5--javascript-engine)

**Rendering**
- [ARCHITECTURE.md#12-layer-6--terminal-painter](ARCHITECTURE.md#12-layer-6--terminal-painter)

**Application**
- [ARCHITECTURE.md#13-layer-7--event-loop--application-shell](ARCHITECTURE.md#13-layer-7--event-loop--application-shell)

### By Question

**"What is Ferrite?"**
→ [README.md](README.md#ferrite--pure-rust-terminal-browser)

**"How do I get started?"**
→ [README.md#getting-started](README.md#getting-started)

**"How do I contribute?"**
→ [CONTRIBUTING.md](CONTRIBUTING.md)

**"What's the architecture?"**
→ [ARCHITECTURE.md](ARCHITECTURE.md)

**"Why were these design decisions made?"**
→ [docs/decisions/](docs/decisions/)

**"How is CSS cascade implemented?"**
→ [docs/specs/css-cascade.md](docs/specs/css-cascade.md)

**"What are the security implications?"**
→ [SECURITY.md](SECURITY.md)

**"How is the project governed?"**
→ [GOVERNANCE.md](GOVERNANCE.md)

---

## Document Relationships

```
README.md (entry point)
  ├─→ CONTRIBUTING.md (how to help)
  │    ├─→ CODE_OF_CONDUCT.md (community standards)
  │    └─→ ARCHITECTURE.md (what to work on)
  ├─→ ARCHITECTURE.md (what this is)
  │    ├─→ docs/decisions/ (why design this way)
  │    └─→ docs/specs/ (implementation standards)
  ├─→ FAQ.md (common questions)
  ├─→ SECURITY.md (safety)
  └─→ CHANGELOG.md (progress)

GOVERNANCE.md (how decisions are made)
  ├─→ CODE_OF_CONDUCT.md (behavioral expectations)
  └─→ docs/decisions/ (precedent)

DOCUMENTATION.md (this file)
  └─→ All other files (cross-references)
```

---

## Statistics

### Documentation Volume
- **Total Files:** 21
- **Total Words:** ~20,000
- **Markdown Files:** 21

### Breakdown by Category
- **Root documentation:** 10 files
- **Architecture decisions:** 4 files (1 README + 3 ADRs)
- **Specification references:** 5 files (1 README + 4 specs)

### File Sizes (Approximate)
- **README.md:** 2,000 words
- **ARCHITECTURE.md:** 4,000 words (pre-existing)
- **CONTRIBUTING.md:** 2,500 words
- **CODE_OF_CONDUCT.md:** 1,200 words
- **FAQ.md:** 2,500 words
- **GOVERNANCE.md:** 1,800 words
- **SECURITY.md:** 1,500 words
- **CHANGELOG.md:** 800 words
- **ADRs (total):** 4,500 words
- **Specs (total):** 6,000 words

---

## How to Use This Index

### As a New Contributor
1. Start with [README.md](README.md)
2. Read [CONTRIBUTING.md](CONTRIBUTING.md)
3. Review [ARCHITECTURE.md](ARCHITECTURE.md) section relevant to your work
4. Check [docs/specs/](docs/specs/) for implementation standards

### As a Project Maintainer
1. Use [GOVERNANCE.md](GOVERNANCE.md) for decision-making
2. Reference [docs/decisions/](docs/decisions/) for precedent
3. Update [CHANGELOG.md](CHANGELOG.md) with releases
4. Review [FAQ.md](FAQ.md) to identify documentation gaps

### As a User
1. Read [README.md](README.md) for overview
2. Check [FAQ.md](FAQ.md) for questions
3. Review [SECURITY.md](SECURITY.md) for safety
4. Reference [CONTRIBUTING.md](CONTRIBUTING.md) to report issues

### As a Security Researcher
1. Read [SECURITY.md](SECURITY.md) for threat model
2. Review [docs/specs/](docs/specs/) for standards compliance
3. Check [CODE_OF_CONDUCT.md](CODE_OF_CONDUCT.md) for responsible disclosure
4. Contact security@ferrite.dev for vulnerabilities

---

**Last Updated:** May 24, 2026  
**Status:** Complete  
**Maintenance:** See [DOCUMENTATION.md](DOCUMENTATION.md#maintenance-notes) for update schedule
