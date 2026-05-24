# Contributing to Ferrite

Thank you for your interest in contributing to Ferrite! We welcome contributions of all kinds — code, documentation, bug reports, feature requests, and more.

## Code of Conduct

Please read and adhere to our [CODE_OF_CONDUCT.md](CODE_OF_CONDUCT.md). We are committed to providing a welcoming and inclusive environment for all contributors.

## How to Contribute

### Reporting Bugs

Before creating a bug report, please search existing issues to avoid duplicates.

**When reporting a bug, include:**
- A clear, descriptive title
- A detailed description of the behavior you observed
- Steps to reproduce the issue
- Expected behavior
- Actual behavior
- Your environment (OS, Rust version, terminal type, Ferrite version)
- Screenshots or terminal recordings if applicable

### Suggesting Features

We love feature requests! Please:
- Search existing issues first to check if it's already been requested
- Describe the feature clearly and explain why it would be useful
- Provide examples of how it would be used
- Mention any related standards (HTML5 spec, CSS spec, etc.)

### Contributing Code

#### Getting Started

1. **Fork the repository** on GitHub
2. **Clone your fork** locally:
   ```bash
   git clone https://github.com/your-username/ferrite.git
   cd ferrite
   ```
3. **Add the upstream remote**:
   ```bash
   git remote add upstream https://github.com/original-owner/ferrite.git
   ```
4. **Create a feature branch**:
   ```bash
   git checkout -b feature/my-feature
   # or for bug fixes
   git checkout -b fix/issue-number
   ```

#### Development Setup

```bash
# Build the project
cargo build

# Run all tests
cargo test --workspace

# Run tests for a specific crate
cargo test -p ferrite-css

# Build documentation
cargo doc --workspace --open

# Check code with clippy
cargo clippy --workspace

# Format code
cargo fmt --all
```

#### Code Style & Standards

- **Format**: Run `cargo fmt` before committing. The project uses Rust's standard formatting.
- **Linting**: Run `cargo clippy` and fix any warnings. We aim for zero clippy warnings.
- **Comments**: Include comments for non-obvious logic. Reference specification sections where applicable:
  ```rust
  // Per WHATWG HTML spec § 12.2.4.1:
  // The script processing algorithm must be run ...
  fn process_script() {
      // implementation
  }
  ```
- **Documentation**: Public APIs must have doc comments:
  ```rust
  /// Parses the HTML source and returns a DOM tree.
  ///
  /// # Arguments
  /// * `source` - The HTML source code to parse
  ///
  /// # Returns
  /// A `Result` containing the root `NodeId` or a parse error.
  pub fn parse(source: &str) -> Result<NodeId, ParseError> {
  ```
- **Testing**: All code changes must include tests. See the Testing section below.

#### Commit Guidelines

- Write clear, descriptive commit messages
- Use imperative mood ("Add feature" not "Added feature")
- Reference issues when applicable: "Fix #123"
- Keep commits atomic — one logical change per commit
- Example:
  ```
  Add support for CSS Grid layout

  Implement the CSS Grid layout algorithm per CSS Grid Level 1 spec.
  Handles grid-template-columns, grid-template-rows, and grid placement.

  Fixes #456
  ```

#### Pull Request Process

1. **Keep it focused**: One feature or fix per PR. Large changes should be discussed in an issue first.
2. **Update documentation**: Update README.md, ARCHITECTURE.md, or docs/ if your changes warrant it.
3. **Add tests**: All PRs must include tests (see Testing section below).
4. **Rebase on main**: Before submitting, ensure your branch is up to date:
   ```bash
   git fetch upstream
   git rebase upstream/main
   ```
5. **Push to your fork**:
   ```bash
   git push origin feature/my-feature
   ```
6. **Create a PR** on GitHub with:
   - A clear title and description
   - Reference to any related issues
   - Screenshots or examples if applicable
   - Checklist:
     - [ ] Tests added/updated
     - [ ] Documentation updated
     - [ ] `cargo fmt` and `cargo clippy` pass
     - [ ] All tests pass locally

#### Review Process

- At least one maintainer will review your PR
- We may request changes or ask questions
- Once approved, a maintainer will merge your PR
- Maintainers may make minor adjustments before merging

### Contributing Documentation

Documentation is just as important as code!

- **README updates**: Changes to usage or setup instructions
- **Architecture decisions**: Significant design changes should include an ADR in `docs/decisions/`
- **Specs**: Annotated references to standards in `docs/specs/`
- **Comments**: Inline documentation with spec references

#### Writing an Architecture Decision Record (ADR)

ADRs document significant technical decisions. Create a new file in `docs/decisions/` with the format `NNNN-short-title.md`:

```markdown
# ADR-0001: Use Arena Allocation for DOM Tree

## Status
Accepted

## Context
The DOM tree requires parent-child bidirectional references, creating cycle problems for Rust's ownership system.

## Decision
We will use an arena allocator pattern where all DOM nodes are stored in a single `Vec<NodeData>` and referenced by typed indices (`NodeId`).

## Consequences
- ✅ Eliminates Rust borrow checker issues
- ✅ O(1) node allocation
- ✅ O(1) document cleanup (drop the arena)
- ❌ Cannot hold direct Rust references to nodes
- ❌ Requires bounds checking on access

## References
- [bumpalo](https://docs.rs/bumpalo/) crate for arena allocation patterns
```

## Testing

Tests are mandatory for all code changes. Here's how to write tests for Ferrite:

### Unit Tests

Add tests in the same file as the code:

```rust
#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_selector_matching() {
        let dom = parse_html("<div class=\"test\"></div>");
        let matches = select(".test", &dom);
        assert_eq!(matches.len(), 1);
    }
}
```

### Integration Tests

Place integration tests in `tests/integration/`:

```rust
// tests/integration/layout_test.rs
use ferrite_layout::*;
use ferrite_dom::*;

#[test]
fn test_flex_layout() {
    let html = r#"
        <div style="display: flex; width: 20; height: 10;">
            <div style="flex: 1;"></div>
            <div style="flex: 1;"></div>
        </div>
    "#;
    let layout = compute_layout(html);
    assert_eq!(layout.children[0].width, 10);
    assert_eq!(layout.children[1].width, 10);
}
```

### Fixture Files

Place static HTML/CSS test files in `tests/fixtures/`:

```
tests/fixtures/
├── simple.html
├── table.html
├── flexbox.html
└── grid.html
```

### Running Tests

```bash
# All tests
cargo test --workspace

# Specific crate
cargo test -p ferrite-css

# Specific test
cargo test test_selector_matching

# Show output
cargo test -- --nocapture

# All tests including ignored
cargo test -- --include-ignored
```

### Test Coverage

While we don't have a strict coverage requirement, aim for:
- **Unit tests**: 80%+ for parsing and algorithm code
- **Integration tests**: Critical paths and public APIs
- **Fixtures**: One fixture per major feature

## Architecture Guidelines

Before making large architectural changes, please:

1. **Open an issue** to discuss the design
2. **Reference the spec** — justify decisions with standards
3. **Consider layering** — each crate should have a single responsibility
4. **Avoid up-dependencies** — Layer N should never depend on Layer N+1
5. **Write an ADR** if the change affects multiple crates

## Performance Considerations

- Use `cargo bench` to profile changes:
  ```bash
  cargo bench --package ferrite-layout
  ```
- Avoid allocations in hot paths (parsing, rendering)
- Use arena allocation for bulk operations
- Cache computed values when safe

## Documentation Building

```bash
# Build and open documentation
cargo doc --workspace --open

# Build with private items included
cargo doc --workspace --open --document-private-items
```

## Dependency Policy

We aim to minimize dependencies while using battle-tested, well-maintained crates:

✅ **Encouraged**:
- `rustls` (TLS, pure Rust)
- `html5ever` (HTML parser from Servo)
- `boa_engine` (JavaScript, pure Rust)
- `taffy` (Layout algorithm, pure Rust)

❌ **Discouraged**:
- C/C++ FFI crates (use pure Rust alternatives)
- Deprecated or unmaintained crates
- Crates that violate the "Pure Rust" philosophy

When adding a dependency:
1. Ensure it's actively maintained
2. Check if a pure-Rust alternative exists
3. Update `Cargo.toml` with comments explaining why
4. Document the choice in an ADR if it's significant

## Release Process

The project maintainers handle releases, but here's the process:

1. **Update version** in all `Cargo.toml` files
2. **Update CHANGELOG.md** with notable changes
3. **Create a git tag**: `git tag v0.1.0`
4. **Push the tag**: `git push origin v0.1.0`
5. **Create GitHub release** with CHANGELOG excerpt
6. **Publish to crates.io**: `cargo publish`

## Getting Help

- **Questions?** Open a discussion on GitHub Discussions or ask in an issue
- **Stuck?** Feel free to ask for help — we're here to support you
- **Mentorship**: First-time contributors are welcome; we're happy to guide you

## Maintainers

The project is maintained by [list of maintainers]. For questions about the project's direction, contact the maintainers.

## Thank You!

We appreciate every contribution, no matter how small. Thank you for helping make Ferrite better!
