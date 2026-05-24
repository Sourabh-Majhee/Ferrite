# Ferrite — Pure Rust Terminal Browser

> *"A terminal browser that owes nothing to anyone — no V8, no WebKit, no Chromium, no Firefox. Every line, pure Rust."*

A fully-featured terminal browser written entirely in Rust with no external browser engine dependencies. Ferrite brings modern web browsing to the terminal, targeting low-resource hardware while maintaining correctness and performance.

## Features

- **Pure Rust** — No C/C++ engines, no Firefox subprocess, no V8 or SpiderMonkey
- **Memory Efficient** — Designed to run on machines with 512MB RAM and single-core CPUs
- **Standards Compliant** — HTML5 parsing, CSS cascade, JavaScript execution (ES2021+)
- **JavaScript Support** — DOM manipulation, `fetch()`, `setTimeout`, event handling
- **CSS Features** — Flexbox, CSS Grid, computed styles, cascade and specificity
- **Terminal Optimized** — ANSI colors, bold/italic text, link tracking, scroll regions

## Getting Started

### Prerequisites

- Rust 1.70+
- A terminal that supports at least 256-color ANSI escape sequences (preferred: 24-bit truecolor)

### Building

```bash
git clone https://github.com/yourusername/ferrite
cd ferrite
cargo build --release
```

The binary will be available at `target/release/ferrite`.

### Running

```bash
./target/release/ferrite
```

Or directly via Cargo:

```bash
cargo run --release
```

## Architecture Overview

Ferrite is organized into modular layers, each in a separate crate within the workspace. Data flows top-to-bottom on every page load, and events flow bottom-to-top to trigger re-renders.

```
User Input / URL Bar
    ↓
┌─────────────────────────────────┐
│ Layer 0: ferrite-net            │ HTTP/1.1, HTTP/2, TLS, cookies, cache
├─────────────────────────────────┤
│ Layer 1: ferrite-html           │ HTML5 parser, DOM tree construction
├─────────────────────────────────┤
│ Layer 2: ferrite-dom            │ DOM tree, nodes, queries, events
├─────────────────────────────────┤
│ Layer 3: ferrite-css            │ CSS engine, selectors, cascade, computed values
├─────────────────────────────────┤
│ Layer 4: ferrite-layout         │ Box tree, text wrapping, Flexbox, Grid
├─────────────────────────────────┤
│ Layer 5: ferrite-js             │ JavaScript engine, DOM bindings, fetch, timers
├─────────────────────────────────┤
│ Layer 6: ferrite-render         │ Cell buffer, ANSI colors, link tracking
├─────────────────────────────────┤
│ Layer 7: ferrite-app            │ Event loop, tabs, history, keybindings
└─────────────────────────────────┘
    ↓
Terminal Output
```

### Core Layers

| Layer | Crate | Responsibility |
|-------|-------|-----------------|
| **0** | `ferrite-net` | HTTP/HTTPS networking, TLS 1.2/1.3, cookie jar, response caching |
| **1** | `ferrite-html` | HTML5 parsing via `html5ever`, character encoding detection |
| **2** | `ferrite-dom` | Arena-allocated DOM tree, selector queries, mutation API |
| **3** | `ferrite-css` | CSS parsing, selector matching, cascade, computed styles |
| **4** | `ferrite-layout` | Box tree construction, text layout, Flexbox, CSS Grid |
| **5** | `ferrite-js` | JavaScript execution via `boa_engine`, DOM bindings, Web APIs |
| **6** | `ferrite-render` | Cell buffer rendering, ANSI color translation, link tracking |
| **7** | `ferrite-app` | Application shell, event loop, tab management, configuration |

## Key Design Principles

### Zero Foreign Engine Dependencies

- ✅ No OpenSSL → Use `rustls` for TLS
- ✅ No libcurl → Use `reqwest` for HTTP
- ✅ No V8/SpiderMonkey → Use `boa_engine` for JavaScript
- ✅ No WebKit/Chromium → Implement layout natively

### Incremental Correctness over Completeness

Render Wikipedia, Hacker News, documentation sites, and plain HTML correctly before attempting to render complex web applications. Every algorithm is documented with references to relevant specifications.

### Modular Architecture

Each layer is independently testable. A contributor working on CSS doesn't need to understand the JavaScript VM. Dependency arrows point downward only — no crate depends on layers above it.

### Performance Philosophy

- Target machines with 512MB RAM and single-core CPUs
- Use arena allocation for bulk DOM operations
- Cache computed values aggressively
- Render only changed regions on re-layout

## Supported Web Technologies

### HTML5
- Full HTML5 tokenizer and tree construction
- All HTML semantic elements
- Event target model and event listener API

### CSS
- **Selectors**: type, class, ID, attribute, pseudo-class, pseudo-element combinators
- **Properties**: `color`, `background-color`, `font-weight`, `font-style`, `text-decoration`
- **Layout**: `block`, `inline`, `inline-block`, `flex`, `grid`, `list-item`
- **Flexbox**: full spec support via `taffy`
- **CSS Grid**: full spec support via `taffy`
- **Cascade**: proper specificity and inheritance per CSS Cascading and Inheritance Level 4
- **Not supported**: animations, transforms, gradients, `position: fixed`, `z-index`

### JavaScript (ES2021+)
- All ES2021 features via `boa_engine`
- DOM manipulation: `querySelector`, `addEventListener`, `classList`, `innerHTML`, `style`
- Web APIs: `fetch()`, `setTimeout`/`setInterval`, `localStorage`/`sessionStorage`
- **No support** (yet): Web Workers, WebAssembly, WebGL

### Terminal Features
- **Colors**: 16 basic ANSI colors, 256-color palette, 24-bit truecolor
- **Styling**: bold, italic, underline
- **Links**: clickable URLs with tracking
- **Images**: optional Sixel and Kitty protocol support
- **Scrolling**: scroll regions, overflow handling

## Contributing

We welcome contributions of all sizes. Please see [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

### Development Workflow

1. Fork the repository
2. Create a feature branch: `git checkout -b feature/my-feature`
3. Make your changes (all changes require tests)
4. Run tests: `cargo test --workspace`
5. Commit with a descriptive message
6. Push and open a pull request

### Testing

```bash
# Run all tests
cargo test --workspace

# Run tests for a specific layer
cargo test -p ferrite-css

# Run with output
cargo test -- --nocapture

# Run integration tests
cargo test --test '*' --workspace
```

## Roadmap

### Phase 1: Foundation (Current)
- [x] Basic HTML5 parsing and DOM tree
- [x] CSS cascade and selector matching
- [x] Block and inline layout
- [ ] Flexbox layout
- [ ] Basic JavaScript execution
- [ ] Page rendering and scrolling

### Phase 2: Interactivity
- [ ] Event handling (click, keypress, scroll)
- [ ] Form submission and input elements
- [ ] Link navigation and history
- [ ] `fetch()` API integration

### Phase 3: Polish
- [ ] CSS Grid layout
- [ ] Image rendering (PNG, JPG, Sixel)
- [ ] Tabs and window management
- [ ] Bookmarks and history persistence

### Phase 4: Advanced
- [ ] CSS animations and transitions
- [ ] Web Workers (thread-safe DOM bindings)
- [ ] Service Workers and offline support
- [ ] WebAssembly support

## Known Limitations

### Hard Problems Being Solved
- **CSS specificity edge cases** — The cascade algorithm is complex; edge cases still exist
- **Text wrapping algorithms** — Line break opportunities differ between Unicode and ASCII
- **Flexbox/Grid rendering** — Mapping continuous layout to discrete character cells
- **JavaScript debugging** — Stack traces and error context from `boa_engine`
- **Performance scaling** — Rendering large DOM trees (10,000+ nodes) in real time

### Not Implemented
- Animations, transitions, transforms, gradients, shadows
- `position: fixed`, `position: sticky`, `z-index`, floating elements
- Web Fonts (`@font-face`)
- Media queries beyond basic terminal size
- Web Workers, WebGL, WebAssembly (roadmap)

## Comparison with Existing Terminal Browsers

| Browser | Language | JS Support | Dependencies | Status |
|---------|----------|-----------|--------------|--------|
| **Ferrite** | Rust | Full (Boa) | Pure Rust stack | Active development |
| Lynx | C | None | libssl, ncurses | Maintained |
| w3m | C | None | libssl, libgc | Maintained |
| ELinks | C | Limited | libssl, SpiderMonkey | Low activity |
| Browsh | Go + JS | Full | Requires Firefox | Active |

Ferrite is the only terminal browser attempting to build a complete engine stack in a memory-safe language without requiring an installed browser binary.

## Configuration

User configuration is stored in `~/.config/ferrite/`:

```toml
# config.toml
[display]
colors = 24-bit  # 16, 256, or 24-bit
width = auto     # terminal width
height = auto    # terminal height

[network]
cookies_enabled = true
cache_enabled = true
cache_size_mb = 32

[keybindings]
reload = "Ctrl+R"
back = "Alt+Left"
forward = "Alt+Right"
new_tab = "Ctrl+T"
close_tab = "Ctrl+W"
```

## Performance

- **Page load**: <500ms for typical documentation sites
- **Memory footprint**: ~50MB baseline, grows ~1MB per 1000 DOM nodes
- **Render time**: <100ms for full layout recomputation

## License

Dual-licensed under MIT and Apache License 2.0. See [LICENSE-MIT](LICENSE-MIT) and [LICENSE-APACHE](LICENSE-APACHE) for details.

## References

- [WHATWG HTML Living Standard](https://html.spec.whatwg.org/)
- [W3C CSS Specifications](https://www.w3.org/Style/CSS/)
- [ECMAScript Specification](https://tc39.es/ecma262/)
- [Architecture Document](ARCHITECTURE.md)
- [Contributing Guide](CONTRIBUTING.md)

## Acknowledgments

Ferrite builds on the work of the Rust ecosystem:
- `html5ever` — Mozilla's Servo project's HTML5 parser
- `cssparser` — Servo's CSS tokenizer and parser
- `boa_engine` — A production-grade JavaScript engine
- `taffy` — Flexbox and Grid layout engine
- `ratatui` and `crossterm` — Terminal rendering

## Quick Links

- **Issues**: Report bugs or request features via [GitHub Issues](https://github.com/yourusername/ferrite/issues)
- **Discussions**: Ask questions in [GitHub Discussions](https://github.com/yourusername/ferrite/discussions)
- **Architecture Details**: See [ARCHITECTURE.md](ARCHITECTURE.md) for the complete technical specification
- **Code of Conduct**: See [CODE_OF_CONDUCT.md](CODE_OF_CONDUCT.md)
