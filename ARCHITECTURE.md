# Ferrite — Pure Rust Terminal Browser
### Project Foundation Document · Architecture · Roadmap · Contributing

> *"A terminal browser that owes nothing to anyone — no V8, no WebKit, no Chromium, no Firefox. Every line, pure Rust."*

---

## Table of Contents

1. [Vision & Philosophy](#1-vision--philosophy)
2. [Why This Project Exists](#2-why-this-project-exists)
3. [What "Pure Rust" Actually Means](#3-what-pure-rust-actually-means)
4. [High-Level Architecture](#4-high-level-architecture)
5. [Workspace Structure](#5-workspace-structure)
6. [Layer 0 — Networking](#6-layer-0--networking)
7. [Layer 1 — HTML5 Parser](#7-layer-1--html5-parser)
8. [Layer 2 — DOM Tree](#8-layer-2--dom-tree)
9. [Layer 3 — CSS Engine](#9-layer-3--css-engine)
10. [Layer 4 — Terminal Layout Engine](#10-layer-4--terminal-layout-engine)
11. [Layer 5 — JavaScript Engine](#11-layer-5--javascript-engine)
12. [Layer 6 — Terminal Painter](#12-layer-6--terminal-painter)
13. [Layer 7 — Event Loop & Application Shell](#13-layer-7--event-loop--application-shell)
14. [Crate Dependency Map](#14-crate-dependency-map)
15. [Phased Roadmap](#15-phased-roadmap)
16. [Known Hard Problems](#16-known-hard-problems)
17. [Design Decisions & Trade-offs](#17-design-decisions--trade-offs)
18. [Performance Philosophy](#18-performance-philosophy)
19. [Security Model](#19-security-model)
20. [Testing Strategy](#20-testing-strategy)
21. [Contributing](#21-contributing)
22. [Governance](#22-governance)
23. [Comparison with Existing Terminal Browsers](#23-comparison-with-existing-terminal-browsers)
24. [References & Inspiration](#24-references--inspiration)

---

## 1. Vision & Philosophy

Ferrite is a terminal browser written entirely in Rust — no embedded C++ engines, no foreign JavaScript runtimes, no dependency on an installed browser binary. It is designed to run on any machine that can compile Rust, including low-spec hardware.

The long-term goal is a terminal browser that, given enough time and contributors, needs nothing outside of the Rust ecosystem to render the web in a character-cell terminal. This is not a weekend project — it is a multi-year open-source effort and this document is its foundation.

**Core principles:**

- **Zero foreign engine dependencies.** No V8. No SpiderMonkey. No WebKit. No libcurl. No OpenSSL. No Chromium. If it cannot be expressed in Rust, we write the Rust.
- **Low resource footprint.** Targeting machines with 512MB RAM and single-core CPUs. The browser should feel fast on hardware that chokes on Chromium.
- **Incremental correctness over completeness.** A browser that renders Wikipedia, Hacker News, documentation sites, and plain HTML correctly is more valuable early on than one that half-renders everything.
- **Modular, contributor-friendly architecture.** Every layer is a separate crate. A contributor who knows CSS should be able to work on `ferrite-css` without understanding the JavaScript VM.
- **No magic, no shortcuts.** Every algorithm is documented in code comments with references to the relevant spec section. Future contributors must be able to trace every behavior back to a standard.

---

## 2. Why This Project Exists

### The problem with existing terminal browsers

| Browser | Written in | JS support | Dependencies |
|---------|------------|------------|--------------|
| Lynx | C | None | libssl, ncurses |
| w3m | C | None | libssl, libgc |
| ELinks | C | Limited | libssl, SpiderMonkey (optional) |
| Browsh | Go + JS | Full (headless Firefox) | Firefox must be installed |

Every existing terminal browser is either written in C (with all the memory safety implications that brings), lacks JavaScript support entirely, or — in Browsh's case — is a thin wrapper around an installed Firefox instance that defeats the low-resource goal entirely.

There is no terminal browser written in a memory-safe language that attempts to implement its own engine stack from scratch. Ferrite fills this gap.

### The opportunity

The Rust ecosystem has matured to the point where most browser sub-components now exist as production-quality Rust crates: `html5ever` for HTML5 parsing (from Mozilla's Servo project), `cssparser` for CSS tokenization (also from Servo), `taffy` for CSS Flexbox/Grid layout (from Dioxus Labs), `boa_engine` for JavaScript execution, and `ratatui` + `crossterm` for terminal rendering. The infrastructure is there. What is missing is the browser that connects all of it and, over time, replaces the third-party crates it relies on with its own implementations.

---

## 3. What "Pure Rust" Actually Means

"Pure Rust" in this project means:

1. **No C/C++ foreign function interface (FFI) in the critical path.** `rustls` replaces OpenSSL. `html5ever` replaces libxml2/libexpat. We do not link against any browser engine.
2. **No subprocess execution of external browsers.** Ferrite does not launch Firefox, Chrome, or any headless browser in the background.
3. **No Node.js, Deno, or Bun runtime.** JavaScript — where supported — is executed by a Rust-native engine.
4. **`sys` crates are acceptable for terminal I/O.** `crossterm` may use platform syscalls. This is acceptable because it is not a browser engine — it is an OS interface layer.
5. **The long-term goal for every major layer is first-party.** Where we currently use `html5ever`, `boa_engine`, or `taffy`, the intent is to eventually replace them with `ferrite-html`, `ferrite-js`, and `ferrite-layout` respectively — fully under project control.

---

## 4. High-Level Architecture

```
┌─────────────────────────────────────────────────────┐
│                   User Input / URL Bar               │
└─────────────────────┬───────────────────────────────┘
                      │ URL string
                      ▼
┌─────────────────────────────────────────────────────┐
│              Layer 0 · ferrite-net                   │
│   HTTP/1.1, HTTP/2, TLS (rustls), cookies, cache    │
└─────────────────────┬───────────────────────────────┘
                      │ Raw bytes + MIME type
                      ▼
┌─────────────────────────────────────────────────────┐
│              Layer 1 · ferrite-html                  │
│   HTML5 tokenizer → tree construction → DOM nodes   │
└─────────────────────┬───────────────────────────────┘
                      │ Raw DOM tree
                      ▼
┌──────────────────────────────────────────────────────┐
│              Layer 2 · ferrite-dom                   │
│   Node tree, parent/child links, event targets,      │
│   querySelector, attribute access, mutation API      │
└───────────┬──────────────────────┬───────────────────┘
            │                      │
            ▼                      ▼
┌─────────────────┐    ┌──────────────────────────────┐
│  Layer 3        │    │  Layer 5 · ferrite-js         │
│  ferrite-css    │    │  Lexer → Parser → AST →       │
│  Tokenizer,     │    │  Bytecode → Register VM       │
│  selector match,│    │  DOM bindings, fetch(), timers│
│  cascade,       │    └──────────────┬───────────────┘
│  computed values│                   │ DOM mutations
└────────┬────────┘                   │
         │ Styled DOM                 │
         ▼                            ▼
┌─────────────────────────────────────────────────────┐
│              Layer 4 · ferrite-layout                │
│   Block, Inline, Flex, Grid → character-cell boxes  │
│   Word wrap, overflow, scroll regions               │
└─────────────────────┬───────────────────────────────┘
                      │ Positioned layout tree
                      ▼
┌─────────────────────────────────────────────────────┐
│              Layer 6 · ferrite-render                │
│   Cell buffer, ANSI colors, bold/italic, links,     │
│   scroll offset, optional Sixel image rendering     │
└─────────────────────┬───────────────────────────────┘
                      │ Ratatui frame
                      ▼
┌─────────────────────────────────────────────────────┐
│              Layer 7 · ferrite-app                   │
│   Event loop (tokio + crossterm), tab manager,      │
│   history, bookmarks, config, keybindings           │
└─────────────────────────────────────────────────────┘
```

Data flows top-to-bottom on every page load. Events (keypress, resize, link click, timeout firing) flow from Layer 7 back up to trigger partial or full re-renders. The DOM is the shared mutable state that Layers 2, 3, 4, 5, and 6 all operate on.

---

## 5. Workspace Structure

The repository is organized as a Cargo workspace. Each layer is an independent crate.

```
ferrite/
├── Cargo.toml                  # Workspace root
├── ARCHITECTURE.md             # This file
├── CONTRIBUTING.md
├── CODE_OF_CONDUCT.md
├── LICENSE-MIT
├── LICENSE-APACHE
├── README.md
│
├── crates/
│   ├── ferrite-net/            # Layer 0: Networking
│   │   ├── Cargo.toml
│   │   └── src/
│   │       ├── lib.rs
│   │       ├── client.rs       # HTTP client abstraction
│   │       ├── tls.rs          # rustls integration
│   │       ├── cache.rs        # In-memory + disk cache
│   │       ├── cookies.rs      # Cookie jar
│   │       └── redirect.rs     # Redirect chain handling
│   │
│   ├── ferrite-html/           # Layer 1: HTML5 Parser
│   │   ├── Cargo.toml
│   │   └── src/
│   │       ├── lib.rs
│   │       ├── tokenizer.rs    # HTML5 tokenizer states
│   │       ├── tree_builder.rs # Tree construction algorithm
│   │       └── encoding.rs     # encoding_rs integration
│   │
│   ├── ferrite-dom/            # Layer 2: DOM
│   │   ├── Cargo.toml
│   │   └── src/
│   │       ├── lib.rs
│   │       ├── node.rs         # Node enum and NodeData
│   │       ├── arena.rs        # Arena allocator
│   │       ├── selector.rs     # querySelector / querySelectorAll
│   │       ├── events.rs       # EventTarget, addEventListener
│   │       └── mutation.rs     # DOM mutation API
│   │
│   ├── ferrite-css/            # Layer 3: CSS Engine
│   │   ├── Cargo.toml
│   │   └── src/
│   │       ├── lib.rs
│   │       ├── tokenizer.rs    # CSS tokenizer (wraps cssparser)
│   │       ├── parser.rs       # Rule / declaration parser
│   │       ├── selector.rs     # Selector matching engine
│   │       ├── cascade.rs      # Cascade + specificity
│   │       ├── computed.rs     # Specified → computed values
│   │       └── properties.rs   # All supported CSS properties
│   │
│   ├── ferrite-layout/         # Layer 4: Layout Engine
│   │   ├── Cargo.toml
│   │   └── src/
│   │       ├── lib.rs
│   │       ├── box_tree.rs     # CSS box tree construction
│   │       ├── block.rs        # Block formatting context
│   │       ├── inline.rs       # Inline formatting context
│   │       ├── flex.rs         # Flexbox (wraps taffy initially)
│   │       ├── grid.rs         # CSS Grid (wraps taffy initially)
│   │       ├── text.rs         # Text measurement in char units
│   │       └── scroll.rs       # Overflow and scroll regions
│   │
│   ├── ferrite-js/             # Layer 5: JavaScript Engine
│   │   ├── Cargo.toml
│   │   └── src/
│   │       ├── lib.rs
│   │       ├── engine.rs       # Engine trait + boa adapter
│   │       ├── dom_bindings.rs # document.*, element.*, window.*
│   │       ├── fetch.rs        # fetch() wired to ferrite-net
│   │       ├── timers.rs       # setTimeout / setInterval
│   │       └── storage.rs      # localStorage / sessionStorage
│   │
│   ├── ferrite-render/         # Layer 6: Terminal Painter
│   │   ├── Cargo.toml
│   │   └── src/
│   │       ├── lib.rs
│   │       ├── cell_buffer.rs  # 2D character cell buffer
│   │       ├── painter.rs      # Layout tree → cell buffer
│   │       ├── ansi.rs         # Color and style translation
│   │       ├── links.rs        # Link region tracking
│   │       └── images.rs       # Sixel / Kitty protocol (opt-in)
│   │
│   └── ferrite-app/            # Layer 7: Application Shell
│       ├── Cargo.toml
│       └── src/
│           ├── main.rs
│           ├── event_loop.rs   # tokio::select! over all events
│           ├── tabs.rs         # Tab manager
│           ├── history.rs      # Navigation history stack
│           ├── bookmarks.rs    # Bookmark store (TOML)
│           ├── config.rs       # User config (TOML)
│           └── keybindings.rs  # Configurable keybindings
│
├── tests/
│   ├── integration/            # End-to-end render tests
│   └── fixtures/               # Static HTML/CSS test files
│
└── docs/
    ├── decisions/              # Architecture Decision Records (ADRs)
    └── specs/                  # Annotated spec references
```

**Rule:** No crate in this workspace may depend on a crate above it in the layer stack. `ferrite-css` may depend on `ferrite-dom`. `ferrite-dom` must never depend on `ferrite-css`. Dependency arrows point downward only.

---

## 6. Layer 0 — Networking

**Crate:** `ferrite-net`  
**Responsible for:** Making HTTP requests, handling TLS, managing cookies, caching responses, following redirects.

### Dependencies

| Crate | Purpose | Why not first-party yet |
|-------|---------|------------------------|
| `reqwest` | HTTP client | Battle-tested async HTTP; writing HTTP/2 from scratch is months of work |
| `rustls` | TLS 1.2 / 1.3 | Pure Rust TLS — **no OpenSSL** |
| `tokio` | Async runtime | Industry standard; drives the event loop |
| `url` | URL parsing + resolution | Handles relative URLs, fragments, encoding |
| `encoding_rs` | Charset detection | Firefox's own charset library; handles every encoding the web uses |
| `mime` | MIME type parsing | Determine how to handle response bodies |

### Key design decisions

**Cookie storage** is an in-memory `HashMap<RegistrableDomain, Vec<Cookie>>` with optional disk persistence to `~/.config/ferrite/cookies.json`. No third-party cookie tracking — cross-origin cookies are blocked by default.

**Cache** is a two-level system: an in-memory LRU cache (configurable size, default 32MB) and an optional disk cache at `~/.cache/ferrite/`. Cache-Control headers are respected. ETags and Last-Modified are honored for conditional requests.

**HTTP/2** is supported via `reqwest`'s built-in `h2` feature. HTTP/3 / QUIC is deferred to a later milestone.

**Redirects** are followed automatically up to a configurable maximum (default 10). Redirect history is preserved for display in the status bar.

### The `Fetcher` trait

All networking goes through a `Fetcher` trait:

```rust
#[async_trait]
pub trait Fetcher: Send + Sync {
    async fn fetch(&self, request: Request) -> Result<Response, FetchError>;
}
```

This allows test code to inject a `MockFetcher` that serves fixture files without making real network requests. Every integration test uses this.

---

## 7. Layer 1 — HTML5 Parser

**Crate:** `ferrite-html`  
**Responsible for:** Turning raw HTML bytes into a sequence of parse events that the DOM layer can consume.

### Current approach: html5ever

In Phase 1, `ferrite-html` wraps `html5ever` from Mozilla's Servo project. `html5ever` is a production-grade, WHATWG-spec-compliant HTML5 parser written in pure Rust. It passes all tokenizer tests from `html5lib-tests` and handles the full range of malformed, real-world HTML that the internet contains.

`html5ever` uses a callback (sink) API rather than building its own DOM tree. `ferrite-html` implements the `TreeSink` trait to feed parse events directly into `ferrite-dom`'s arena allocator. This avoids building an intermediate tree.

### Long-term goal: ferrite-html native tokenizer

The WHATWG HTML5 parsing spec defines a deterministic state machine for tokenization. Every state and transition is enumerable. The long-term goal is to implement this state machine directly in `ferrite-html/src/tokenizer.rs`, removing the `html5ever` dependency. This is a multi-month effort that should only begin once the rest of the browser stack is stable.

**Spec reference:** https://html.spec.whatwg.org/multipage/parsing.html

### Encoding detection

HTML encoding detection follows the WHATWG encoding spec priority order:
1. BOM sniffing (UTF-8, UTF-16 LE/BE)
2. `Content-Type` HTTP header charset parameter
3. `<meta charset="...">` or `<meta http-equiv="Content-Type" ...>` pragma
4. `encoding_rs` prescan algorithm
5. Default: UTF-8

---

## 8. Layer 2 — DOM Tree

**Crate:** `ferrite-dom`  
**Responsible for:** Storing the parsed document as a queryable, mutable tree of nodes. The DOM is the shared state of the entire browser.

### The Rust ownership problem

A DOM tree is a graph with cycles. Every child node has a reference to its parent; every parent has references to all children. Rust's ownership system does not natively permit this.

We solve it with an **arena allocator**. All nodes are allocated into a single `Arena<Node>` that owns all memory. References between nodes are `NodeId` values — typed indices into the arena — not Rust references. This eliminates the ownership cycle problem entirely, gives O(1) node allocation, and enables bulk deallocation of an entire document in O(1) by dropping the arena.

```rust
pub struct Arena {
    nodes: Vec<NodeData>,
}

#[derive(Clone, Copy, PartialEq, Eq, Hash, Debug)]
pub struct NodeId(u32);

pub struct NodeData {
    pub kind:     NodeKind,
    pub parent:   Option<NodeId>,
    pub first_child: Option<NodeId>,
    pub last_child:  Option<NodeId>,
    pub next_sibling: Option<NodeId>,
    pub prev_sibling: Option<NodeId>,
}

pub enum NodeKind {
    Document,
    Element(ElementData),
    Text(String),
    Comment(String),
    Doctype(DoctypeData),
}
```

### DOM API surface (JS-facing)

These are the methods wired to `ferrite-js` DOM bindings, in priority order:

**Phase 1 (required for any JS to work):**
- `document.querySelector(selector)` → `Option<NodeId>`
- `document.querySelectorAll(selector)` → `Vec<NodeId>`
- `element.textContent` getter/setter
- `element.innerHTML` getter/setter
- `element.getAttribute(name)` / `setAttribute(name, value)`
- `element.classList` (add, remove, contains, toggle)
- `element.style` (inline style mutation)
- `document.createElement(tag)` / `document.createTextNode(data)`
- `parent.appendChild(child)` / `parent.removeChild(child)`

**Phase 2:**
- `element.addEventListener(type, callback)`
- `element.dispatchEvent(event)`
- `document.getElementById(id)`
- `element.getBoundingClientRect()` (returns char-cell coordinates)

**Phase 3:**
- `window.location` (href, assign, replace, reload)
- `history.pushState` / `history.replaceState`
- `MutationObserver`
- `IntersectionObserver` (simplified)

---

## 9. Layer 3 — CSS Engine

**Crate:** `ferrite-css`  
**Responsible for:** Parsing CSS, matching selectors against the DOM, computing the cascade, and producing computed style values for every element.

### What CSS properties we implement

Because the output is a character-cell terminal, many CSS properties are irrelevant. We implement a curated subset:

**Fully implemented:**
- `color`, `background-color` → ANSI terminal colors (16, 256, or truecolor)
- `font-weight` → bold terminal attribute
- `font-style` → italic terminal attribute
- `text-decoration` → underline terminal attribute
- `display` (block, inline, inline-block, flex, grid, none, list-item)
- `visibility` (visible, hidden)
- `margin`, `padding` (mapped to character offsets)
- `width`, `height` (in character columns/rows, or percentage of terminal)
- `max-width`, `min-width`, `max-height`, `min-height`
- `overflow` (visible, hidden, scroll, auto)
- `white-space` (normal, pre, nowrap, pre-wrap, pre-line)
- `text-align` (left, right, center, justify)
- `list-style-type` (disc, decimal, none, etc.)
- `flex-direction`, `flex-wrap`, `justify-content`, `align-items`, `flex-grow`, `flex-shrink`, `flex-basis`
- CSS Grid track definitions, `grid-template-columns`, `grid-template-rows`

**Explicitly not implemented (terminal has no concept of these):**
- `transform`, `transition`, `animation`, `@keyframes`
- `border-radius`, `box-shadow`, `filter`, `opacity`
- `position: fixed` or `position: sticky` (scroll-based positioning)
- `float` and `clear` (approximated as block layout)
- `z-index` (terminal has no z axis)
- `font-family`, `font-size` (terminal uses one fixed-width font)

### CSS cascade algorithm

The cascade follows the CSS Cascading and Inheritance Level 4 specification:

1. **Collect** all declarations that apply to an element (from user-agent stylesheet, author stylesheets, inline styles)
2. **Sort** by origin (user-agent < author < inline) and `!important` flag
3. **Sort by specificity** within the same origin: (a, b, c) where a = ID selectors, b = class/attribute/pseudo-class, c = type/pseudo-element
4. **Sort by order** — later declarations win on equal specificity
5. **Inherit** unset properties from parent where `inherited: true` in the property definition
6. **Apply initial values** for all remaining unset properties

The result is a `ComputedStyle` struct stored per `NodeId`, accessible by both the layout engine and the painter.

### cssparser integration

The `cssparser` crate from Mozilla provides the low-level CSS tokenizer and block parser. `ferrite-css` uses it to implement `StylesheetParser` and `DeclarationParser`. This is an implementation detail; the public API of `ferrite-css` exposes only high-level types.

---

## 10. Layer 4 — Terminal Layout Engine

**Crate:** `ferrite-layout`  
**Responsible for:** Taking the styled DOM and producing a positioned layout tree where every node has an `(x, y, width, height)` in character-cell coordinates.

### The terminal coordinate system

Unlike a graphical browser where coordinates are in pixels, Ferrite's coordinate system is:
- **x** — column index (0 = leftmost column visible)
- **y** — row index (0 = topmost row of the content area)
- **width** — number of character columns
- **height** — number of character rows

The terminal's current width and height in cells is queried from `crossterm` on startup and on every resize event. Layout is recomputed on resize.

### Box tree construction

The DOM + computed styles are first converted into a **box tree** — a tree of anonymous and non-anonymous boxes following the CSS visual formatting model:

- **Block boxes** — elements with `display: block`
- **Inline boxes** — elements with `display: inline`; sequences of inline boxes are wrapped in anonymous block containers
- **Inline-block boxes** — treated as atomic inline-level blocks
- **Flex containers / items** — `display: flex`
- **Grid containers / items** — `display: grid`
- **List item boxes** — include marker boxes
- **`display: none` subtrees** — pruned from the box tree entirely

### Flexbox and Grid: Taffy

In Phase 1, `ferrite-layout` delegates Flexbox and CSS Grid layout computation to the `taffy` crate from Dioxus Labs, which implements both algorithms faithfully against the CSS specification. Taffy's `TaffyTree` is populated with nodes and style data derived from our computed CSS values, and the resulting positions are mapped back to character-cell coordinates.

The long-term goal is to remove the `taffy` dependency and implement these layout algorithms natively in `ferrite-layout`, where we can optimize specifically for integer character-cell arithmetic.

### Inline layout and text wrapping

Inline layout is the hardest part of this layer. Text must wrap at word boundaries within the available column width. The `unicode-linebreak` crate provides Unicode line-break opportunity detection (UAX #14). Each word gets a measured width in columns (most glyphs = 1 column; CJK ideographs and emoji = 2 columns via `unicode-width`).

Inline boxes are collected into **line boxes**. When a line box would exceed the container's available width, a new line is started. The resulting line boxes are stacked vertically to fill the block container.

---

## 11. Layer 5 — JavaScript Engine

**Crate:** `ferrite-js`  
**Responsible for:** Executing JavaScript found in `<script>` tags or event handlers, with access to the DOM and Web APIs.

### Phase 1: Boa engine adapter

In Phase 1, `ferrite-js` wraps the `boa_engine` crate. Boa is a pure-Rust JavaScript engine with 94.12% ECMAScript Test262 compliance as of v0.21. It is an interpreter (no JIT), which is acceptable for our initial goals.

`ferrite-js` is structured around an `Engine` trait:

```rust
pub trait Engine: Send {
    fn eval(&mut self, source: &str, context: &mut DomContext) -> Result<JsValue, JsError>;
    fn register_dom_api(&mut self, dom: Arc<Mutex<Arena>>);
    fn dispatch_event(&mut self, event: DomEvent) -> Result<(), JsError>;
}
```

The `BoeEngine` struct implements this trait by wrapping `boa_engine::Context`. The `DomContext` carries a reference to the DOM arena, allowing JS code to mutate the tree.

### DOM bindings

DOM bindings are Rust functions registered into Boa's `Context` as native host objects. For example:

```rust
// document.querySelector("selector") → NodeId | null
fn js_query_selector(
    _this: &JsValue,
    args: &[JsValue],
    ctx: &mut Context,
) -> JsResult<JsValue> {
    let selector = args.get(0)...;
    let dom = DOM_CONTEXT.with(|d| d.borrow().clone());
    match dom.query_selector(&selector) {
        Some(node_id) => Ok(node_id_to_js_object(node_id, ctx)),
        None => Ok(JsValue::null()),
    }
}
```

Every DOM mutation triggers a **dirty flag** on affected nodes. The event loop checks this flag at the end of each JS execution cycle and schedules a partial or full re-render.

### Web API surface (Phase 1)

| API | Implementation |
|-----|----------------|
| `console.log/warn/error` | Write to status bar / debug log |
| `document.querySelector` / `querySelectorAll` | `ferrite-dom` selector engine |
| `document.getElementById` | `ferrite-dom` |
| `element.textContent` / `innerHTML` | `ferrite-dom` mutation |
| `element.style.*` | Inline style mutation, triggers re-layout |
| `element.classList.*` | Class list mutation |
| `window.setTimeout` / `clearTimeout` | `tokio::time::sleep` tasks |
| `window.setInterval` / `clearInterval` | `tokio::time::interval` tasks |
| `fetch(url)` | `ferrite-net::Fetcher` |
| `window.location.href` | Navigation event to event loop |
| `localStorage` | File-backed JSON store |
| `JSON.parse` / `JSON.stringify` | Native to Boa |
| `XMLHttpRequest` | Deferred to Phase 2 |

### Long-term: ferrite-js native VM

The eventual goal is a JavaScript VM written from scratch within this project. This is a multi-year effort. The architecture target is:

```
Source text
  └─► Lexer (ferrite-js/src/lexer.rs)
        └─► Recursive-descent parser (ferrite-js/src/parser.rs)
              └─► AST (ferrite-js/src/ast.rs)
                    └─► Bytecode compiler (ferrite-js/src/compiler.rs)
                          └─► Register-based VM (ferrite-js/src/vm.rs)
                                └─► Garbage collector (ferrite-js/src/gc.rs)
```

This will only begin once the browser renders static pages correctly and has an active contributor community. Boa is a perfectly good intermediate solution; the project is not blocked on writing its own JS engine.

---

## 12. Layer 6 — Terminal Painter

**Crate:** `ferrite-render`  
**Responsible for:** Walking the positioned layout tree and writing styled characters into a `ratatui` buffer.

### Cell buffer model

Ratatui's `Buffer` is a 2D array of `Cell` structs. Each `Cell` contains:
- A Unicode character (or grapheme cluster)
- A foreground ANSI color
- A background ANSI color
- Style flags: bold, italic, underline, dim, crossed-out, blink

The painter walks the layout tree in document order and fills the buffer. Text nodes at position `(x, y)` write their characters to the buffer starting at column `x`, row `y`. Block boxes clip their content to their bounding box.

### Color translation

CSS colors are translated to terminal colors using the following priority:

1. **Truecolor** — if the terminal supports 24-bit color (`$COLORTERM=truecolor` or `$TERM_PROGRAM=iTerm.app`), CSS `rgb()` and hex colors are passed directly as ANSI 24-bit escapes.
2. **256-color** — if 24-bit is not available, CSS colors are quantized to the nearest color in the xterm-256 palette.
3. **16-color** — fallback for terminals that only support the basic 16 ANSI colors. Colors are mapped to the nearest ANSI color by luminance.

### Link region tracking

Every link (`<a href="...">`) in the layout tree registers a `LinkRegion`:

```rust
pub struct LinkRegion {
    pub node_id: NodeId,
    pub href:    String,
    pub x: u16, pub y: u16,
    pub width: u16, pub height: u16,
}
```

The event loop uses these regions to:
- Highlight the focused link (tab navigation)
- Open the link on Enter or mouse click
- Display the URL in the status bar on hover

### Scroll state

Vertical scroll position is stored as `scroll_y: u32` in `AppState`. The painter offsets all `y` coordinates by `-scroll_y` and clips anything with `y < 0` or `y >= terminal_height`. This means the layout tree is always computed for the full document height — only the painting pass is windowed.

### Image rendering (opt-in)

If the terminal emulator supports Sixel graphics (`$TERM` contains `sixel`) or the Kitty graphics protocol (`$TERM_PROGRAM=kitty`), `ferrite-render` can decode and render inline images using the `viuer` crate. This is disabled by default. Enable with `--enable-images` at the command line or `images = true` in the config file.

---

## 13. Layer 7 — Event Loop & Application Shell

**Crate:** `ferrite-app`  
**Responsible for:** The main binary, the application state machine, tab management, history, bookmarks, and configuration.

### Application state

```rust
pub struct AppState {
    pub tabs:         Vec<Tab>,
    pub active_tab:   usize,
    pub mode:         Mode,           // Normal | UrlEntry | Search | Command
    pub status:       StatusLine,
    pub config:       Config,
}

pub struct Tab {
    pub url:          String,
    pub title:        String,
    pub dom:          Arc<Mutex<Arena>>,
    pub layout:       Option<LayoutTree>,
    pub scroll_y:     u32,
    pub history:      Vec<HistoryEntry>,
    pub history_pos:  usize,
    pub load_state:   LoadState,      // Idle | Loading | Rendering | Error
    pub link_regions: Vec<LinkRegion>,
    pub focused_link: Option<usize>,
}
```

### Event loop

```rust
loop {
    tokio::select! {
        // Terminal keyboard/mouse event
        event = crossterm_event() => handle_input(event, &mut state),

        // Page fetch completed
        result = fetch_receiver.recv() => handle_fetch(result, &mut state),

        // JS setTimeout / setInterval fired
        timer = timer_receiver.recv() => handle_timer(timer, &mut state),

        // Terminal resize
        resize = resize_receiver.recv() => handle_resize(resize, &mut state),
    }

    if state.dirty {
        rerender(&mut terminal, &state);
        state.dirty = false;
    }
}
```

### Default keybindings

| Key | Action |
|-----|--------|
| `j` / `↓` | Scroll down one line |
| `k` / `↑` | Scroll up one line |
| `d` | Scroll down half page |
| `u` | Scroll up half page |
| `g` | Scroll to top |
| `G` | Scroll to bottom |
| `Tab` | Focus next link |
| `Shift+Tab` | Focus previous link |
| `Enter` | Follow focused link |
| `H` / `Alt+←` | Go back |
| `L` / `Alt+→` | Go forward |
| `o` / `:` | Open URL bar |
| `r` / `F5` | Reload |
| `t` | New tab |
| `w` | Close current tab |
| `1`–`9` | Switch to tab N |
| `f` | Find in page |
| `b` | Open bookmarks |
| `q` | Quit |
| `?` | Show help |

All keybindings are configurable via `~/.config/ferrite/config.toml`.

### Configuration

```toml
# ~/.config/ferrite/config.toml

[browser]
homepage = "https://en.wikipedia.org"
max_redirects = 10
request_timeout_secs = 30
user_agent = "Ferrite/0.1 (terminal browser)"

[display]
colors = "truecolor"   # truecolor | 256 | 16 | mono
images = false
tab_width = 4
scroll_lines = 3       # lines per scroll step

[privacy]
block_third_party_cookies = true
do_not_track = true

[cache]
memory_mb = 32
disk_cache = true
disk_cache_dir = "~/.cache/ferrite"

[keys]
# Override any default keybinding here
# scroll_down = "ctrl+e"
```

---

## 14. Crate Dependency Map

```
ferrite-app
    ├── ferrite-render
    │       ├── ferrite-layout
    │       │       ├── ferrite-css
    │       │       │       ├── ferrite-dom
    │       │       │       │       └── ferrite-html
    │       │       │       └── [cssparser, selectors]
    │       │       └── [taffy, unicode-linebreak, unicode-width]
    │       └── [ratatui, crossterm, viuer]
    ├── ferrite-js
    │       ├── ferrite-dom
    │       ├── ferrite-net
    │       └── [boa_engine]
    └── ferrite-net
            └── [reqwest, rustls, tokio, url, encoding_rs]
```

**External crates used (full list):**

| Crate | Version | Layer | Will replace? |
|-------|---------|-------|---------------|
| `tokio` | 1.x | 0, 7 | No — OS async runtime |
| `reqwest` | 0.12 | 0 | Long-term: own HTTP |
| `rustls` | 0.23 | 0 | No — pure Rust TLS |
| `url` | 2.x | 0 | No — spec-compliant |
| `encoding_rs` | 0.8 | 0, 1 | No — Firefox's own lib |
| `html5ever` | 0.27 | 1 | Yes — own tokenizer |
| `cssparser` | 0.34 | 3 | Yes — own CSS tokenizer |
| `selectors` | 0.25 | 3 | Yes — own selector engine |
| `taffy` | 0.5 | 4 | Yes — own layout engine |
| `unicode-linebreak` | 0.1 | 4 | No — spec impl |
| `unicode-width` | 0.1 | 4 | No — spec impl |
| `boa_engine` | 0.21 | 5 | Yes — own JS VM |
| `ratatui` | 0.29 | 6 | No — terminal abstraction |
| `crossterm` | 0.29 | 6, 7 | No — OS terminal I/O |
| `viuer` | 0.7 | 6 | No — image protocol |
| `serde` + `toml` | latest | 7 | No |
| `anyhow` / `thiserror` | latest | all | No |

---

## 15. Phased Roadmap

### Phase 0 — Foundation (Months 1–2)
*Goal: Repo setup, CI, workspace structure, basic HTTP fetch to stdout.*

- [ ] Initialize Cargo workspace with all 8 crate stubs
- [ ] Set up GitHub Actions: `cargo check`, `cargo test`, `cargo clippy`, `cargo fmt`
- [ ] `ferrite-net`: basic `GET` request with `reqwest` + `rustls`, print raw body to stdout
- [ ] `ferrite-html`: wrap `html5ever`, extract text content from DOM, print to stdout
- [ ] Write `ARCHITECTURE.md` (this file), `CONTRIBUTING.md`, `CODE_OF_CONDUCT.md`
- [ ] Create initial GitHub issue templates: Bug Report, Feature Request, Good First Issue

**Milestone:** `ferrite-app --dump-text https://en.wikipedia.org/wiki/Rust_(programming_language)` prints readable plain text.

---

### Phase 1 — Readable Browser (Months 3–5)
*Goal: A working read-only browser for static content. No JS. Navigate links.*

- [ ] `ferrite-dom`: arena-based node tree, basic querySelector
- [ ] `ferrite-css`: `cssparser` integration, cascade, computed `color`, `background-color`, `font-weight`, `display`, `margin`, `padding`
- [ ] `ferrite-layout`: block layout, inline text wrapping with `unicode-linebreak`
- [ ] `ferrite-render`: cell buffer painter, ANSI color, bold/italic, link regions
- [ ] `ferrite-app`: ratatui TUI shell, scroll, link focus, Enter to navigate, URL bar, back/forward
- [ ] Integration tests: render Wikipedia, Hacker News, GitHub READMEs, plain HTML fixtures

**Milestone:** `ferrite https://en.wikipedia.org/wiki/Rust_(programming_language)` renders a readable, scrollable, navigable Wikipedia article in the terminal.

---

### Phase 2 — CSS & Layout Depth (Months 6–9)
*Goal: Correct rendering of most static sites. Flexbox. Images (opt-in).*

- [ ] `ferrite-css`: full selector specificity, inheritance, `em`/`rem`/`%` value resolution
- [ ] `ferrite-layout`: Flexbox via `taffy`, Grid via `taffy`, list markers, `overflow: hidden`
- [ ] `ferrite-render`: Sixel / Kitty inline images via `viuer` (opt-in)
- [ ] `ferrite-app`: tabs, bookmarks, configuration file, find-in-page, history persistence
- [ ] `ferrite-net`: HTTP cache (ETag, Cache-Control, Last-Modified), cookie jar
- [ ] Test against: documentation sites (docs.rs, MDN), news sites, plain blogs

**Milestone:** Most documentation-focused websites render correctly. Tabs, bookmarks, and full navigation work.

---

### Phase 3 — JavaScript Foundation (Months 10–15)
*Goal: Execute simple JS. DOM manipulation. Dynamic content on non-SPA sites.*

- [ ] `ferrite-js`: `boa_engine` integration, `Engine` trait, JS execution context per tab
- [ ] DOM bindings: `document.querySelector/All`, `element.textContent`, `element.style.*`, `element.classList.*`, `createElement`, `appendChild`
- [ ] Web API bindings: `console`, `fetch`, `setTimeout/setInterval`, `localStorage`
- [ ] Dirty flag system: DOM mutations trigger incremental re-layout and repaint
- [ ] `window.location` navigation integration
- [ ] Test against: sites with basic DOM manipulation (cookie banners, tab UIs, lazy-loaded content)

**Milestone:** Hacker News, lobste.rs, and similar sites with minimal JS render correctly including dynamic elements.

---

### Phase 4 — Engine Independence (Months 16–30)
*Goal: Replace third-party crates with first-party implementations.*

- [ ] Native HTML5 tokenizer in `ferrite-html` — remove `html5ever` dependency
- [ ] Native CSS selector engine in `ferrite-css` — remove `selectors` crate dependency
- [ ] Native block/inline layout engine in `ferrite-layout` — remove `taffy` dependency
- [ ] Begin `ferrite-js` native lexer and parser (Boa remains the execution backend)
- [ ] Improve JS compatibility: ES2022+ features, Promise chains, async/await
- [ ] HTTP/2 server push, WebSocket support
- [ ] Performance profiling and optimization pass

---

### Phase 5 — Native JS VM (Months 30+)
*Goal: Run the web's JavaScript without any external engine.*

- [ ] Native JS bytecode compiler in `ferrite-js`
- [ ] Register-based VM with mark-and-sweep garbage collector
- [ ] Replace Boa with native VM
- [ ] Tracing JIT (long-term stretch goal)
- [ ] Full ES2024 spec compliance via Test262 test suite
- [ ] `fetch`, `WebSocket`, `Worker` (single-threaded) in native Rust

---

## 16. Known Hard Problems

These are the engineering challenges that will require the most time and the most contributors. Anyone wanting to make a large impact should look here first.

### 1. Inline layout and bidirectional text

The CSS Inline Layout Module Level 3 and Unicode Bidirectional Algorithm (UBA, Unicode Standard Annex #9) are both exceptionally complex. Correctly laying out a paragraph of mixed left-to-right and right-to-left text in a terminal is genuinely hard. We implement LTR first and handle RTL as a later milestone.

### 2. JavaScript + DOM co-mutation

When JavaScript modifies the DOM (e.g., `element.innerHTML = ...`), the following must happen atomically from the user's perspective: DOM mutation → style recalculation for affected nodes → layout recomputation for affected subtree → repaint of affected cells. Getting this pipeline correct, fast, and deadlock-free (given that the DOM lives behind a `Mutex`) is the central concurrency challenge of this project.

### 3. The `Rc<RefCell<T>>` vs Arena trade-off

Early prototypes may use `Rc<RefCell<NodeData>>` for the DOM because it is easier to write. The arena approach requires more upfront design but is significantly faster and eliminates runtime borrow panics. The project commits to the arena from the beginning. Contributors must not introduce `Rc<RefCell<>>` for DOM nodes.

### 4. CSS specificity and the cascade

The CSS cascade is one of the most complex algorithms in software when implemented correctly. The `ferrite-css` crate must handle: multiple stylesheets, inline styles, `!important`, specificity ties broken by source order, user-agent stylesheet, inheritance, and `initial` / `inherit` / `unset` keywords. This will have bugs for years. Fuzzing against real browser computed styles is the long-term testing strategy.

### 5. Writing a GC in Rust

A JavaScript engine requires garbage collection. Rust's ownership system is fundamentally incompatible with the object graph that JS creates. The options are: use `unsafe` with raw pointers (what most mature Rust JS engines do), use `Rc<RefCell<>>` (slow), or use an arena with manual GC. Boa solves this with a custom GC crate (`boa_gc`). Our eventual native VM will need to make the same choice.

---

## 17. Design Decisions & Trade-offs

### ADR-001: Arena allocator for the DOM (not Rc<RefCell<>>)

**Decision:** Use a typed arena with integer `NodeId` indices as inter-node references.  
**Rationale:** `Rc<RefCell<>>` introduces runtime borrow-check overhead on every DOM access and will panic in complex borrow situations that are hard to debug. Arena allocation gives O(1) alloc, O(1) free-all, and zero runtime overhead on node access.  
**Trade-off:** More verbose code when walking the tree; requires passing the `Arena` reference everywhere.

### ADR-002: Boa as the initial JS engine (not rusty_v8)

**Decision:** Use `boa_engine` for Phase 1-3 JavaScript execution.  
**Rationale:** `rusty_v8` compiles V8 from C++ source, adding 30-60 minutes to the first build time on a slow machine. This is unacceptable for a project targeting low-spec hardware. Boa adds ~2MB to the binary and has zero native dependencies.  
**Trade-off:** No JIT compilation means slower JS execution for heavy sites. Acceptable for Phase 1-3 targets (static sites with light scripting).

### ADR-003: Taffy for Flexbox/Grid (not custom implementation)

**Decision:** Use the `taffy` crate for Flexbox and CSS Grid layout in Phase 1-3.  
**Rationale:** Correctly implementing the CSS Flexbox specification is a multi-month effort. Taffy has been tested against Chrome's layout output. Using it lets us get a working browser faster.  
**Trade-off:** Taffy works in floating-point pixels; we must convert to integer character-cell coordinates. The long-term plan is replacement with a native integer-arithmetic layout engine.

### ADR-004: Single-threaded DOM, async networking

**Decision:** DOM access and layout computation run on the main thread. Network requests run in a separate Tokio task pool and communicate via channels.  
**Rationale:** Multithreaded DOM access requires either heavy locking or a complex actor model. All major browser engines use a single main thread for layout/paint. Async networking prevents the UI from freezing during page load.  
**Trade-off:** Long-running JS will block the UI. A future `Worker` API would offload JS to a separate thread.

---

## 18. Performance Philosophy

Ferrite targets machines with constrained resources. Every architectural decision is made with this in mind.

- **Incremental layout.** When a DOM mutation affects only one subtree, only that subtree is re-laid-out. The rest of the layout tree is cached.
- **Dirty regions.** The cell buffer tracks which regions have changed since the last frame. Only changed regions are flushed to the terminal.
- **Lazy image decoding.** Images are decoded only when they are about to be scrolled into view.
- **Pre-compiled CSS.** Parsed stylesheets are cached per URL. They are not re-parsed on re-render.
- **Selector indexing.** The CSS selector engine builds an index of class names and IDs at parse time, enabling O(1) matching for the most common selector types.
- **No allocation in the hot path.** The cell buffer painter is designed to work without heap allocation during the render pass. All temporary state fits in stack-allocated arrays.

**Target benchmarks (to be established in Phase 1):**
- Time-to-first-paint (Wikipedia article): < 500ms on a 1GHz CPU with a 10Mbps connection
- Memory usage (5 tabs open): < 100MB RSS
- Re-render on scroll: < 16ms (60 FPS equivalent for terminal)

---

## 19. Security Model

### Network security

- TLS via `rustls` only — no fallback to OpenSSL, no support for SSL 3.0 / TLS 1.0 / 1.1
- Certificate validation is on by default and cannot be turned off in release builds
- HTTP Strict Transport Security (HSTS) headers are stored and enforced
- Mixed content (HTTP resources on an HTTPS page) are blocked by default

### JavaScript sandbox

- JS runs in Boa's isolated context — no access to the filesystem, environment variables, or OS APIs
- `fetch()` is restricted to HTTP/HTTPS — no `file://`, `data:`, or `blob:` URLs from JS
- No `eval()` on externally supplied strings from within DOM event handlers (configurable)
- JS execution has a configurable CPU time limit per page (default 5 seconds) to prevent infinite loops from hanging the browser

### Privacy

- Third-party cookies are blocked by default
- `Referer` headers are stripped for cross-origin navigations
- `Do-Not-Track: 1` is sent by default (configurable)
- No telemetry, no analytics, no data collection of any kind — ever

---

## 20. Testing Strategy

### Unit tests

Every crate has a `tests/` module with unit tests. Run with `cargo test -p ferrite-<name>`.

### Integration tests

The `tests/integration/` directory contains end-to-end render tests. Each test:
1. Loads a static HTML fixture from `tests/fixtures/`
2. Runs the full pipeline (parse → style → layout → paint)
3. Compares the resulting cell buffer against a golden file

Golden files are checked in to the repository. Changes to golden files require explicit reviewer approval.

### Real-world snapshot tests

Phase 2+ will include a set of "real-world snapshot" tests that fetch live pages (with `MockFetcher` in CI, real network in developer mode) and compare rendered output against expected snapshots. Snapshots are updated with `cargo test --features update-snapshots`.

### Fuzzing

`ferrite-html` and `ferrite-css` are fuzz targets using `cargo-fuzz`. The HTML tokenizer must not panic or produce undefined behavior on any input byte sequence. Fuzzing is run in CI on every merge to main.

### CSS conformance

Long-term, `ferrite-css` will be tested against the W3C CSS Test Suite. Passing percentage is reported in the README.

---

## 21. Contributing

We welcome contributors of all experience levels. Here is how to get started.

### Setting up

```bash
git clone https://github.com/your-username/ferrite
cd ferrite
rustup update stable
cargo build
cargo test
```

Minimum Rust version: **1.75.0 (stable)**. We do not use nightly features.

### Finding something to work on

- Issues labeled **`good first issue`** are explicitly sized for new contributors
- Issues labeled **`help wanted`** are higher priority but may need more context
- The roadmap above shows what is planned for each phase — pick a Phase 1 or 2 item and open an issue before starting to avoid duplicate work

### Before opening a pull request

1. Run `cargo fmt --all` — code style is enforced by `rustfmt`
2. Run `cargo clippy --all -- -D warnings` — no new clippy warnings
3. Run `cargo test --all` — all tests pass
4. If adding a new feature, add tests
5. If changing behavior, update relevant golden files with a clear explanation

### Pull request process

1. Fork the repository and create a branch: `git checkout -b feat/ferrite-css-inheritance`
2. Make your changes with clear, atomic commits
3. Open a PR against `main` with a description that explains *what* and *why*
4. Link to the relevant issue(s)
5. A maintainer will review within 7 days

### Commit message format

```
<layer>: <short imperative description>

<optional longer description>

Closes #<issue number>
```

Examples:
```
ferrite-css: implement specificity scoring for class selectors
ferrite-dom: fix arena NodeId overflow on documents with >4B nodes
ferrite-layout: add word-wrap support for CJK characters
```

### What we do NOT accept

- Introductions of `unsafe` code without a detailed safety comment and reviewer approval
- External C/C++ dependencies in the critical rendering path
- Breaking changes to public crate APIs without an RFC
- Code that does not pass `clippy --deny warnings`

---

## 22. Governance

Ferrite is maintained by its founder and a growing group of core contributors.

**Founder:** The project creator — has final say on architectural decisions.

**Core contributors:** Contributors who have merged 5+ substantial PRs may be invited to become core contributors. Core contributors can merge PRs, triage issues, and vote on RFCs.

**RFC process:** Any change that affects a public API, the workspace structure, or a fundamental architectural decision requires an RFC. RFCs are opened as GitHub issues with the `rfc` label, remain open for 14 days for discussion, and are merged by consensus.

**Code of Conduct:** This project follows the [Contributor Covenant v2.1](https://www.contributor-covenant.org/version/2/1/code_of_conduct/). Harassment, discrimination, and bad-faith participation are not tolerated.

---

## 23. Comparison with Existing Terminal Browsers

| Feature | Ferrite | Lynx | w3m | ELinks | Browsh |
|---------|---------|------|-----|--------|--------|
| Language | Rust | C | C | C | Go+JS |
| Memory safety | ✓ | ✗ | ✗ | ✗ | Partial |
| JavaScript | Boa (Phase 3) | ✗ | ✗ | Optional | Full (Firefox) |
| TLS | rustls (pure Rust) | OpenSSL | OpenSSL | OpenSSL | Firefox |
| CSS support | Full cascade (Phase 2) | Very limited | Limited | Limited | Full (Firefox) |
| Images | Sixel/Kitty (opt-in) | ✗ | Sixel | ✗ | Unicode blocks |
| Tabs | ✓ | ✗ | ✗ | ✓ | ✓ |
| Requires installed browser | ✗ | ✗ | ✗ | ✗ | ✓ (Firefox) |
| External engine deps | None (goal) | libssl | libssl/libgc | libssl | Firefox |
| Low-spec hardware | ✓ | ✓ | ✓ | ✓ | ✗ |

---

## 24. References & Inspiration

### Specifications
- [WHATWG HTML Living Standard](https://html.spec.whatwg.org/multipage/) — HTML5 parsing, the DOM API
- [WHATWG Encoding Standard](https://encoding.spec.whatwg.org/) — charset detection and decoding
- [CSS Snapshot 2023](https://www.w3.org/TR/css-2023/) — the full set of CSS specifications in scope
- [CSS Cascading and Inheritance Level 4](https://www.w3.org/TR/css-cascade-4/) — the cascade algorithm
- [CSS Inline Layout Module Level 3](https://www.w3.org/TR/css-inline-3/) — inline formatting contexts
- [CSS Flexbox Level 1](https://www.w3.org/TR/css-flexbox-1/) — Flexbox layout
- [CSS Grid Layout Level 1](https://www.w3.org/TR/css-grid-1/) — Grid layout
- [Unicode Bidirectional Algorithm (UAX #9)](https://unicode.org/reports/tr9/) — bidi text
- [Unicode Line Breaking Algorithm (UAX #14)](https://unicode.org/reports/tr14/) — line break opportunities
- [ECMAScript 2024 (ES15) Specification](https://tc39.es/ecma262/) — JavaScript language spec
- [Fetch Living Standard](https://fetch.spec.whatwg.org/) — the fetch() API

### Prior art and learning resources
- [Matt Brubeck's "Let's build a browser engine!" series](https://limpet.net/mbrubeck/2014/08/08/toy-layout-engine-1.html) — the foundational tutorial for small browser engines in Rust
- [Servo](https://github.com/servo/servo) — Mozilla's experimental Rust browser engine; source of `html5ever`, `cssparser`, and `selectors`
- [Gosub Browser Engine](https://github.com/gosub-browser/gosub-engine) — another pure-Rust browser engine project
- [Browsh](https://github.com/browsh-org/browsh) — the headless Firefox terminal browser that proved the concept
- [Boa](https://github.com/boa-dev/boa) — the pure-Rust JavaScript engine powering Phase 1-3 JS support
- [Taffy](https://github.com/DioxusLabs/taffy) — the Rust Flexbox/Grid layout library from Dioxus Labs
- [How Browsers Work](https://web.dev/articles/howbrowserswork) — Tali Garsiel's seminal overview of browser internals

---

*This document is the living architectural foundation of Ferrite. It will be updated as the project evolves. If you are reading this and want to build it — welcome. Open an issue, pick a layer, and start.*

---

**License:** Ferrite is dual-licensed under [MIT](LICENSE-MIT) and [Apache 2.0](LICENSE-APACHE). Contributions are accepted under the same terms.
