---
type: concept
title: "Rust Ownership and Borrowing"
created: 2026-06-21
updated: 2026-06-21
tags:
  - rust
  - memory-safety
  - programming
status: seed
related:
  - "[[Cargo Workflow]]"
---

# Rust Ownership and Borrowing

Ownership is Rust's mechanism for **memory safety without a garbage collector**.
The compiler tracks who owns each value and when it goes out of scope, freeing it
deterministically — no GC, no manual `free`, no use-after-free.

> [!key-insight]
> Ownership moves the "when is this freed / who can touch it" question from
> runtime (GC) or the programmer's head (C) into the **type system**, checked at
> compile time at zero runtime cost.

## The three rules

1. **Each value has exactly one owner.**
2. **When the owner goes out of scope, the value is dropped** (freed).
3. **Either one mutable reference OR any number of immutable references** to a
   value at a time — never both.

```rust
let data = String::from("hello"); // `data` owns the string
let moved = data;                 // ownership MOVES to `moved`
// println!("{data}");            // error: `data` no longer valid

let s = String::from("hi");
let r = &s;                       // immutable borrow
println!("{s} {r}");              // both fine — read-only sharing
```

## Borrowing patterns

| Goal | Signature | Why |
|------|-----------|-----|
| Transform + return | `fn f(data: Vec<T>) -> Vec<T>` | take ownership, hand it back |
| Inspect only | `fn f(data: &[T]) -> usize` | immutable borrow, caller keeps ownership |
| Modify in place | `fn f(data: &mut Vec<T>)` | exclusive mutable borrow |
| Independent copy | `data.clone()` | explicit, visible cost |

## Lifetimes

A **lifetime** is the compiler's name for "how long a reference is valid." They
ensure a reference never outlives the data it points to:

```rust
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() { x } else { y }
}
```

Most lifetimes are inferred ("elision"); you only annotate when the compiler can't
prove the relationship itself.

## Why it matters

- **No data races** — the one-mutable-XOR-many-immutable rule prevents two threads
  mutating shared data simultaneously ("fearless concurrency").
- **No use-after-free / double-free** — the borrow checker rejects them at compile
  time.
- **Zero-cost** — all checks happen at compile time; the binary has no GC overhead.

The flip side is a learning curve: the borrow checker rejects patterns that *would*
be unsafe, which feels strict until the mental model clicks.

## Related

- [[Cargo Workflow]] — building, testing, and linting Rust projects
