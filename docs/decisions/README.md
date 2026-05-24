# Architecture Decision Records

This directory contains Architecture Decision Records (ADRs) that document significant technical decisions made in the Ferrite project.

## What is an ADR?

An ADR is a short markdown document that describes a significant architectural decision, its context, alternatives considered, and the consequences of the chosen approach. ADRs are immutable records — they document why a decision was made at a point in time.

## Format

Each ADR is numbered sequentially: `0001-short-title.md`, `0002-another-title.md`, etc.

### Structure

```markdown
# ADR-NNNN: Descriptive Title

## Status
- **Proposed** - Suggested but not yet decided
- **Accepted** - Decision has been made
- **Deprecated** - Superseded by another ADR
- **Rejected** - Decision was not accepted

## Context
Explain the issue or problem that prompted the decision. Include relevant background information and constraints.

## Decision
State the decision that was made, clearly and concisely.

## Rationale
Explain why this decision was made. What alternatives were considered and why were they rejected?

## Consequences
Describe the outcomes and implications:
- ✅ Positive consequences
- ❌ Negative consequences
- ⚠️ Neutral or uncertain consequences

## References
Links to related ADRs, issues, PRs, or external documentation.
```

## Index of ADRs

- [ADR-0001: Use Arena Allocation for DOM Tree](0001-arena-allocation.md)
- [ADR-0002: Pure Rust Dependency Policy](0002-pure-rust-deps.md)
- [ADR-0003: Layer-Based Architecture with No Up-Dependencies](0003-layered-architecture.md)

## How to Create a New ADR

1. Determine the next ADR number (look at the highest numbered file in this directory)
2. Create a new markdown file: `NNNN-descriptive-title.md`
3. Fill in all required sections
4. Submit as a pull request
5. Link to the ADR in the index above once accepted

## Discussion

When proposing an ADR for a significant decision:
1. Open an issue on GitHub describing the problem
2. Discuss with the community and maintainers
3. Once consensus is reached, create the ADR in the PR
4. Update this index once merged

## References

- [Documenting Architecture Decisions - Michael Nygard](https://cognitect.com/blog/2011/11/15/documenting-architecture-decisions)
- [ADR GitHub Organization](https://adr.github.io/)
