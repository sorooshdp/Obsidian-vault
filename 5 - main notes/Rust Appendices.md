---
created: 2026-10-18 17:51
updated: 2026-10-18 17:51
type: note
status: seed
tags:
  - rust
aliases: []
---

## Summary


## Notes
In the **Appendices** of *The Rust Programming Language*, the book provides reference guides for essential vocabulary, operators, derivable traits, and macro systems.

---

### **Part 1: Appendix A – Keywords**

Keywords are reserved identifiers that cannot be used as names for variables, functions, parameters, struct fields, modules, crates, constants, macros, or lifetimes.

#### **1. Keywords Currently in Use**
* **Control Flow & Branching**: `if`, `else`, `match`, `loop`, `while`, `for`, `in`, `break`, `continue`, `return`.
* **Types & Data Structures**: `struct`, `enum`, `type` (type alias/associated type), `trait`, `impl`, `Self` (type alias for implementing type), `self` (method subject).
* **Variable Binding & Ownership**: `let`, `mut`, `ref`, `move` (forces closure ownership capture), `const`, `static`.
* **Visibility & Modules**: `pub`, `mod`, `use`, `crate`, `super`.
* **Foreign Interfaces & Safety**: `unsafe`, `extern`, `as` (casting/renaming).
* **Generics & Lifetimes**: `where`, `for` (higher-ranked lifetimes).
* **Booleans**: `true`, `false`.

#### **2. Keywords Reserved for Future Use**
The following identifiers currently have no functionality but are reserved for potential future language features:
`abstract`, `alignof`, `become`, `box`, `do`, `final`, `macro`, `offsetof`, `override`, `priv`, `proc`, `pure`, `sizeof`, `typeof`, `unsized`, `virtual`, `yield`.

---

### **2. Appendix B – Operators and Symbols**

Appendix B summarizes Rust's syntax, categorizing operators by whether they are **overloadable** via standard library traits in `std::ops`.

#### **1. Overloadable Operators**

| Operator | Context / Example | Explanation | Overloadable Trait |
| :--- | :--- | :--- | :--- |
| **`!`** | `!expr` | Bitwise or logical complement | `Not` |
| **`!=`** | `var != expr` | Nonequality comparison | `PartialEq` |
| **`%` / `%=`** | `expr % expr` | Arithmetic remainder & assignment | `Rem` / `RemAssign` |
| **`&` / `&=`** | `expr & expr` | Bitwise AND & assignment | `BitAnd` / `BitAndAssign` |
| **`*` / `*=`** | `expr * expr` | Arithmetic multiplication & assignment | `Mul` / `MulAssign` |
| **`*`** | `*expr` | Dereference operator | `Deref` / `DerefMut` |
| **`+` / `+=`** | `expr + expr` | Arithmetic addition & assignment | `Add` / `AddAssign` |
| **`-` / `-=`** | `-expr`, `expr - expr` | Arithmetic negation & subtraction | `Neg`, `Sub` / `SubAssign` |
| **`/` / `/=`** | `expr / expr` | Arithmetic division & assignment | `Div` / `DivAssign` |
| **`<<` / `<<=`** | `expr << expr` | Left-shift & assignment | `Shl` / `ShlAssign` |
| **`>>` / `>>=`** | `expr >> expr` | Right-shift & assignment | `Shr` / `ShrAssign` |
| **`<` / `<=`** | `expr < expr` | Less than / Less than or equal to | `PartialOrd` |
| **`>` / `>=`** | `expr > expr` | Greater than / Greater than or equal to | `PartialOrd` |
| **`==`** | `expr == expr` | Equality comparison | `PartialEq` |
| **`^` / `^=`** | `expr ^ expr` | Bitwise XOR & assignment | `BitXor` / `BitXorAssign` |
| **`\|` / `\|= `** | `expr \| expr` | Bitwise OR & assignment | `BitOr` / `BitOrAssign` |
| **`[]`** | `expr[expr]` | Collection indexing | `Index` / `IndexMut` |

#### **2. Non-Overloadable & Specialized Symbols**
* **`?`**: Error propagation operator returning early on `Err` or `None`.
* **`..` / `...`**: Range literals (`0..5` right-exclusive, `1...5` inclusive pattern) and struct update syntax (`..default`).
* **`::`**: Path separator for modules (`std::cmp`), associated functions (`String::from`), or turbofish generic invocation (`parse::<i32>()`).
* **`@`**: Pattern binding operator (`id @ 3..7`).
* **`!`**: Never type for diverging functions (`fn foo() -> !`).
* **`#[meta]` / `#![meta]`**: Outer (annotates item following) and inner (annotates containing item) attributes.

---

### **3. Appendix C – Derivable Traits**

The **`#[derive(...)]`** attribute automatically generates default trait implementations for custom structs and enums.

```rust
#[derive(Debug, PartialEq, Eq, PartialOrd, Ord, Clone, Copy, Hash, Default)]
struct Point {
    x: i32,
    y: i32,
}
```

#### **1. `Debug` (Programmer Output)**
Enables debug formatting inside format strings using **`{:?}`** or **`{:#?}`**.
* **Requirement**: Needed by macros like `assert_eq!` to print comparison values upon test failure.

#### **2. `PartialEq` and `Eq` (Equality Comparisons)**
* **`PartialEq`**: Enables `==` and `!=` operators by checking equality across all struct fields or enum variants.
* **`Eq`**: A marker trait signaling **full equivalence relation** (where \\(a == a\\) is always true). Types containing floating-point numbers (`f32`/`f64`) cannot implement `Eq` because `NaN != NaN`.
* **Requirement**: `Eq` is required for keys stored inside a `HashMap<K, V>` or `HashSet<T>`.

#### **3. `PartialOrd` and `Ord` (Ordering Comparisons)**
* **`PartialOrd`**: Enables `<`, `>`, `<=`, and `>=` operators by comparing fields in the order they are defined inside the struct. Returns `Option<Ordering>` (returning `None` for unorderable values like `NaN`).
* **`Ord`**: Implements `cmp(&self, other) -> Ordering` for types where every pair of values produces a strict total ordering.
* **Requirement**: `Ord` is required for keys stored in sorted collections like `BTreeSet<T>`.

#### **4. `Clone` and `Copy` (Value Duplication)**
* **`Clone`**: Enables explicit deep copies via `.clone()`, executing code that duplicates heap allocations.
* **`Copy`**: Enables implicit stack-only bitwise duplication without invalidating the source variable. `Copy` cannot run custom destructor logic and cannot be derived if a type or any of its fields implements `Drop`.

#### **5. `Hash` and `Default`**
* **`Hash`**: Computes a fixed-size integer hash from a value using a hasher. Required for keys in a `HashMap`.
* **`Default`**: Provides a default value via `Default::default()`. Commonly combined with struct update syntax (`Point { x: 5, ..Default::default() }`).

---

### **4. Appendix D – Macros & Metaprogramming**

Macros are a way of writing code that generates other code at compile time (**metaprogramming**).

#### **1. Macros vs. Functions**
* **Variadic Parameters**: Macros can take a variable number of parameters (e.g., `println!("hello")` vs `println!("hello {}", name)`), whereas functions require a fixed signature.
* **Compile-Time Expansion**: Macros expand *before* the compiler interprets code meaning, enabling macros to implement traits on types at compile time.
* **Scoping**: Macro definitions use **`#[macro_use]`** / **`#[macro_export]`** annotations instead of standard module namespace paths.

---

#### **2. Declarative Macros (`macro_rules!`)**
Declarative macros match input Rust code against pattern arms and emit replacement code during compilation.

##### **Code Example: Simplified `vec!` Macro Definition**
```rust
#[macro_export]
macro_rules! vec {
    ( $( $x:expr ),* ) => {
        {
            let mut temp_vec = Vec::new();
            $(
                temp_vec.push($x);
            )*
            temp_vec
        }
    };
}
```
*(Source: Listing D-1)*

* **`#[macro_export]`**: Makes the macro available whenever the enclosing crate is imported.
* **`$x:expr`**: Matches any Rust expression and assigns it to variable `$x`.
* **`$( ... ),*`**: Matches zero or more repetitions of the inner pattern separated by commas.

---

#### **3. Procedural Macros for Custom `derive`**
Procedural macros operate like functions: they accept a stream of Rust source code tokens as input (`TokenStream`), transform the Abstract Syntax Tree (AST), and produce a new `TokenStream` as output.

Procedural macros must be defined inside their own dedicated library crate configured with `proc-macro = true` in `Cargo.toml`.

##### **Code Example: Custom `derive` Procedural Macro (`hello_macro_derive`)**
```rust
// hello_macro_derive/src/lib.rs
extern crate proc_macro;
extern crate syn;
#[macro_use]
extern crate quote;

use proc_macro::TokenStream;

#[proc_macro_derive(HelloMacro)]
pub fn hello_macro_derive(input: TokenStream) -> TokenStream {
    // Construct a string representation of the Rust code
    let s = input.to_string();

    // Parse string representation into a DeriveInput syntax tree
    let ast = syn::parse_derive_input(&s).unwrap();

    // Build the trait implementation code
    let gen = impl_hello_macro(&ast);

    // Convert generated tokens back into a TokenStream
    gen.parse().unwrap()
}

fn impl_hello_macro(ast: &syn::DeriveInput) -> quote::Tokens {
    let name = &ast.ident;
    quote! {
        impl HelloMacro for #name {
            fn hello_macro() {
                println!("Hello, Macro! My name is {}", stringify!(#name));
            }
        }
    }
}
```
*(Source: Listing D-3 & Procedural Macro Implementation)*

* **`syn` Crate**: Parses raw Rust source code strings into a structured `DeriveInput` AST.
* **`quote` Crate**: Converts Rust AST structures back into executable code tokens.
* **`#name` Templating**: Injects captured AST identifiers into generated output blocks.

##### **Consumer Usage**
```rust
use hello_macro::HelloMacro;
use hello_macro_derive::HelloMacro;

#[derive(HelloMacro)]
struct Pancakes;

fn main() {
    Pancakes::hello_macro(); // Prints: Hello, Macro! My name is Pancakes!
}
```
*(Source: Listing D-2)*

---
🦀 

## References
