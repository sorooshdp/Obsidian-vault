---
created: 2026-10-17 17:53
updated: 2026-10-17 17:53
type: note
status: seed
tags:
  - rust
aliases: []
---

## Summary


## Notes
In **Chapter 14: More About Cargo and Crates.io**, _The Rust Programming Language_ details advanced features of Rust's package manager, including customizing build optimizations, generating documentation, publishing open-source libraries, organizing multi-crate projects using workspaces, and extending the Cargo command-line interface.

---

### **1. Customizing Builds with Release Profiles**

Cargo provides predefined **release profiles** that allow developers to control build configurations independently.

- **`dev` Profile**: Used when running **`cargo build`**. It defaults to **`opt-level = 0`**, skipping optimizations to maximize compilation speed during development iteration.
- **`release` Profile**: Used when running **`cargo build --release`**. It defaults to **`opt-level = 3`**, applying aggressive optimizations for production execution at the cost of longer compile times.

#### **Customizing Profiles in `Cargo.toml`**

You can override profile defaults by adding **`[profile.*]`** sections to your project's `Cargo.toml` file:

```
[profile.dev]
opt-level = 1

[profile.release]
opt-level = 3
```

_(Source: Profile Customization)_

---

### **2. Publishing a Library Crate to Crates.io**

#### **A. Documentation Comments (`///` and `//!`)**

Rust supports dedicated **documentation comments** that format text using Markdown and render HTML documentation via the **`rustdoc`** tool.

1. **Outer Doc Comments (`///`)**: Document the item that immediately follows them (e.g., functions, structs, enums).
2. **Inner Doc Comments (`//!`)**: Document the item that contains them, typically placed at the root of a crate (`src/lib.rs`) or module file to describe the container as a whole.

##### **Code Example: Function and Crate Documentation (`src/lib.rs`)**

```
//! # My Crate
//!
//! `my_crate` is a collection of utilities to make performing certain
//! calculations more convenient.

/// Adds one to the number given.
///
/// # Examples
///
/// ```
/// let five = 5;
/// assert_eq!(6, my_crate::add_one(5));
/// ```
pub fn add_one(x: i32) -> i32 {
    x + 1
}
```

_(Source: Listings 14-1 & 14-2)_

- **Common Documentation Sections**: Idiomatic docs include sections titled **`# Examples`**, **`Panics`** (scenarios where the item panics), **`Errors`** (conditions returning `Result::Err`), and **`Safety`** (invariants for `unsafe` functions).
- **Documentation Tests**: Code examples inside `///` comments are automatically compiled and run as tests during **`cargo test`**, ensuring that documentation examples never fall out of sync with code updates.
- **Generating HTML Docs**: Running **`cargo doc --open`** compiles the HTML documentation into `target/doc` and opens it in a browser.

---

#### **B. Re-Exporting a Clean Public API with `pub use`**

Deeply nested module structures (e.g., `art::kinds::PrimaryColor` or `art::utils::mix`) can be inconvenient for external consumers. Using **`pub use`** re-exports nested items at the crate root, exposing a simplified, flat public API while maintaining modular internal code organization.

##### **Code Example: Re-Exporting Types (`src/lib.rs`)**

```
//! # Art
//! A library for modeling artistic concepts.

pub use kinds::PrimaryColor;
pub use kinds::SecondaryColor;
pub use utils::mix;

pub mod kinds {
    pub enum PrimaryColor { Red, Yellow, Blue }
    pub enum SecondaryColor { Orange, Green, Purple }
}

pub mod utils {
    use kinds::*;
    pub fn mix(c1: PrimaryColor, c2: PrimaryColor) -> SecondaryColor {
        // ...
    }
}
```

_(Source: Listing 14-5)_

External users can then import types cleanly with `use art::PrimaryColor;` directly.

---

#### **C. Account Setup and Package Metadata**

1. **Authentication**: Log into (https://crates.io/) via GitHub to retrieve an API token, then run **`cargo login <token>`** to save credentials locally in `~/.cargo/credentials`.
2. **Package Metadata (`Cargo.toml`)**: Crates require a unique name on (https://crates.io/), along with `description` and `license` specifications using Linux Foundation SPDX identifiers (or `license-file`).

```
[package]
name = "guessing_game"
version = "0.1.0"
authors = ["Your Name <you@example.com>"]
description = "A fun game where you guess what number the computer has chosen."
license = "MIT OR Apache-2.0"

[dependencies]
```

_(Source: Package Metadata in `Cargo.toml`)_

---

#### **D. Publishing and Version Management**

- **Publishing (`cargo publish`)**: Uploads the package version to (https://crates.io/). **Publishing is permanent**—versions cannot be overwritten or deleted, ensuring a reliable build archive for dependent projects.
- **Updating Versions**: Update the `version` field according to Semantic Versioning (SemVer) rules and run `cargo publish` again.
- **Yanking Versions (`cargo yank`)**: If a version has a critical bug, run **`cargo yank --vers 1.0.1`**. Yanking prevents _new_ projects from taking that version as a dependency while allowing existing projects with `Cargo.lock` to continue building safely. Pass `--undo` to reverse a yank. _(Note: Yanking does not delete code or erase exposed secrets)_.

---

### **3. Cargo Workspaces**

A **Cargo workspace** is a group of related packages that share the same **`Cargo.lock`** file and top-level **`target/`** output directory.

#### **A. Workspace Layout & Root `Cargo.toml`**

A workspace directory contains a root `Cargo.toml` file defining the `[workspace]` members, but no `[package]` section.

```
add/
├── Cargo.lock          # Shared dependency lock file
├── Cargo.toml          # Workspace root configuration
├── add-one/            # Member crate (library)
│   ├── Cargo.toml
│   └── src/lib.rs
├── adder/              # Member crate (binary)
│   ├── Cargo.toml
│   └── src/main.rs
└── target/             # Shared build artifact directory
```

_(Source: Workspace Directory Layout)_

##### **Root `Cargo.toml`**

```
[workspace]
members = [
    "adder",
    "add-one",
]
```

_(Source: Workspace Root `Cargo.toml`)_

#### **B. Path Dependencies Between Workspace Crates**

Cargo does not automatically assume member crates depend on each other; dependencies must be declared explicitly using path references.

##### **`adder/Cargo.toml`**

```
[dependencies]
add-one = { path = "../add-one" }
```

_(Source: Workspace Path Dependency)_

#### **C. Building, Running, and Testing Workspaces**

- **Shared `Cargo.lock`**: Guarantees that all member crates use identical versions of shared external dependencies (like `rand`), ensuring compatibility across the workspace.
- **Running Packages**: Execute a specific binary package using the `-p` flag:
    
    ```
    $ cargo run -p adder
    ```
    
    _(Source: Workspace Command Execution)_
- **Testing Workspaces**: **`cargo test`** runs tests across all crates in the workspace simultaneously. To test a specific member crate, run **`cargo test -p add-one`**.
- **Publishing**: Workspaces cannot be published in a single command; each crate must be published individually via `cargo publish` inside its crate directory.

---

### **4. Installing Binaries and Extending Cargo**

#### **Installing Binaries (`cargo install`)**

The **`cargo install`** command downloads and installs binary executable crates directly from (https://crates.io/) into `$HOME/.cargo/bin` for local CLI tool usage (e.g., installing `ripgrep`):

```
$ cargo install ripgrep
```

_(Source: Installing Binaries)_

#### **Extending Cargo with Custom Commands**

If an executable binary in your system `$PATH` is named `cargo-something`, you can invoke it as a native Cargo subcommand by executing **`cargo something`**.

---
🦀

## References
