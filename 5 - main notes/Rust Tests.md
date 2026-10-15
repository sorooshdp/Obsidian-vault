---
created: 2026-10-15 16:34
updated: 2026-10-15 16:34
type: note
status: seed
tags:
  - rust
aliases: []
---

## Summary


## Notes

In **Chapter 11: Writing Automated Tests**, *The Rust Programming Language* details the language's built-in testing harness, attributes, assertion macros, execution flags, and organizational conventions for unit and integration testing.

---

### **1. How to Write Tests**

Tests are Rust functions that verify that non-test code functions as intended. A test function typically performs three actions: setting up data/state, running the code under test, and asserting that the results match expectations.

#### **A. Anatomy of a Test Function**
A standard function becomes a test function when annotated with the **`#[test]`** attribute. Attributes are metadata about pieces of Rust code. When running **`cargo test`**, Rust builds a test runner binary that executes all `#[test]`-annotated functions and reports whether each passes or fails. A test fails whenever the test thread panics.

##### **Code Example: Standard Test Module (`src/lib.rs`)**
```rust
#[cfg(test)]
mod tests {
    #[test]
    fn exploration() {
        assert_eq!(2 + 2, 4);
    }

    #[test]
    fn another() {
        panic!("Make this test fail");
    }
}
```
*(Source: Listing 11-1 & Listing 11-3)*

---

#### **B. Assertion Macros (`assert!`, `assert_eq!`, `assert_ne!`)**

1. **`assert!(boolean_expr)`**: Ensures a condition evaluates to `true`. If `false`, `assert!` calls `panic!`.
2. **`assert_eq!(left, right)` & `assert_ne!(left, right)`**: Tests two values for equality or inequality.
   * **Trait Requirement**: Under the hood, these macros use `==` and `!=` and print values using debug formatting. Therefore, types being compared **must implement the `PartialEq` and `Debug` traits**. On custom structs and enums, this is typically done by adding `#[derive(PartialEq, Debug)]`.
   * **Parameter Naming**: Unlike frameworks that enforce `expected` vs. `actual` ordering, Rust names parameters `left` and `right`, allowing arguments to be passed in either order.

##### **Code Example: Testing Equality and Struct Methods**
```rust
#[derive(Debug, PartialEq)]
pub struct Rectangle {
    length: u32,
    width: u32,
}

impl Rectangle {
    pub fn can_hold(&self, other: &Rectangle) -> bool {
        self.length > other.length && self.width > other.width
    }
}

pub fn add_two(a: i32) -> i32 {
    a + 2
}

#[cfg(test)]
mod tests {
    use super::*; // Brings outer module definitions into inner test scope

    #[test]
    fn larger_can_hold_smaller() {
        let larger = Rectangle { length: 8, width: 7 };
        let smaller = Rectangle { length: 5, width: 1 };
        assert!(larger.can_hold(&smaller));
    }

    #[test]
    fn it_adds_two() {
        assert_eq!(4, add_two(2));
    }
}
```
*(Source: Listings 11-5, 11-6, 11-7)*

---

#### **C. Adding Custom Failure Messages**
Any arguments passed after the required parameters of `assert!`, `assert_eq!`, or `assert_ne!` are passed to the **`format!`** macro. This allows you to construct custom error messages with `{}` placeholders to document failure context.

```rust
#[test]
fn greeting_contains_name() {
    let result = greeting("Carol");
    assert!(
        result.contains("Carol"),
        "Greeting did not contain name, value was `{}`", result
    );
}
```
*(Source: Custom Failure Messages)*

---

#### **D. Checking for Panics (`#[should_panic]`)**
To test that error conditions properly trigger panics, annotate the test function with **`#[should_panic]`** alongside `#[test]`.
* **`expected` Parameter**: To ensure the function panics for the *correct* reason (rather than an unrelated bug), add `expected = "substring"` to verify that the panic message contains specific text.

```rust
pub struct Guess {
    value: u32,
}

impl Guess {
    pub fn new(value: u32) -> Guess {
        if value < 1 {
            panic!("Guess value must be greater than or equal to 1, got {}.", value);
        } else if value > 100 {
            panic!("Guess value must be less than or equal to 100, got {}.", value);
        }

        Guess { value }
    }
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    #[should_panic(expected = "Guess value must be less than or equal to 100")]
    fn greater_than_100() {
        Guess::new(200);
    }
}
```
*(Source: Listing 11-9)*

---

#### **E. Using `Result<T, E>` in Tests**
Tests can return a `Result<T, E>` type instead of panicking. A test passes if it returns `Ok(())` and fails if it returns `Err(E)`. This allows using the `?` operator inside tests to handle functions that return errors.

```rust
#[test]
fn it_works() -> Result<(), String> {
    if 2 + 2 == 4 {
        Ok(())
    } else {
        Err(String::from("two plus two does not equal four"))
    }
}
```

---

### **2. Controlling How Tests Are Run**

Command-line arguments can customize `cargo test` behavior. Flags passed directly to `cargo test` go before `--`, while flags passed to the compiled test binary go after `--`.

#### **A. Parallel Execution vs. Single-Threaded**
By default, tests execute in parallel using multiple threads. If tests share state (such as writing to a shared file on disk or modifying environment variables), parallel execution can cause intermittent failures due to race conditions.
* **Control Thread Count**: Pass `-- --test-threads=1` to enforce sequential execution:
  ```bash
  $ cargo test -- --test-threads=1
  ```

#### **B. Controlling Standard Output Capture**
By default, if a test passes, Rust's test harness captures all output printed to standard output (e.g., `println!`) and suppresses it. Output is displayed only if a test fails.
* **Disable Capture**: Pass `-- --nocapture` to display stdout for passing tests in real-time:
  ```bash
  $ cargo test -- --nocapture
  ```

#### **C. Filtering Tests and Ignoring Slow Tests**
* **Run a Single Test**: Pass the exact function name (`$ cargo test one_hundred`).
* **Filter by Substring**: Pass part of a name to execute all matching tests (`$ cargo test add` runs `add_two_and_two` and `add_three_and_two`).
* **Ignore Expensive Tests (`#[ignore]`)**: Annotate slow tests with `#[ignore]`.
  ```rust
  #[test]
  #[ignore]
  fn expensive_test() {
      // Code that takes an hour to run
  }
  ```
  *(Source: Ignoring Tests)*
* **Run Only Ignored Tests**: Run `$ cargo test -- --ignored`.

---

### **3. Test Organization: Unit Tests vs. Integration Tests**

Rust categorizes tests into two distinct types: **unit tests** and **integration tests**.

```
My Crate Layout
├── Cargo.toml
├── src/
│   └── lib.rs              <-- Unit tests reside inside source files
└── tests/
    ├── integration_test.rs <-- Integration tests (treated as external crates)
    └── common/
        └── mod.rs          <-- Helper modules (ignored by test runner)
```
*(Source: Test Organization)*

---

#### **A. Unit Tests and `#[cfg(test)]`**
Unit tests test isolated modules in isolation and reside directly inside `src/` files alongside the code they test.

##### **The `#[cfg(test)]` Attribute**
The `#[cfg(test)]` attribute on a test module directs Rust to compile and include the test code **only when running `cargo test`**, excluding it when building the release binary (`cargo build`) to save compile time and binary size.

##### **Testing Private Functions**
Rust's privacy rules allow unit tests to test **private functions**. Because the inner `tests` module is a child of the outer module, bringing items into scope with `use super::*;` grants access to private functions like `internal_adder`.

```rust
pub fn add_two(a: i32) -> i32 {
    internal_adder(a, 2)
}

fn internal_adder(a: i32, b: i32) -> i32 { // Private function
    a + b
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn internal() {
        assert_eq!(4, internal_adder(2, 2)); // Testing private function
    }
}
```
*(Source: Listing 11-12)*

---

#### **B. Integration Tests**

Integration tests are **entirely external to your library**. They reside in a root-level **`tests/`** directory next to `src/`.

* **External Scope**: Cargo compiles every file in `tests/` as its own separate crate. Integration tests can only call functions exposed in your crate's **public API**.
* **Importing the Crate**: Each file must bring the library into scope using `extern crate <crate_name>;` (or `use <crate_name>;`).
* **No `#[cfg(test)]` Needed**: Files inside `tests/` are compiled only during `cargo test`.

##### **Code Example: Integration Test (`tests/integration_test.rs`)**
```rust
extern crate adder;

#[test]
fn it_adds_two() {
    assert_eq!(4, adder::add_two(2));
}
```
*(Source: Listing 11-13)*

##### **Submodules in Integration Tests (`tests/common/mod.rs`)**
Because every file in `tests/` is compiled as a separate test crate, creating a helper file like `tests/common.rs` causes Cargo to treat it as a test file and print a `running 0 tests` section.

To share helper code across integration test files without generating dummy test output, use the alternate module naming convention **`tests/common/mod.rs`**. Files in subdirectories of `tests/` are not compiled as individual test crates.

```rust
// tests/integration_test.rs
extern crate adder;
mod common;

#[test]
fn it_adds_two() {
    common::setup();
    assert_eq!(4, adder::add_two(2));
}
```
*(Source: Using submodules in integration tests)*

##### **Integration Tests for Binary Crates**
If a project contains only `src/main.rs` without a `src/lib.rs`, you cannot create integration tests in `tests/` because binary crates cannot be imported with `extern crate`. Idiomatic Rust binary projects maintain a minimal `src/main.rs` that delegates logic to `src/lib.rs`, allowing `tests/` to thoroughly exercise the public library API.

---
🦀 
## References
