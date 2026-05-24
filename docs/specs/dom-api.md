# DOM API Specification Reference

**Specification:** [WHATWG DOM Living Standard](https://dom.spec.whatwg.org/)

## Overview

The DOM (Document Object Model) is the programmatic interface to the HTML document. This document outlines which parts of the DOM specification are implemented in Ferrite.

## Node Types

**Fully implemented per § 3 (Node):**

- ✅ `ELEMENT_NODE` — HTML elements
- ✅ `TEXT_NODE` — Text content
- ✅ `COMMENT_NODE` — HTML comments
- ✅ `DOCUMENT_NODE` — Root document
- ✅ `DOCUMENT_TYPE_NODE` — `<!DOCTYPE>`
- 🟡 `CDATA_SECTION_NODE` — Not used (HTML, not XML)
- ❌ `ENTITY_NODE` — Not used (deprecated, XML-only)
- ❌ `ENTITY_REFERENCE_NODE` — Not used (deprecated)
- ❌ `PROCESSING_INSTRUCTION_NODE` — Not used
- ❌ `NOTATION_NODE` — Not used (deprecated)

**Code:** `ferrite-dom/src/node.rs::NodeKind`

## Node Interface

**Status:** 🟡 Partially implemented

### ✅ Properties Implemented

- `nodeName` — Name of the node (tag name, `#text`, `#comment`, `#document`, etc.)
- `nodeValue` — Content of text/comment nodes (null for element nodes)
- `nodeType` — Type constant (ELEMENT_NODE, TEXT_NODE, etc.)
- `parentNode` — Parent node reference (Option<NodeId>)
- `childNodes` — Collection of child nodes
- `firstChild`, `lastChild` — First/last child (Option<NodeId>)
- `previousSibling`, `nextSibling` — Sibling references
- `ownerDocument` — Document node reference

**Code:** `ferrite-dom/src/node.rs`

### ✅ Methods Implemented

- `appendChild(newChild)` — Append child node
- `removeChild(child)` — Remove child node
- `replaceChild(newChild, oldChild)` — Replace child node
- `insertBefore(newChild, refChild)` — Insert before reference node
- `contains(node)` — Check if node is descendant
- `compareDocumentPosition(other)` — Compare node positions
- `cloneNode(deep)` — Deep or shallow copy of node tree

**Code:** `ferrite-dom/src/mutation.rs`

### ❌ Not Implemented (Deferred)

- `normalize()` — Merge adjacent text nodes
- `isEqualNode(other)` — Deep equality check
- `getRootNode()` — Get document or shadow root
- Mutation events (use MutationObserver instead)

**Planned:** Phase 2

## Element Interface

**Status:** ✅ Mostly implemented (§ 4)

### ✅ Attributes Implemented

- `tagName` — Tag name in uppercase (e.g., `"DIV"`)
- `id` — ID attribute value
- `className` — Class attribute value (space-separated)
- `classList` — DOMTokenList for class manipulation

**Code:** `ferrite-dom/src/element.rs`

### ✅ Methods Implemented

- `getAttribute(qualifiedName)` — Get attribute value
- `setAttribute(qualifiedName, value)` — Set attribute value
- `removeAttribute(qualifiedName)` — Remove attribute
- `hasAttribute(qualifiedName)` — Check if attribute exists
- `getAttributeNames()` — Get all attribute names
- `toggleAttribute(qualifiedName)` — Toggle boolean attribute
- `querySelector(selectors)` — Find first matching element
- `querySelectorAll(selectors)` — Find all matching elements
- `getElementsByTagName(qualifiedName)` — Get elements by tag (live collection)
- `getElementsByClassName(classNames)` — Get elements by class (live collection)

**Code:** `ferrite-dom/src/element.rs`, `ferrite-dom/src/selector.rs`

### 🟡 Properties Partially Implemented

- `innerHTML` — Get/set inner HTML (parsing/serialization)
  - ✅ Getter implemented
  - 🟡 Setter uses fragment parsing (deferred)
  
- `outerHTML` — Get/set outer HTML
  - ✅ Getter implemented
  - ❌ Setter deferred

- `textContent` — Get/set text content
  - ✅ Fully implemented

**Code:** `ferrite-dom/src/element.rs`

### ❌ Not Implemented

- `style` property and CSSStyleDeclaration
  - **Reason:** Requires CSS OM; works via attribute for now
  - **Planned:** Phase 2
  
- `dataset` property and DOMStringMap
  - **Reason:** Low priority
  - **Planned:** Phase 3

- Slot-related properties (Web Components)
  - **Reason:** Not applicable to terminal browser
  - **Planned:** Not planned

- `scrollTop`, `scrollLeft`, `scrollWidth`, `scrollHeight`
  - **Status:** Partially implemented (computed from layout)
  - **Code:** `ferrite-render/src/scroll.rs`

## Document Interface

**Status:** 🟡 Partially implemented (§ 3)

### ✅ Properties Implemented

- `documentElement` — Root element
- `body` — Body element (if exists)
- `head` — Head element (if exists)
- `title` — Document title from `<title>` tag
- `charset` — Encoding charset

**Code:** `ferrite-dom/src/document.rs`

### ✅ Methods Implemented

- `createElement(tagName)` — Create element node
- `createTextNode(data)` — Create text node
- `createComment(data)` — Create comment node
- `createDocumentFragment()` — Create document fragment
- `getElementById(elementId)` — Get element by ID
- `querySelectorAll(selectors)` — Query all
- `getElementsByTagName(tagName)` — Get by tag (live)
- `getElementsByClassName(classNames)` — Get by class (live)

**Code:** `ferrite-dom/src/document.rs`

### ❌ Not Implemented

- `getSelection()` — User text selection
  - **Reason:** Terminal has no native selection API
  - **Planned:** Limited support Phase 2

- `execCommand()` — Execute editing commands
  - **Reason:** Not applicable to terminal
  - **Planned:** Not planned

- Event handlers (`onload`, `onunload`, etc.)
  - **Status:** 🟡 Limited support
  - **Code:** `ferrite-dom/src/events.rs`

## Event Interface

**Status:** 🟡 Partially implemented (§ 5)

### ✅ Implemented

- `Event` interface with:
  - `type` — Event type name
  - `target` — Target node
  - `currentTarget` — Currently handling node
  - `eventPhase` — Capture, target, or bubble phase
  - `bubbles` — Whether event bubbles
  - `cancelable` — Whether event is cancelable
  - `preventDefault()` — Cancel default action
  - `stopPropagation()` — Stop propagation
  - `stopImmediatePropagation()` — Stop all propagation

- Event types:
  - ✅ `click` — Mouse/touch click
  - ✅ `change` — Form input changed
  - ✅ `input` — Text input
  - ✅ `load` — Resource loaded
  - ✅ `unload` — Leaving page
  - 🟡 `submit` — Form submitted (basic)
  - ❌ `mousemove`, `mouseover`, `mouseout` — Deferred

### ✅ EventTarget Interface

- `addEventListener(type, listener)` — Add event listener
- `removeEventListener(type, listener)` — Remove listener
- `dispatchEvent(event)` — Dispatch event

**Code:** `ferrite-dom/src/events.rs`

## Collections

**Status:** 🟡 Partially implemented (§ 7)

### ✅ NodeList
- Created by `childNodes`, `querySelectorAll()`
- Static or live depending on source
- Iterable (via `[Symbol.iterator]`)

**Code:** `ferrite-dom/src/collection.rs::NodeList`

### 🟡 HTMLCollection
- Created by `getElementsByTagName()`, `getElementsByClassName()`
- Live collections (update as DOM changes)
- Partial implementation

**Code:** `ferrite-dom/src/collection.rs::HTMLCollection`

### ⚠️ DOMTokenList
- `classList` property
- Methods: `add()`, `remove()`, `toggle()`, `contains()`, `replace()`

**Code:** `ferrite-dom/src/token_list.rs`

## Range Interface

**Status:** ❌ Not implemented

Deferred to Phase 2. Would support:
- `Selection` of text ranges
- `Range` manipulation
- Useful for find-in-page feature

**Spec:** § 5.2

## Tree Walking

**Status:** 🟡 Partially implemented

- `TreeWalker` — Traverse DOM with filters
  - ❌ Not implemented (can use manual recursion)
- `NodeIterator` — Iterate over matching nodes
  - ❌ Not implemented

**Planned:** Phase 2

## Testing

DOM API tests are in:

```bash
# Test node operations
cargo test -p ferrite-dom node

# Test element queries
cargo test -p ferrite-dom element

# Test events
cargo test -p ferrite-dom event

# All DOM tests
cargo test -p ferrite-dom
```

## References

- [WHATWG DOM Living Standard](https://dom.spec.whatwg.org/)
- [MDN: Document Object Model](https://developer.mozilla.org/en-US/docs/Web/API/Document_Object_Model)
- [W3C DOM Level 3 Core](https://www.w3.org/TR/DOM-Level-3-Core/)
