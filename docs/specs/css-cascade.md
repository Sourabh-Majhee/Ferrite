# CSS Cascade and Inheritance Specification Reference

**Specification:** [CSS Cascading and Inheritance Level 4](https://www.w3.org/TR/css-cascade-4/)

## Overview

The CSS cascade algorithm defines how conflicting style declarations are resolved and how properties are inherited from parent to child elements. This document outlines Ferrite's implementation.

## Cascade Algorithm

**Fully implemented per § 6 (Cascading):**

The cascade resolves all applicable declarations in the following order:

1. **Origin** (lowest to highest precedence)
   - User-agent stylesheet (browser default styles)
   - Author stylesheet (website CSS)
   - Inline styles (`style` attribute)
   - `!important` flag reverses precedence

2. **Specificity** (within same origin)
   - (a, b, c) where:
     - a = number of ID selectors (`#id`)
     - b = number of class selectors (`.class`), attribute selectors (`[...]`), pseudo-class selectors (`:hover`)
     - c = number of type selectors (`div`), pseudo-element selectors (`::before`)
   - Compare tuples lexicographically

3. **Order** — Later declarations win on equal specificity

**Code:** `ferrite-css/src/cascade.rs`

## Specificity Calculation

**Fully implemented per § 6.1 (Calculating specificity):**

```rust
// Example: a.link:hover#primary
// ID selectors: #primary = 1
// Class/pseudo-class: .link, :hover = 2
// Type: a = 1
// Specificity: (1, 2, 1)

// Example: div p.highlight
// Type: div, p = 2
// Class: .highlight = 1
// Specificity: (0, 1, 2)

// (1, 2, 1) > (0, 1, 2) because 1 > 0 in first component
```

**Code:** `ferrite-css/src/cascade.rs::calculate_specificity()`

## Property Inheritance

**Fully implemented per § 5 (Inheritance):**

Each CSS property is marked `inherited: true` or `inherited: false`:

- ✅ **Inherited properties** (computed value inherited from parent if not specified)
  - `color`, `font-size`, `font-weight`, `line-height`, `text-align`, `white-space`, `visibility`
  
- ❌ **Non-inherited properties** (initial value used if not specified)
  - `margin`, `padding`, `border`, `background-color`, `width`, `height`, `display`, `position`

**Code:** `ferrite-css/src/properties.rs` — each property struct includes `inherited: bool`

## Specified vs. Computed Values

**Fully implemented per § 3 (Specified values and cascading):**

For each property:

1. **Specified value** — Result of cascade algorithm (may contain relative values like `50%`, `1em`)
2. **Computed value** — Resolved relative values (e.g., `50%` → actual width, `1em` → pixel value)
3. **Used value** — Final value after layout (character cells instead of pixels)

**Code:**
- Cascade: `ferrite-css/src/cascade.rs`
- Computed: `ferrite-css/src/computed.rs`
- Used (terminal-specific): `ferrite-render/src/painter.rs`

## Color Value Resolution

**Spec:** § 5.2 (Computing color values)

**Status:** ✅ Implemented

Colors are specified in CSS (e.g., `color: #ff0000; background-color: rgb(255, 0, 0);`) and converted to ANSI terminal colors:

- 16-color mode: Basic colors + bold for bright variants
- 256-color mode: CSS color approximated to nearest 256-color palette index
- Truecolor mode: Full 24-bit RGB sent to terminal (terminal permitting)

**Code:** `ferrite-render/src/ansi.rs::color_to_ansi()`

## Special Values

**Fully implemented:**

- **`inherit`** — Explicitly inherit from parent
- **`initial`** — Use the property's initial value
- **`unset`** — Behave as `inherit` if inherited property, else `initial`

**Status:** ⚠️ Partially implemented
- ✅ `inherit` keyword supported
- ✅ `initial` keyword supported
- 🟡 `unset` keyword deferred

**Code:** `ferrite-css/src/computed.rs`

## Not Implemented (Deferred)

- **CSS Variables** (custom properties, `--my-var`)
  - **Reason:** Would require additional parsing and substitution pass
  - **Planned:** Phase 2
  - **Spec:** § 3.1 (CSS custom properties)

- **Media queries with custom media**
  - **Reason:** Terminal browser has no concept of screen types
  - **Planned:** Minimal support in Phase 2
  - **Spec:** § 13 (Media queries)

- **`revert` and `revert-layer` keywords**
  - **Reason:** Cascade layers not yet implemented
  - **Planned:** Phase 3+
  - **Spec:** § 6.3 (Cascade layers)

- **Conditional at-rules** (`@supports`, `@media`)
  - **Reason:** Simplified for terminal
  - **Planned:** Basic support in Phase 2
  - **Spec:** § 12-13

## Terminal-Specific Adaptations

Ferrite adapts CSS cascade for terminal constraints:

### No Cascading for Terminal Properties

Terminal-specific CSS (not in spec) for controlling scrolling, overflow regions, etc.:

```css
/* Terminal extensions (non-standard) */
element {
  terminal-scroll: auto;        /* Show scrollbar */
  terminal-overflow: visible;   /* Wrap to next line */
}
```

**Code:** `ferrite-css/src/properties.rs` (terminal-specific section)

## Testing

Cascade tests are in:

```bash
# Test cascade algorithm
cargo test -p ferrite-css cascade

# Test specificity calculation
cargo test -p ferrite-css specificity

# Test inheritance
cargo test -p ferrite-css inherit

# All CSS tests
cargo test -p ferrite-css
```

### Test Cases

- Specificity tie-breaking
- `!important` flag handling
- Multi-level property inheritance
- Vendor-prefixed properties (ignored)
- Invalid syntax recovery

## Implementation Notes

### Performance

The cascade algorithm is O(n * m) where:
- n = number of CSS rules
- m = number of DOM elements

Optimizations:

1. **Rule indexing** — Rules sorted by selector type for faster matching
2. **Computed value caching** — Store computed values per element
3. **Invalidation** — Only recompute styles when CSS changes

### Browser Compatibility

Ferrite's cascade implementation is compatible with all major browsers. We follow the W3C specification exactly.

## References

- [CSS Cascading and Inheritance Level 4 Specification](https://www.w3.org/TR/css-cascade-4/)
- [MDN: Cascade](https://developer.mozilla.org/en-US/docs/Web/CSS/Cascade)
- [MDN: Specificity](https://developer.mozilla.org/en-US/docs/Web/CSS/Specificity)
