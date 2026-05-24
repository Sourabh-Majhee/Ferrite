# ADR-0003: Layer-Based Architecture with No Up-Dependencies

## Status
**Accepted**

## Context

Ferrite implements a complex system with many interconnected components: networking, parsing, DOM management, CSS cascade, layout, JavaScript execution, and rendering. As the codebase grows, we need a clear architectural pattern to prevent:

- Circular dependencies
- Tangled module imports
- Difficulty in testing individual components
- Making changes in one layer that break others
- Cognitive overhead for contributors working on specific subsystems

The project is organized as a Cargo workspace with separate crates for each layer. However, without clear dependency rules, crates can grow into a tangled mess.

## Decision

Ferrite adopts a **strict layered architecture** with the following rules:

### Layer Stack (Top to Bottom)

```
Layer 7: ferrite-app        (Application shell, event loop, UI)
Layer 6: ferrite-render     (Rendering, ANSI output)
Layer 5: ferrite-js         (JavaScript engine, DOM bindings)
Layer 4: ferrite-layout     (Box tree, text layout, positioning)
Layer 3: ferrite-css        (CSS engine, cascade, computed styles)
Layer 2: ferrite-dom        (DOM tree, nodes, mutation API)
Layer 1: ferrite-html       (HTML parsing)
Layer 0: ferrite-net        (HTTP, TLS, caching)
```

### Dependency Rules

1. **Downward dependencies only** — Layer N may depend on any Layer 0 through N-1, but never on Layer N+1 or higher
2. **No circular dependencies** — No crate may depend on another crate that transitively depends back on it
3. **Minimal dependency surface** — Each crate should depend only on the minimum set of lower layers it needs
4. **Public vs. private API** — Crates should use Rust's visibility rules (pub, pub(crate), etc.) to control what other layers can access

### Dependency Matrix

| Layer | May Depend On |
|-------|---------------|
| ferrite-app (7) | 6, 5, 4, 3, 2, 1, 0 (any) |
| ferrite-render (6) | 5, 4, 3, 2, 1, 0 |
| ferrite-js (5) | 4, 3, 2, 1, 0 |
| ferrite-layout (4) | 3, 2, 1, 0 |
| ferrite-css (3) | 2, 1, 0 |
| ferrite-dom (2) | 1, 0 |
| ferrite-html (1) | 0 |
| ferrite-net (0) | (no Ferrite crates) |

### Violation Example (❌ Not Allowed)

```rust
// ferrite-css/src/lib.rs
use ferrite_layout::LayoutBox;  // ❌ VIOLATION: CSS depends on Layout

// Why? CSS is computed per-element. Layout uses the CSS to build boxes.
// CSS should never need to know about Layout internals.
```

### Correct Pattern (✅ Allowed)

```rust
// ferrite-layout/src/lib.rs
use ferrite_css::ComputedStyle;  // ✅ OK: Layout depends on CSS results
use ferrite_dom::NodeId;         // ✅ OK: Layout depends on DOM

// Layout reads CSS properties and DOM structure to build box tree.
```

## Communication Between Layers

When layers need to communicate, use these patterns:

### Pattern 1: Data Flows Downward (Common)
Lower layers produce data that higher layers consume.

```rust
// ferrite-html produces DOM
let root = ferrite_html::parse(html_source);  // → NodeId

// ferrite-css reads DOM and produces styles
let styles = ferrite_css::compute_styles(&dom, &style_rules);  // StyleMap

// ferrite-layout reads both and produces layout
let layout = ferrite_layout::compute_layout(&dom, &styles);  // LayoutTree
```

### Pattern 2: Callbacks / Traits (When Upward Communication Needed)

If a lower layer must communicate with a higher layer, use a trait that the higher layer implements:

```rust
// ferrite-dom defines what "rendering" looks like (trait, not implementation)
pub trait Renderer {
    fn paint(&self, node: NodeId, box_: &Box);
}

// ferrite-render implements the trait
impl Renderer for TerminalPainter {
    fn paint(&self, node: NodeId, box_: &Box) {
        // Actual rendering logic
    }
}

// ferrite-dom knows about the trait, but not the implementation
pub fn render_tree(renderer: &dyn Renderer, root: NodeId) {
    // Uses the trait
}
```

### Pattern 3: Event Systems (For Complex Flows)

When many layers need to react to the same event, use an event bus:

```rust
// ferrite-app dispatches an event
event_bus.emit(Event::PageLoaded { root_node });

// ferrite-css listens and computes styles
css_subscriber.on_page_loaded(root_node);

// ferrite-layout listens and computes layout
layout_subscriber.on_page_loaded(root_node);

// ferrite-render listens and repaints
render_subscriber.on_page_loaded(root_node);
```

## Testing

The layered architecture enables isolated testing:

```rust
// Test ferrite-css in isolation, with mock DOM
#[test]
fn test_cascade_specificity() {
    let css = parse_css("p { color: blue; } .highlight { color: red; }");
    let dom = create_mock_dom();
    let styles = compute_styles(&dom, &css);
    assert_eq!(styles[p_with_class_highlight].color, Color::Red);
}

// Test ferrite-layout, with real DOM and CSS but mock renderer
#[test]
fn test_flex_layout() {
    let dom = parse_html(...);
    let styles = compute_styles(&dom, ...);
    let layout = compute_layout(&dom, &styles);
    assert_eq!(layout.root.width, expected_width);
}
```

Each layer can be tested without instantiating the entire browser.

## Consequences

### ✅ Positive
- **Clear responsibility** — Each crate has one reason to change
- **Easy onboarding** — New contributors understand where to make changes
- **Testability** — Each layer can be tested independently
- **Refactoring safety** — Changes to Layer 3 don't affect Layer 1
- **Parallel development** — Teams can work on different layers simultaneously
- **Code review** — Simpler to review changes when dependencies are clear

### ❌ Negative
- **Slightly more abstraction** — Some elegant designs might require indirection
- **Performance concerns** — Could introduce abstraction layers that add latency
- **Restrictive during design** — May prevent elegant cross-cutting concerns
- **Requires discipline** — Developers must remember the rules

### ⚠️ Neutral
- **Learning curve** — Contributors need to understand the layer model
- **Documentation overhead** — Must document why certain designs are chosen

## Exceptions

Exceptions to the layered architecture require:
1. **Written justification** in the code as a comment
2. **ADR discussion** (create a new ADR)
3. **Maintainer approval** before merging

Example:

```rust
// EXCEPTION: ferrite-js depends on ferrite-render to implement toDataURL()
// Justification: Browsers require this API; no other way to implement without exposing render internals
// ADR: Link to discussion
use ferrite_render::FrameBuffer;
```

## Monitoring

The CI/CD pipeline will check for violations:

```bash
# Check for disallowed dependencies
cargo tree --all | grep "layer violation"  # (custom script)
```

Pull requests that introduce upward dependencies will be rejected with a link to this ADR.

## References

- [Software Architecture Patterns: Layered Pattern](https://www.oreilly.com/library/view/software-architecture-patterns/9781491971437/ch01.html)
- [Clean Architecture - Robert C. Martin](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html)
- [Rust Book: Module System](https://doc.rust-lang.org/book/ch07-00-managing-growing-projects-with-packages-modules-and-paths.html)
- [Ferrite Architecture Document](../ARCHITECTURE.md)
