# Specification References

This directory contains annotated references to web standards that guide Ferrite's implementation. Each file explains which parts of a specification we support, which we defer, and any design decisions that deviate from the spec.

## Why Annotated Specs?

Web standards are complex. Rather than maintain a separate implementation guide, we link directly to the spec sections that justify our code. Comments in the codebase reference these files, creating a chain from code → decision → spec.

## How to Use This Directory

### For Contributors

When implementing a feature:
1. Find the relevant spec file in this directory
2. Read the annotated sections
3. Check what's marked "Implemented", "Deferred", or "Partially Implemented"
4. In your code, add a comment referencing the spec:
   ```rust
   // Per HTML5 spec § 12.2.4.1 (see docs/specs/html5-parsing.md)
   fn process_script() { ... }
   ```

### For Code Reviewers

When reviewing a PR that modifies parsing or rendering:
1. Check the relevant spec file to understand the requirement
2. Verify the PR implements the spec correctly (or documents the deviation)
3. Request changes if the implementation diverges from the spec without justification

## Specification Files

### HTML Parsing

**File:** [html5-parsing.md](html5-parsing.md)  
**Spec:** [WHATWG HTML Living Standard - Parsing Section](https://html.spec.whatwg.org/#parsing)  
**Coverage:** HTML5 tokenizer, tree construction, error handling  
**Implementation:** `ferrite-html` and `html5ever`

### DOM API

**File:** [dom-api.md](dom-api.md)  
**Spec:** [WHATWG DOM Living Standard](https://dom.spec.whatwg.org/)  
**Coverage:** Node, Element, Document interfaces, querying, events  
**Implementation:** `ferrite-dom`

### CSS

**File:** [css-cascade.md](css-cascade.md)  
**Spec:** [CSS Cascading and Inheritance Level 4](https://www.w3.org/TR/css-cascade-4/)  
**Coverage:** Cascade algorithm, specificity, inheritance  
**Implementation:** `ferrite-css`

**File:** [css-selectors.md](css-selectors.md)  
**Spec:** [CSS Selectors Level 3](https://www.w3.org/TR/selectors-3/)  
**Coverage:** Type, class, attribute, pseudo-class selectors; combinators  
**Implementation:** `ferrite-css`

**File:** [css-flexbox.md](css-flexbox.md)  
**Spec:** [CSS Flexible Box Layout Level 1](https://www.w3.org/TR/css-flexbox-1/)  
**Coverage:** Flexbox layout algorithm  
**Implementation:** `ferrite-layout` (via `taffy`)

**File:** [css-grid.md](css-grid.md)  
**Spec:** [CSS Grid Layout Level 1](https://www.w3.org/TR/css-grid-1/)  
**Coverage:** Grid layout algorithm, track sizing, placement  
**Implementation:** `ferrite-layout` (via `taffy`)

### JavaScript

**File:** [ecmascript.md](ecmascript.md)  
**Spec:** [ECMAScript 2021 Language Specification](https://tc39.es/ecma262/)  
**Coverage:** Language syntax, semantics, built-in objects  
**Implementation:** `ferrite-js` (via `boa_engine`)

### Web APIs

**File:** [fetch-api.md](fetch-api.md)  
**Spec:** [WHATWG Fetch Living Standard](https://fetch.spec.whatwg.org/)  
**Coverage:** `fetch()` function, Request/Response, CORS  
**Implementation:** `ferrite-js` (DOM binding)

**File:** [timers-api.md](timers-api.md)  
**Spec:** [HTML Living Standard - Timers Section](https://html.spec.whatwg.org/#timers)  
**Coverage:** `setTimeout()`, `setInterval()`, `clearTimeout()`, `clearInterval()`  
**Implementation:** `ferrite-js` (DOM binding)

## Standards We Don't Implement

Some web standards are not applicable to a terminal browser or are explicitly out of scope:

- **CSS Transforms** — Requires 3D geometry; terminal has no z-axis
- **Animations & Transitions** — Terminal has no continuous rendering; would require full screen redraws per frame
- **Web Fonts** — Terminal is monospace; font-family has no effect
- **Media Queries** — Mostly not implemented; only basic terminal-size queries
- **WebGL** — Not applicable to terminal rendering
- **Service Workers** — Deferred to Phase 3+
- **Web Workers** — Deferred; requires thread-safe DOM bindings

See [ARCHITECTURE.md](../../ARCHITECTURE.md#what-css-properties-we-implement) for the complete scope.

## Spec Deviation Log

When we intentionally deviate from a specification, we document it:

### Example: CSS Color Space

Ferrite uses ANSI 256-color palette by default, not CSS Color Module Level 4 full sRGB.

**Reason:** Terminal color support is limited; we approximate CSS colors to the nearest ANSI color.  
**Spec:** [CSS Color Module Level 4 - Color Matching](https://www.w3.org/TR/css-color-4/#color-matching)  
**Code:** `ferrite-render/src/ansi.rs` (see `ColorMapper`)

## Contributing Spec References

If you implement a feature that follows a spec:

1. Reference the spec in your PR description
2. Add comments in the code linking to `docs/specs/<topic>.md`
3. Update the relevant spec file to mark sections as "Implemented"
4. If significant, create an entry in the Deviation Log

## External Resources

- [Web Hypertext Application Technology Working Group (WHATWG)](https://whatwg.org/)
- [W3C Cascading Style Sheets (CSS)](https://www.w3.org/Style/CSS/)
- [ECMAScript Standards Committee (TC39)](https://tc39.es/)
- [Can I Use](https://caniuse.com/) — Browser compatibility data
- [MDN Web Docs](https://developer.mozilla.org/) — Community documentation
