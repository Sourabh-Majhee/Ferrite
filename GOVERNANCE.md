# Governance

This document describes how the Ferrite project is governed, including decision-making processes, roles, and dispute resolution.

## Mission

Ferrite's mission is to create a production-quality, memory-safe terminal browser written entirely in Rust with no external engine dependencies. The project serves three communities:

1. **Users** — Those who need to browse the web in terminal environments
2. **Contributors** — Those who want to help build a terminal browser
3. **Researchers** — Those studying web standards, performance, or security

## Project Structure

### Benevolent Dictator Model (Current)

Ferrite is currently governed by a **Benevolent Dictator for Life (BDFL)** — the original project creator who makes final decisions on disputes and direction.

**Current BDFL:** [Project Creator Name]

This model allows rapid decision-making in early development. As the project matures, governance may evolve to a Steering Committee.

### Decision-Making Process

#### Minor Decisions (No Discussion Needed)

- Bug fixes that don't change architecture
- Documentation improvements
- Test additions
- Refactoring that doesn't affect public APIs

**Process:** Submit PR, maintainers review and merge

#### Medium Decisions (Discussion in PR)

- New features that fit the roadmap
- Minor API changes
- Dependencies additions
- Performance optimizations

**Process:**
1. Describe change in PR description
2. Discuss in comments (24-48 hours)
3. Maintainers approve or request changes
4. Merge if consensus reached

#### Major Decisions (Issue Discussion + ADR Required)

- Architecture changes
- New subsystems
- Major dependency additions/removals
- Roadmap changes
- Breaking API changes

**Process:**
1. Open GitHub issue describing the proposal
2. Discuss for 1-2 weeks
3. Reach consensus or BDFL decides
4. Submit PR with Architecture Decision Record (ADR)
5. Merge after maintainer review

#### Strategic Decisions (Community Consultation)

- Project mission changes
- Governance changes
- Long-term vision/roadmap

**Process:**
1. BDFL opens discussion on GitHub
2. Community votes if requested (informal)
3. BDFL makes final decision
4. Document decision in governance update

### Roles and Responsibilities

#### Benevolent Dictator (BDFL)

- **Responsibility:** Project direction, strategic decisions, dispute resolution
- **Authority:** Final say on any project decision
- **Term:** Indefinite (unless voluntarily stepped down)
- **Succession:** Documented in advance

#### Steering Committee (Future)

When the project reaches production status, a Steering Committee may be established:

- 3-5 members representing different project areas
- Elected annually by contributors
- Makes decisions by consensus or majority vote
- BDFL retains veto power

#### Maintainers

- **Responsibility:** Code review, PR approval, issue triage
- **Authority:** Approve PRs, manage branches, deploy releases
- **Commitment:** Regular engagement (8+ hours/week expected)
- **Recruitment:** Nominated by existing maintainers, approved by BDFL

#### Contributors

- **Responsibility:** Submit PRs, report issues, review code
- **Authority:** Participate in discussions, propose changes
- **Commitment:** No minimum; as much as you want

#### Users

- **Responsibility:** Report bugs, request features, provide feedback
- **Authority:** Vote on feature importance (informal surveys)
- **Commitment:** Share your experience with the project

### Maintainer Selection

New maintainers are identified and selected based on:

1. **Consistent contributions** — 20+ PRs or equivalent involvement
2. **Code quality** — Demonstrated understanding of architecture
3. **Community respect** — Positive interactions with other contributors
4. **Project alignment** — Shared vision and values
5. **Time commitment** — Able to dedicate regular time

**Selection process:**
1. Existing maintainers discuss candidates
2. Nominee agrees to responsibility
3. BDFL approves
4. Announcement to community

### Conflict Resolution

#### Level 1: Discussion

Most conflicts can be resolved through respectful discussion. We ask participants to:

- Assume good intent
- Ask clarifying questions
- Propose solutions, not just criticisms
- Refer to facts and evidence
- Be patient (others may be in different time zones)

#### Level 2: Mediation

If discussion doesn't resolve the conflict:

1. Either party requests mediation
2. A neutral maintainer facilitates discussion
3. Both parties propose solutions
4. Mediator recommends approach

#### Level 3: BDFL Decision

If mediation fails:

1. Case presented to BDFL in writing
2. BDFL hears both sides
3. BDFL makes binding decision
4. Decision is documented

#### Appeals Process

If someone disagrees with a BDFL decision:

1. Request reconsideration within 2 weeks
2. Provide new information or reasoning
3. BDFL may reconsider or uphold decision
4. Final decision is binding

### Code Review Standards

All code changes must be reviewed by at least one maintainer before merging. Reviewers check for:

- **Correctness** — Does the code do what it's supposed to?
- **Quality** — Is it well-written and maintainable?
- **Testing** — Are there adequate tests?
- **Documentation** — Is it documented?
- **Style** — Does it follow project conventions?
- **Architecture** — Does it fit the layered architecture?

Reviewers should:

- Be constructive and respectful
- Explain "why" not just "what"
- Approve promptly (24-48 hours)
- Ask questions if unclear

### Consensus vs. Voting

The project prefers **consensus over voting**:

- Aim for decisions everyone can support
- If consensus can't be reached, BDFL decides
- Voting is used only in community surveys (non-binding)

### Transparency

The project values transparency:

- ✅ Decisions are documented (in ADRs, issues, or governance)
- ✅ Discussions happen in public (GitHub, not private)
- ✅ Roadmap is publicly visible
- ✅ Meeting notes are shared
- ✅ Code of Conduct violations are handled confidentially

Exceptions:
- ❌ Security vulnerabilities (until patched)
- ❌ Sensitive personal information
- ❌ Legal matters

### Transition to Steering Committee

When conditions are met, the project will transition to a Steering Committee model:

**Conditions:**
- 50+ active contributors
- 5+ maintainers
- Stable API (v1.0+)
- Community size >1000

**Process:**
1. Community discussion (2-4 weeks)
2. Draft Steering Committee charter
3. Vote on new governance (simple majority)
4. Elect first committee (if approved)
5. Update documentation

**Steering Committee Charter** (to be drafted):
- Committee composition and roles
- Decision-making process
- Election process and terms
- Powers and limitations
- BDFL role in new structure

## Community Values

The Ferrite project values:

- **Inclusivity** — Welcome everyone, regardless of background
- **Respect** — Treat others with dignity and courtesy
- **Collaboration** — Work together toward common goals
- **Transparency** — Be open about decisions and reasoning
- **Excellence** — Strive for high-quality code and processes
- **Fun** — Building software should be enjoyable
- **Patience** — Remember that others are volunteers

See [CODE_OF_CONDUCT.md](CODE_OF_CONDUCT.md) for details.

## Policy Changes

To propose changes to this governance document:

1. Open a GitHub issue with your proposal
2. Discuss with community (1-2 weeks)
3. Reach consensus or BDFL decides
4. Update document if approved

Major changes should be voted on by the community.

## Contact

### For Questions
- Email: governance@ferrite.dev
- GitHub Discussions: [Link]

### For Code of Conduct Violations
- Email: conduct@ferrite.dev (confidential)

### For Disputes
- Email: disputes@ferrite.dev

## Document History

| Version | Date | Changes |
|---------|------|---------|
| 1.0 | 2026-05-24 | Initial governance document |

---

**Last Updated:** May 24, 2026

This governance model may evolve as the project grows. We remain committed to transparency, inclusivity, and community-driven development.
