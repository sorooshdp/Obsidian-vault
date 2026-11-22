---
created: 2026-11-10 13:54
updated: 2026-11-10 13:54
type: moc
status: mature
tags:
  - rust
aliases:
  - Rust MOC
  - Rust Index
---

# Rust Knowledge Hub

## 1. Foundations & Core Syntax
- [[Rust - getting started]] — Cargo, `rustc`, and project anatomy
- [[Rust common Concepts]] — Mutability, scalar/compound types, expressions, control flow
- [[Rust - Ownership]] — Move semantics, borrowing (`&` / `&mut`), slices, and RAII
- [[The stack and the heap]] — Memory allocation fundamentals

## 2. Type System, Data Modeling & Patterns
- [[Rust Structs]] — Named/tuple/unit structs, `impl` blocks, and method receivers
- [[Rust Enums and Pattern Matching]] — Algebraic data types, `Option<T>`, `match`, and `if let`
- [[Rust Patterns and Matching]] — Refutability, destructuring, match guards, and `@` bindings
- [[Rust Generic Types, Traits, and Lifetimes]] — Monomorphization, trait bounds, orphan rule, and lifetime annotations

## 3. Architecture, Collections & Error Handling
- [[Rust Modules]] — Packages, crates, visibility (`pub`), `use`, and file hierarchy
- [[Rust Common Collections]] — Heap collections: `Vec<T>`, UTF-8 `String`, and `HashMap<K, V>` (SipHash)
- [[Rust Error Handling]] — `panic!` vs. `Result<T, E>`, `?` propagation, and custom validation types

## 4. Testing, I/O & Tooling
- [[Rust Tests]] — Unit vs. integration testing, `#[cfg(test)]`, and `#[should_panic]`
- [[Rust IO Project]] — Building `minigrep`: CLI args, TDD, `stderr`, and separation of concerns
- [[Rust Crates.io]] — Release profiles, `rustdoc` (`///`), `pub use`, and Cargo Workspaces

## 5. Memory Primitives, Concurrency & Advanced Rust
- [[Rust Functional Features]] — Closures (`Fn`, `FnMut`, `FnOnce`), lazy iterators, and zero-cost abstractions
- [[Rust Smart Pointers]] — `Box<T>`, `Deref` coercion, `Drop`, `Rc<T>`, `RefCell<T>`, and `Weak<T>`
- [[Rust Fearless Concurrency]] — OS threads (`spawn`), `mpsc` channels, `Arc<Mutex<T>>`, `Send` & `Sync`
- [[Rust OOP]] — Trait objects (`dyn`), dynamic dispatch, vtables, and the Type-State pattern
- [[Rust Advanced Features]] — Unsafe Rust, advanced traits/types, and macros
- [[Rust Appendices]] — Keywords, overloadable operators, derivable traits, and procedural macros
- [[Foreign function interface (FFI)]] — Cross-language ABI boundaries and `extern "C"`
