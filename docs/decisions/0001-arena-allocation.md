# ADR-0001: Use Arena Allocation for DOM Tree

## Status
**Accepted**

## Context

The DOM tree is a bidirectional graph: each child node maintains a reference to its parent, and each parent maintains references to all its children. Additionally, sibling nodes need to reference each other in a linked-list structure.

In a memory-safe language like Rust, this creates a fundamental ownership problem. Rust's borrow checker prevents creating cycles where multiple owners would be needed. Traditional approaches using `Rc<RefCell<Node>>` lead to runtime overhead and make reasoning about data flow difficult.

The DOM tree will be accessed concurrently:
- Layout engine reads node properties
- CSS engine reads and updates computed styles
- JavaScript engine can mutate the tree at runtime
- Rendering engine reads the positioned layout

We need a solution that:
- Eliminates Rust ownership issues
- Provides efficient node access and manipulation
- Allows zero-copy reads from multiple systems
- Maintains memory safety without runtime overhead

## Decision

We will implement DOM storage using an **arena allocation pattern**. All DOM nodes are stored in a single `Vec<NodeData>` arena. References between nodes use typed indices (`NodeId`) rather than Rust references.

```rust
pub struct Arena {
    nodes: Vec<NodeData>,
}

#[derive(Clone, Copy, PartialEq, Eq, Hash, Debug)]
pub struct NodeId(u32);

pub struct NodeData {
    pub kind: NodeKind,
    pub parent: Option<NodeId>,
    pub first_child: Option<NodeId>,
    pub last_child: Option<NodeId>,
    pub next_sibling: Option<NodeId>,
    pub prev_sibling: Option<NodeId>,
}
```

## Rationale

### Why Not Rc<RefCell<T>>?

The `Rc<RefCell<T>>` approach creates runtime overhead:
- Every node access requires a runtime borrow check
- Cloning an `Rc` is thread-unsafe for mutation
- Adds 48+ bytes of overhead per node (atomic refcount, vtable)
- Makes performance profiling difficult

### Why Not Indices?

Direct index-based approaches work but lack type safety and make refactoring difficult. By using a newtype wrapper `NodeId(u32)`, we get:
- Zero-cost abstraction (compiles to just `u32`)
- Type safety (cannot accidentally pass a different index type)
- Self-documenting code

### Why Not Generational Indices?

Generational indices (GGen) add a generation counter to detect use-after-free. We skip this for now because:
- The DOM tree lifetime is the page lifetime — we don't deallocate individual nodes
- Bulk deallocation of the entire document is O(1) by dropping the arena
- If needed later, it's a localized change to `NodeId` representation

## Consequences

### ✅ Positive
- **No ownership issues** — Bidirectional references work naturally
- **O(1) allocation** — Pushing to the vec is constant time
- **O(1) deallocation** — Drop the arena to free the entire document
- **Cache-friendly** — Sequential memory layout improves CPU cache hits
- **Zero runtime overhead** — `NodeId` is just a `u32` at runtime
- **Supports concurrency** — Multiple systems can read the arena safely while holding `Arc<Arena>`
- **Easy serialization** — The arena is just a `Vec`, trivial to serialize

### ❌ Negative
- **Cannot hold Rust references** — Code must pass `NodeId` around, not `&Node`
- **Bounds checking** — Accessing nodes requires checking the index is valid
- **Harder debugging** — Stack traces show indices, not reference pointers
- **Requires careful lifetime management** — Arena must outlive all `NodeId`s
- **No easy mutation isolation** — Cannot mark specific nodes as borrowed

### ⚠️ Neutral
- **Slightly more verbose** — `arena[node_id].parent` instead of `node.parent`
- **Different mental model** — Developers need to understand arena pattern

## Alternatives Considered

### 1. Rc<RefCell<Node>>
**Rejected** — Runtime overhead, difficult to debug, not cache-efficient.

### 2. Raw pointers with unsafe
**Rejected** — Defeats the purpose of using Rust; extremely difficult to maintain and verify correctness.

### 3. Parent pointers only, no child pointers
**Rejected** — Would require expensive parent-to-child traversals; quadratic complexity for many operations.

### 4. ECS-style (Components are separate vecs)
**Rejected** — Adds complexity; our DOM is primarily hierarchical, not relational.

## Migration Path

As we refactor systems to work with the arena:

1. **Phase 1** — DOM storage only (current)
2. **Phase 2** — CSS engine reads arena (computed styles stored separately)
3. **Phase 3** — Layout engine uses arena
4. **Phase 4** — JavaScript bindings use arena
5. **Phase 5** — Rendering engine reads arena

Each system maintains its own data structures (styles, layout boxes, render data) but all index into the same DOM arena.

## Implementation Notes

- All node IDs are `u32` for space efficiency (supports 4B nodes per document)
- The arena is stored in `Arc<Arena>` to allow cheap cloning across systems
- No `unsafe` code required — the `Vec` bounds checking is sufficient
- Optional: Could add generation counters if use-after-free becomes a problem

## References

- [Rustonomicon: Managing Complex Data with Arenas](https://doc.rust-lang.org/nomicon/)
- [Gaffo: Arena-allocated data structures](https://gaffo.rs/)
- [Bevy Engine: Using Generational Indices](https://bevyengine.org/)
- [WHATWG DOM Spec: Node Interface](https://dom.spec.whatwg.org/#interface-node)
