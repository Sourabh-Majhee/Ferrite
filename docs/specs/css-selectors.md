# CSS Selectors Specification Reference

**Specification:** [CSS Selectors Level 3](https://www.w3.org/TR/selectors-3/) and [Level 4](https://www.w3.org/TR/selectors-4/)

## Overview

CSS selectors are patterns used to select elements in the DOM based on their type, attributes, relationships, and pseudo-states. This document outlines which selectors are implemented in Ferrite.

## Selector Types Implemented

### ✅ Fully Implemented

#### Type Selectors
- `div`, `p`, `a`, etc. — Select elements by tag name
- `*` — Universal selector (all elements)

**Code:** `ferrite-css/src/selector.rs::match_type_selector()`

#### Class and ID Selectors
- `.classname` — Select elements with class attribute
- `#idname` — Select elements with id attribute

**Code:** `ferrite-css/src/selector.rs::match_class()`, `match_id()`

#### Attribute Selectors
- `[attr]` — Element has attribute
- `[attr="value"]` — Exact match
- `[attr~="value"]` — Word match (space-separated)
- `[attr|="value"]` — Prefix match (with dash)
- `[attr^="value"]` — Prefix match (any)
- `[attr$="value"]` — Suffix match
- `[attr*="value"]` — Substring match
- `[attr="value" i]` — Case-insensitive flag (all variants)

**Code:** `ferrite-css/src/selector.rs::match_attribute()`

#### Pseudo-Classes
- `:root` — Document root element
- `:empty` — No child nodes
- `:first-child` — First child of parent
- `:last-child` — Last child of parent
- `:nth-child(n)`, `:nth-child(2n+1)` — Nth child by formula
- `:nth-last-child(n)` — Nth child from end
- `:only-child` — Only child of parent
- `:link` — Unvisited link
- `:visited` — Visited link
- `:enabled`, `:disabled` — Form input states
- `:checked` — Checked checkbox/radio
- `:not(selector)` — Negation (single selector only in Level 3)

**Code:** `ferrite-css/src/selector.rs::match_pseudo_class()`

#### Pseudo-Elements
- `::before` — Insert content before element (CSS representation only; no actual rendering)
- `::after` — Insert content after element
- `::first-line` — First line of block
- `::first-letter` — First letter of block
- `::selection` — Selected text (ignored in terminal)

**Code:** `ferrite-css/src/selector.rs::match_pseudo_element()`

**Status:** ⚠️ Recognized but not fully rendered. Terminal browser cannot easily insert pseudo-element content.

#### Combinators
- `selector1 selector2` — Descendant combinator (selector2 descendant of selector1)
- `selector1 > selector2` — Child combinator (selector2 direct child of selector1)
- `selector1 + selector2` — Adjacent sibling combinator
- `selector1 ~ selector2` — General sibling combinator

**Code:** `ferrite-css/src/selector.rs::match_combinator()`

#### Multiple Selectors
- `selector1, selector2` — OR logic (match either)
- `selector1selector2` — AND logic (concatenate selectors)

**Code:** `ferrite-css/src/selector.rs::matches()`

## Selector Syntax

**Fully implemented per [CSS Selectors Level 3 § 4](https://www.w3.org/TR/selectors-3/#grammar):**

- Whitespace normalization
- String parsing with escapes (`\"`, `\\`, etc.)
- Identifier tokenization
- Namespace prefixes (recognized but not used; terminal has no namespaces)

**Code:** `cssparser` crate (vendored) + `ferrite-css/src/parser.rs`

## Pseudo-Classes Not Implemented

### ❌ Interaction Pseudo-Classes

- `:hover` — Would require mouse tracking
- `:active` — Would require tracking active clicks
- `:focus` — Would require terminal focus tracking
- `:focus-visible` — Deferred

**Reason:** Terminal doesn't have continuous event streaming; would require full rerender per mouse move.

**Planned:** Phase 2 (limited implementation)

### ❌ Form Pseudo-Classes (Limited)

- `:invalid`, `:valid` — Form validation state
- `:in-range`, `:out-of-range` — Input range validation
- `:required`, `:optional` — Form field requirement
- `:read-only`, `:read-write` — Input mutability

**Reason:** Requires form validation engine

**Planned:** Phase 2

### ❌ Language Pseudo-Classes

- `:lang(language)` — Match by language

**Reason:** Terminal has no language context

**Planned:** Phase 3

### ❌ Structural Pseudo-Classes (Advanced)

- `:nth-of-type(n)` — Nth child of its type
- `:nth-last-of-type(n)` — Nth child of its type from end
- `:first-of-type` — First child of its type
- `:last-of-type` — Last child of its type
- `:only-of-type` — Only child of its type

**Status:** 🟡 Implemented but not tested

### ❌ Target Pseudo-Classes

- `:target` — Element targeted by fragment identifier
- `:target-within` — Deferred

**Reason:** Requires history/navigation support

**Planned:** Phase 2

## Pseudo-Elements Not Implemented

- `::before`, `::after` — Recognized but pseudo-element content (`content` property) not rendered
- `::first-line`, `::first-letter` — Difficult in terminal (no clear line boundaries)
- `::marker` — List marker (custom list-style-type)
- `::placeholder` — Input placeholder text
- `::selection` — Selected text (terminal controls selection)
- `::cue` — Video captions

## Case Sensitivity

**Implemented per spec:**

- **Tag names:** Case-insensitive (HTML allows `<DIV>` or `<div>`)
- **Attribute names:** Case-insensitive (HTML allows `CLASS` or `class`)
- **Attribute values:** Case-sensitive by default, but `[attr="value" i]` flag makes case-insensitive
- **Class names:** Case-sensitive (CSS is case-sensitive)
- **IDs:** Case-sensitive

**Code:** `ferrite-css/src/selector.rs` (with case handling per element type)

## Performance Optimization

**Selector Indexing:**

The CSS engine indexes rules by selector type for faster matching:

```rust
struct SelectorIndex {
    by_id: HashMap<String, Vec<Rule>>,      // Rules with ID selectors
    by_class: HashMap<String, Vec<Rule>>,   // Rules with class selectors
    by_type: HashMap<String, Vec<Rule>>,    // Rules with type selectors
    by_attribute: HashMap<String, Vec<Rule>>, // Rules with attribute selectors
    universal: Vec<Rule>,                     // Rules with * selector
}
```

This allows O(1) lookup of applicable rules, then O(n) matching of individual selectors.

**Code:** `ferrite-css/src/cascade.rs::SelectorIndex`

## DOM Querying

The DOM layer exposes selector-based querying:

```rust
// Query a single element
let element: Option<NodeId> = dom.query_selector(".highlighted");

// Query all matching elements
let elements: Vec<NodeId> = dom.query_selector_all(".highlighted");

// Used by CSS to match rules
let matches: bool = selector.matches(&dom, element);
```

**Code:** `ferrite-dom/src/selector.rs`

## Testing

Selector tests verify matching and parsing:

```bash
# Test selector parsing
cargo test -p ferrite-css selector::parse

# Test selector matching
cargo test -p ferrite-css selector::match

# Test specificity (derived from selectors)
cargo test -p ferrite-css specificity

# All CSS tests
cargo test -p ferrite-css
```

## References

- [CSS Selectors Level 3](https://www.w3.org/TR/selectors-3/)
- [CSS Selectors Level 4](https://www.w3.org/TR/selectors-4/)
- [MDN: CSS Selectors](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Selectors)
- [Web Platform Tests: Selectors](https://github.com/web-platform-tests/wpt/tree/master/css/selectors)
