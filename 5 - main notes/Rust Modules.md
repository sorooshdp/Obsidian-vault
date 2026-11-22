---
created: 2026-10-15 13:30
updated: 2026-10-15 13:30
type: note
status: seed
tags:
  - rust
aliases: []
topic: "[[Rust]]"
---

## Summary


## Notes
In **Chapter 7: Using Modules to Reuse and Organize Code**, *The Rust Programming Language* details how Rust manages code structure, namespaces, file organization, and API privacy as applications grow beyond a single file.

---

### **1. Packages, Crates, and the Module System**

To keep large programs organized and maintainable, Rust provides a multi-tiered hierarchy for grouping functionality:

* **Packages**: A Cargo feature that lets you build, test, and share crates. A package contains a `Cargo.toml` file describing how to build one or more crates.
* **Crates**: A tree of modules that produces a library or an executable.
  * **Binary Crates**: Executable programs compiled with an entry point at `src/main.rs`.
  * **Library Crates**: Reusable code meant to be pulled into other projects as dependencies, with a root file at `src/lib.rs`. You create a library project with `cargo new <name> --lib`.
* **Modules (`mod`)**: Namespaces inside a crate that group related definitions (functions, structs, enums) and control their visibility.

---

### **2. Module Definitions and the Filesystem Hierarchy**

#### **A. Defining Inline Modules**
Every module definition starts with the **`mod`** keyword followed by the module name and a block inside curly brackets `{}`. Modules can be nested inside other modules to construct a logical hierarchy.

```rust
// src/lib.rs
mod network {
    fn connect() {
        // network implementation
    }

    mod client {
        fn connect() {
            // client implementation nested inside network
        }
    }
}
```
*(Source: Listing 7-2)*

Functions in different modules can share the same name (such as `connect`) without colliding because they reside in distinct namespaces (e.g., `network::connect()` vs. `network::client::connect()`).

---

#### **B. Moving Modules to Separate Files**
To prevent large files from becoming difficult to navigate, module contents can be extracted into separate files. In the root file (`src/lib.rs` or `src/main.rs`), replace the module body with a semicolon:

```rust
// src/lib.rs
mod client; // Tells Rust to look in src/client.rs for module contents
mod network; // Tells Rust to look in src/network.rs or src/network/mod.rs
```
*(Source: Listing 7-4)*

##### **Filesystem Rules for Submodules:**
1. **Modules without submodules**: If a module named `client` has no submodules, its declarations live in **`src/client.rs`**.
2. **Modules with submodules**: If a module named `network` has submodules (e.g., `server`), its declarations live in **`src/network/mod.rs`**, and its submodule declarations live inside **`src/network/server.rs`**.

```text
src/
├── lib.rs              # Crate root declaring `mod client;` and `mod network;`
├── client.rs           # Contents of `client` module
└── network/
    ├── mod.rs          # Contents of `network` module declaring `mod server;`
    └── server.rs       # Contents of `network::server` submodule
```
*(Source: Filesystem Layout)*

---

### **3. Controlling Visibility with `pub` and Privacy Rules**

#### **A. Private by Default**
In Rust, **all code (functions, structs, enums, modules, constants) is private by default**. Private items cannot be called or accessed from external code or parent namespaces.

The **`pub`** keyword makes an item public, explicitly exposing it for use outside its namespace and building a clean public API.

```rust
// src/lib.rs
pub mod client; // Public module accessible to external crates
pub mod network;

// src/client.rs
pub fn connect() {
    // Public function exposed inside public client module
}
```
*(Source: Listing 7-4 & `pub` declarations)*

---

#### **B. The Rules of Privacy**
Rust enforces two core visibility rules:
1. **If an item is public (`pub`), it can be accessed through any of its parent modules**.
2. **If an item is private, it can be accessed only by its immediate parent module and any of that parent's child modules**.

##### **Privacy Example Code Block**
```rust
mod outermost {
    pub fn middle_function() {}
    fn middle_secret_function() {} // Private to outermost

    mod inside {
        pub fn inner_function() {}
        fn secret_function() {}    // Private to inside
    }
}

fn try_me() {
    outermost::middle_function();            // Allowed: middle_function is pub
    // outermost::middle_secret_function();  // Error: private item in outermost
    // outermost::inside::inner_function();  // Error: `inside` module is private!
}
```
*(Source: Listing 7-6)*

---

### **4. Bringing Paths into Scope (`use`, Glob `*`, and `pub use`)**

#### **A. Shortening Paths with the `use` Keyword**
Calling deeply nested items (e.g., `a::series::of::nested_modules()`) becomes verbose. The **`use`** keyword brings a path into the current scope so items can be referenced with shorter names.

```rust
pub mod a {
    pub mod series {
        pub mod of {
            pub fn nested_modules() {}
        }
    }
}

use a::series::of; // Brings the `of` module into scope

fn main() {
    of::nested_modules(); // Called directly
}
```
*(Source: Bringing Names into Scope)*

* **Idiomatic Use Paths**:
  * **Functions**: Bring the **parent module** into scope (e.g., `use a::series::of;`) rather than the function directly to clarify that the function is not locally defined.
  * **Structs, Enums, and Other Items**: Bring the item **directly** into scope (e.g., `use std::collections::HashMap;`).

---

#### **B. Multiple Imports & The Glob Operator (`*`)**
* **Grouped Imports**: Bring multiple items from the same namespace into scope using curly brackets:
  ```rust
  use TrafficLight::{Red, Yellow};
  ```
  *(Source: Nested `use` lists)*
* **Glob Operator (`*`)**: Brings **all public items** of a namespace into scope at once:
  ```rust
  use TrafficLight::*;
  ```
  *(Source: Glob Operator)*

---

#### **C. Re-Exporting Clean Public APIs with `pub use`**
When internal module hierarchies are complex, users might find it tedious to import deeply nested paths. You can re-export items at a higher level using **`pub use`**, exposing a simplified public structure that differs from your internal code organization.

```rust
// src/lib.rs
pub use kinds::PrimaryColor;
pub use utils::mix;

pub mod kinds {
    pub enum PrimaryColor { Red, Yellow, Blue }
}

pub mod utils {
    use kinds::*;
    pub fn mix(c1: PrimaryColor, c2: PrimaryColor) { /* ... */ }
}
```
*(Source: Listing 14-5)*

---

### **5. Accessing Parent Modules with `super`**

When referencing items located in a parent module relative to the current module, use the **`super`** keyword. This behaves like starting a filesystem path with `..`.

Using `super` makes code cleaner and easier to update if you move a module hierarchy subtree elsewhere in your project.

```rust
pub mod client {
    pub fn connect() {}
}

#[cfg(test)]
mod tests {
    use super::client; // Accesses sibling `client` module via parent scope

    #[test]
    fn it_works() {
        client::connect();
    }
}
```
*(Source: Using `super` in tests)*

---
🦀 

## References
