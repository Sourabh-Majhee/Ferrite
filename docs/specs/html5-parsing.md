# HTML5 Parsing Specification Reference

**Specification:** [WHATWG HTML Living Standard - Parsing Section](https://html.spec.whatwg.org/#parsing)

## Overview

The WHATWG HTML parsing algorithm defines a deterministic state machine for converting raw bytes into a DOM tree. This document outlines which parts of the spec are implemented in Ferrite and any deviations.

## Implementation Status

### Phase 1: Core Parsing (Current)

#### ✅ Fully Implemented (via html5ever)

- **Tokenization** — The tokenizer's 17 states and 32 character reference entities
- **Tree Construction** — The tree construction phase with all insertion modes
- **Error Recovery** — Malformed HTML is handled gracefully, matching browser behavior
- **Encoding Detection** — BOM sniffing, HTTP charset, meta pragma, prescan algorithm
- **Named Character References** — All 2000+ HTML5 entities
- **Numeric Character References** — Decimal and hexadecimal references

**Spec Sections:**
- § 12.2.1 — Tokenization overview
- § 12.2.4 — Token insertion (complete state machine)
- § 12.2.5 — Tree construction (complete algorithm)
- § 8.2.1 — Encodings (detection and conversion)

#### 🟡 Partially Implemented

- **Scripting during parsing** — Scripts are inserted into the DOM but do not block parsing. We process them after the DOM is complete, which differs from the synchronous model.
  - **Reason:** Terminal browser doesn't have progressive rendering; a batched approach is simpler and safer.
  - **Spec:** § 12.2.4.3 (Script processing)
  - **Code:** `ferrite-app/src/event_loop.rs`

- **Style sheets during parsing** — CSS is parsed but does not affect tree construction (no CSSOM during parse).
  - **Reason:** Simpler model; stylesheet processing happens after DOM creation.
  - **Spec:** § 12.2.4.1 (Style processing)
  - **Code:** `ferrite-css/src/lib.rs`

#### ❌ Not Implemented (Deferred)

- **Fragment Parsing** — `innerHTML` property and `parse` method on fragments
  - **Planned:** Phase 2
  - **Spec:** § 12.4 (Parsing HTML fragments)
  - **Reason:** Not critical for initial MVP; can be added incrementally

- **Template Element Processing** — `<template>` tag support
  - **Planned:** Phase 2
  - **Spec:** § 4.11.2 (The `template` element)
  - **Reason:** Advanced feature; primarily useful for frameworks

- **Foreign Content** — SVG and MathML parsing
  - **Planned:** Phase 3+
  - **Spec:** § 12.2.5.3 (Parsing algorithm - foreign content)
  - **Reason:** Renders in terminal; deferred

## Encoding Detection

**Fully implemented per § 8.2.1:**

1. BOM sniffing (UTF-8, UTF-16 LE/BE)
2. HTTP `Content-Type` header charset parameter
3. `<meta charset="...">` pragma
4. `<meta http-equiv="Content-Type" charset=...>` pragma
5. Prescan algorithm (first 1024 bytes)
6. Fallback: UTF-8

**Code:** `ferrite-html/src/encoding.rs`

## Error Handling

The parser handles malformed HTML following the WHATWG algorithm:

- ✅ Unclosed tags → auto-closed per insertion mode
- ✅ Misnested tags → reconstructed properly
- ✅ Unknown elements → treated as generic elements
- ✅ Invalid attributes → ignored or sanitized

**Code:** `html5ever` (vendored or as dependency)

## Special Cases

### XML vs. HTML

Ferrite only implements HTML5 parsing, not XML. XHTML documents are parsed as HTML.

- **Reason:** Terminal browser has no need for XML; HTML5 can represent all content
- **Spec:** § 12.1 (vs. § 12.3 XML parsing)

### Character Classes

The tokenizer uses the following character classes (per Unicode standard):

- **Whitespace:** U+0009, U+000A, U+000C, U+000D, U+0020
- **Upper-case ASCII:** A-Z
- **Lower-case ASCII:** a-z
- **Digits:** 0-9

**Code:** `html5ever` handles this; uses `char::is_whitespace()` and ASCII checks

## Performance Considerations

- **Streaming parse:** Could be implemented for very large documents
- **Current approach:** Parse entire document at once (simpler, sufficient for typical pages)
- **Alternative:** Could implement lazy parsing for frames/iframes (deferred)

## Testing

The HTML parser is tested against:

- `html5lib-tests` suite (all tokenizer tests)
- `html5lib-tests` suite (all tree construction tests)
- Ferrite-specific integration tests in `tests/fixtures/`

**Run tests:**
```bash
cargo test -p ferrite-html
```

## References

- [WHATWG HTML Living Standard](https://html.spec.whatwg.org/)
- [html5ever documentation](https://github.com/servo/html5ever)
- [html5lib test suite](https://github.com/html5lib/html5lib-tests)
