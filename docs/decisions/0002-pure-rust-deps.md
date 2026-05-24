# ADR-0002: Pure Rust Dependency Policy

## Status
**Accepted**

## Context

Ferrite's core principle is "zero foreign engine dependencies." However, we also need to be pragmatic about dependency selection. The Rust ecosystem has evolved to provide high-quality, battle-tested implementations of critical components:

- `html5ever` — Mozilla's production HTML5 parser from Servo
- `cssparser` — Servo's CSS tokenizer and parser
- `boa_engine` — A JavaScript engine with 94%+ ECMAScript compliance
- `taffy` — Flexbox and CSS Grid layout algorithms
- `ratatui` — Terminal UI framework

Early in the project, building all of these from scratch would be years of work with limited benefit. Using existing crates accelerates development while we focus on the browser shell and integration layer.

However, we need clear guidelines for:
- When to depend on external crates
- When to implement functionality ourselves
- When to eventually replace dependencies with first-party code
- How to handle dependencies that violate our philosophy

## Decision

Ferrite adopts a **phased dependency policy**:

### Phase 1: Pragmatic Dependencies (Current)
Use production-quality Rust crates for algorithmic heavy-lifting:
- ✅ `html5ever` for HTML parsing
- ✅ `cssparser` for CSS tokenization
- ✅ `boa_engine` for JavaScript execution
- ✅ `taffy` for Flexbox/Grid layout
- ✅ `rustls` for TLS (not OpenSSL)
- ✅ `tokio` for async runtime
- ✅ `ratatui` for terminal UI primitives

These dependencies must be:
- Written entirely in Rust (no FFI to C/C++)
- Actively maintained with regular updates
- Well-tested and production-ready
- Focused on a single domain
- Licensed under permissive open-source licenses

### Phase 2: Transition to First-Party (Future)
Once the browser stabilizes, begin replacing dependencies with native implementations:
- [ ] `ferrite-html` replaces `html5ever`
- [ ] `ferrite-css` replaces `cssparser`
- [ ] `ferrite-js` replaces `boa_engine`
- [ ] `ferrite-layout` replaces `taffy`

### Phase 3: Self-Contained (Long-term Goal)
The browser operates with minimal external dependencies. Only `tokio` and `ratatui` remain external to preserve our focus.

### Explicit Rejections
Never depend on:
- ❌ C/C++ libraries via FFI (libssl, libcurl, libxml2, SpiderMonkey, V8, WebKit)
- ❌ Subprocess execution of external browsers (Firefox, Chrome)
- ❌ JavaScript runtimes (Node.js, Deno, Bun)
- ❌ Unmaintained crates or crates with known security vulnerabilities
- ❌ Crates that violate Ferrite's architectural principles (e.g., pulling in massive dependency trees)

## Rationale

### Why Use Dependencies Now?

1. **Accelerated MVP** — HTML/CSS/JS parsing are 80/20 efforts; we benefit from proven implementations
2. **Quality assurance** — Battle-tested crates have found and fixed edge cases we'd rediscover slowly
3. **Spec compliance** — `html5ever` passes all WHATWG HTML5 tests; replicating this would take months
4. **Community focus** — Our value is in integration and the browser shell, not re-implementing standards
5. **Iterative learning** — Using `boa_engine` teaches us about JavaScript VMs before we build our own

### Why Plan to Replace Them?

1. **Control** — We understand every line of critical code
2. **Optimization** — First-party code optimized specifically for terminal rendering
3. **Philosophy** — "Pure Rust" means we own our entire stack
4. **Licensing** — Eliminates any dependency-chain licensing concerns
5. **Performance** — Remove FFI boundaries and tailored for our use case

### Why Not Replace Immediately?

1. **Resource constraint** — Build the browser first, optimize later
2. **Premature optimization** — Not yet clear which layers are bottlenecks
3. **Bug discovery** — First find pain points using existing code
4. **Incremental correctness** — Get the browser working before reimplementing components

## Consequences

### ✅ Positive
- Faster MVP development
- Higher quality initial implementation
- Benefit from ecosystem expertise
- Focus limited resources on integration
- Clear upgrade path over time

### ❌ Negative
- Dependency on external maintainers for critical code
- If a dependency becomes unmaintained, we must fork or replace it
- Adds startup latency (though minimal for CLI tools)
- Requires careful tracking of when to transition

### ⚠️ Neutral
- Philosophy purists may object; philosophy is a long-term goal, not a hard constraint
- Potential licensing complexity if dependencies change licenses

## Adding New Dependencies

Before adding any new dependency to Ferrite:

1. **Propose in an issue** — Discuss the dependency with maintainers
2. **Check alternatives** — Is there a pure-Rust alternative? If yes, prefer it
3. **Evaluate quality** — Is it actively maintained? Well-tested? Does it have any known security issues?
4. **Minimize scope** — Do we need the entire crate or just a small part? (If just a small part, consider vendoring or reimplementing)
5. **Document in Cargo.toml** — Add a comment explaining the dependency:
   ```toml
   [dependencies]
   # html5ever: HTML5 parsing from Mozilla's Servo project
   # Alternative: Could reimplement, but current implementation is WHATWG-compliant and battle-tested
   # Replacement timeline: Phase 2 (post-MVP)
   html5ever = "0.26"
   ```
6. **Update this ADR** if the dependency is a critical path component

## Monitoring Dependencies

The project maintainers will:
- Monitor for new CVEs in dependencies
- Track maintenance status of dependencies
- Plan replacement timeline for critical dependencies
- Evaluate new versions for breaking changes

## References

- [Ferrite Pure Rust Philosophy](../ARCHITECTURE.md#3-what-pure-rust-actually-means)
- [Cargo dependency guidelines](https://doc.rust-lang.org/cargo/guide/dependencies.html)
- [Semantic Versioning](https://semver.org/)
- [NIST Software Composition Analysis](https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-53A-2-related-info.pdf)
