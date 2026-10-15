---
created: 2026-10-13 20:10
updated: 2026-10-13 20:10
type: note
status:
tags:
  - rust
aliases: []
---

## Summary

Here is an in-depth guide through the **Introduction** and **Chapter 1: Getting Started** of *The Rust Programming Language*, covering all foundational concepts, workflow conventions, and essential code examples.

---

### **Part 1: Introduction – The Philosophy & Vision of Rust**

#### **1. What is Rust?**
Rust is an open-source systems programming language designed to help developers write faster, more reliable software. It combines low-level control over memory usage and data representation with high-level ergonomics and developer productivity. Rust aims to eliminate the traditional trade-offs between safety and speed by striving for **zero-cost abstractions**—higher-level language features that compile to lower-level code as fast as code written manually.

#### **2. The Rust Ecosystem & Tooling**
Rust equips developers with contemporary tooling brought directly to systems programming:
* **Cargo**: The built-in package manager and build system that simplifies managing dependencies, compiling code, running tests, and generating documentation.
* `**Rustfmt**`: An automated code formatter that ensures a consistent formatting style across the Rust ecosystem.
* **Rust Language Server (RLS)**: Powers Integrated Development Environment (IDE) integration for code completion and inline error warnings.

#### **3. Compiler-Led Development**
A core aspect of learning and programming in Rust is **compiler-led programming**. The Rust compiler provides detailed error messages and warnings designed to actively guide developers toward valid, safe code.

---

### **Part 2: Chapter 1 – Getting Started with Rust**

#### **1. Installation**
Rust is installed and updated using **`rustup`**, a command-line tool for managing Rust versions and associated development tools.

```bash
$ rustup update
```

#### **2. The Classic "Hello, World!" Program**
Rust source files always end with the **`.rs`** extension. When naming files containing multiple words, Rust style dictates using an underscore to separate words in **`snake_case`**.

##### **Source Code**
```rust
fn main() {
    println!("Hello, world!");
}
```

##### **Compiling and Running Directly with `rustc`**
Rust is an **ahead-of-time (AOT) compiled language**. You compile the source code into an executable binary using the `rustc` compiler, then execute the binary separately:

```bash
# Compile on Linux/macOS or Windows
$ rustc [suspicious link removed]

# Execute on Linux/macOS
$ ./main

# Execute on Windows CMD or PowerShell
> .\main.exe
```

#### **3. Anatomy of a Rust Program**
* **Function Declaration (`fn main()`)**: The `main` function is special—it is always the first code that runs in every executable Rust program. The `fn` keyword declares a function, followed by its name, parameter list `()`, and return signature.
* **Curly Brackets (`{}`)**: Wrap all function bodies. The opening curly bracket conventionally goes on the same line as the declaration with one space in between.
* **Indentation Style**: Standard Rust style indents code using **four spaces** rather than tabs.
* **Macros vs. Functions (`println!`)**: Calling `println!` invokes a **Rust macro** rather than a standard function. The exclamation mark (`!`) signifies a macro call.
* **Semicolons (`;`)**: End statements, signaling that an expression is complete and the next line is ready to begin.

---

### **Part 3: Managing Real-World Rust Projects with Cargo**

Cargo is Rust’s package manager and build system. It handles project creation, compiling code, managing external dependencies (referred to as **crates**), and structuring builds.

#### **1. Creating a Cargo Project**
To create a new executable application with Cargo, pass the project name with the `--bin` flag:

```bash
$ cargo new hello_cargo --bin
$ cd hello_cargo
```

This generates a standard directory layout:
```text
hello_cargo/
├── Cargo.toml
└── src/
    └── [suspicious link removed]
```
* **`src/` Directory**: Cargo expects all source files to live inside `src/`. The root project directory is reserved for configuration files, documentation, and READMEs.
* **Version Control**: `cargo new` initializes a Git repository by default along with a `.gitignore` file.

#### **2. Project Configuration (`Cargo.toml`)**
Cargo uses the **TOML** (Tom’s Obvious, Minimal Language) format for configuration files.

##### **Generated `Cargo.toml`**
```toml
[package]
name = "hello_cargo"
version = "0.1.0"
authors = ["Your Name <you@example.com>"]

[dependencies]
```
* **`[package]`**: Configures package metadata required for compilation (project name, version, author info).
* **`[dependencies]`**: Lists external crate dependencies required by the project.

#### **3. Dependency Locking (`Cargo.lock`)**
When you build a Cargo project for the first time, Cargo creates a **`Cargo.lock`** file at the top level. This file tracks the exact versions of dependencies used, guaranteeing reproducible builds across environments.

#### **4. Core Cargo Workflow Commands**

| Command | Description |
| :--- | :--- |
| **`cargo build`** | Compiles the project and creates an executable file in `target/debug/hello_cargo`. |
| **`cargo run`** | Compiles the code (if modified) and runs the resulting executable in one step. |
| **`cargo check`** | Quickly checks the code to ensure it compiles without incurring the time cost of generating an executable binary. |
| **`cargo build --release`** | Compiles the project with optimizations enabled in `target/release/` for production deployment. |

---

🦀 

## Notes


## References
