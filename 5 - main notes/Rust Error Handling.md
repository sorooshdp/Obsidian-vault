---
created: 2026-10-15 14:09
updated: 2026-10-15 14:09
type: note
status: seed
tags:
  - rust
aliases: []
---

## Summary


## Notes
In **Chapter 9: Error Handling**, *The Rust Programming Language* details Rust's approach to software reliability. Instead of relying on traditional exceptions, Rust categorizes errors into **recoverable** and **unrecoverable** errors, forcing developers to acknowledge and handle failure scenarios at compile time.

---

### **1. The Philosophy of Error Handling in Rust**

#### **A. Recoverable vs. Unrecoverable Errors**
* **Recoverable Errors**: Operational failures that a program can reasonably report to the user and retry (e.g., a *file not found* error). These are represented by the generic **`Result<T, E>`** enum.
* **Unrecoverable Errors**: Symptoms of serious bugs where the program reaches an invalid state and cannot safely continue execution (e.g., attempting to access an index beyond the end of an array). These are handled by the **`panic!`** macro, which halts execution immediately.

#### **B. Why Rust Does Not Have Exceptions**
Most mainstream languages handle both recoverable and unrecoverable errors using exceptions. Rust deliberately excludes exceptions to prevent pervasive null/error-checking ambiguity and unhandled runtime crashes, choosing instead to encode success and failure directly into the type system.

---

### **2. Unrecoverable Errors with `panic!`**

When `panic!` executes, the program prints a failure message, unwinds the stack, cleans up memory resources, and quits.

#### **A. Unwinding the Stack vs. Aborting**
* **Unwinding (Default)**: Rust walks back up the call stack and cleans up data from each function it encounters. This adds extra work to the compiled binary.
* **Aborting**: Ends the program immediately without cleaning up stack data, leaving cleanup to the operating system. To produce the smallest possible binary, you can opt into aborting on panic by configuring `Cargo.toml`:

```toml
[profile.release]
panic = 'abort'
```
*(Source: Profile Panic Abort Configuration)*

#### **B. Preventing Buffer Overreads (Memory Safety Guarantee)**
In languages like C, accessing an out-of-bounds array element reads whatever raw memory sits at that address, creating a **buffer overread** vulnerability. In Rust, indexing past a collection's bounds causes an immediate `panic!`, protecting memory safety.

##### **Code Example: Vector Out-of-Bounds Access (`src/main.rs`)**
```rust
fn main() {
    let v = vec!;

    v[99]; // Causes panic!: index out of bounds: the len is 3 but the index is 99
}
```
*(Source: Listing 9-1)*

#### **C. Using Stack Backtraces (`RUST_BACKTRACE`)**
To trace the exact sequence of function calls that led to a panic, set the `RUST_BACKTRACE` environment variable in your terminal:

```bash
$ RUST_BACKTRACE=1 cargo run
```
*(Source: Stack Backtrace Command)*

Reading a backtrace involves starting from the top and scrolling down until you locate the first line referencing source code you wrote.

---

### **3. Recoverable Errors with `Result<T, E>`**

The standard library defines `Result` as an enum with two generic variants:

```rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```
*(Source: Result Definition)*

* **`T`**: Represents the type of the value returned in a success case within `Ok`.
* **`E`**: Represents the type of the error returned in a failure case within `Err`.

#### **A. Matching on `Result` and Handling Error Kinds**
Because `Result` is included in the prelude, its variants `Ok` and `Err` can be matched directly.

##### **Code Example: Handling Different Errors with Match Guards (`src/main.rs`)**
```rust
use std::fs::File;
use std::io::ErrorKind;

fn main() {
    let f = File::open("hello.txt");

    let f = match f {
        Ok(file) => file,
        // Match guard using `ref` to borrow `error` without moving it
        Err(ref error) if error.kind() == ErrorKind::NotFound => {
            match File::create("hello.txt") {
                Ok(fc) => fc,
                Err(e) => panic!("Tried to create file but there was a problem: {:?}", e),
            }
        },
        Err(error) => panic!("There was a problem opening the file: {:?}", error),
    };
}
```
*(Source: Listing 9-5)*

* **Match Guards & `ref`**: The `if error.kind() == ErrorKind::NotFound` condition is a **match guard** that refines the arm. The **`ref`** keyword binds `error` by reference so it is borrowed by the guard condition rather than moved into it.

---

#### **B. Shortcuts for Panic on Error: `unwrap` and `expect`**

Instead of verbose `match` blocks, `Result` provides helper shortcuts:

1. **`unwrap()`**: Returns the value inside `Ok(T)`. If the result is `Err(E)`, it calls `panic!` with a default message.
2. **`expect("msg")`**: Works like `unwrap()`, but allows you to pass a custom error message to `panic!`, making debugging significantly easier.

```rust
use std::fs::File;

fn main() {
    // Calling unwrap
    let f = File::open("hello.txt").unwrap();

    // Calling expect with a descriptive error message
    let f = File::open("hello.txt").expect("Failed to open hello.txt");
}
```
*(Source: Shortcuts for Panic)*

---

#### **C. Propagating Errors and the `?` Operator**

When a function's inner implementation encounters a failure, it can pass the error to the calling function to let the caller decide how to handle it—a process called **propagating the error**.

##### **Verbose Propagation with `match` (`src/main.rs`)**
```rust
use std::io;
use std::io::Read;
use std::fs::File;

fn read_username_from_file() -> Result<String, io::Error> {
    let f = File::open("hello.txt");

    let mut f = match f {
        Ok(file) => file,
        Err(e) => return Err(e), // Early return of the error
    };

    let mut s = String::new();

    match f.read_to_string(&mut s) {
        Ok(_) => Ok(s),
        Err(e) => Err(e),
    }
}
```
*(Source: Listing 9-6)*

##### **Idiomatic Propagation with the `?` Operator & Call Chaining (`src/main.rs`)**
The **`?` operator** eliminates boilerplate. If a `Result` is `Ok`, the inner value is returned from the expression. If it is `Err`, the operator performs an early return from the whole function and propagates the `Err` value to the caller.

```rust
use std::io;
use std::io::Read;
use std::fs::File;

fn read_username_from_file() -> Result<String, io::Error> {
    let mut s = String::new();

    // Chaining calls using the ? operator
    File::open("hello.txt")?.read_to_string(&mut s)?;

    Ok(s)
}
```
*(Source: Listing 9-7 & Listing 9-8)*

#### **D. Rust-Specific Feature: Error Type Conversion via `From` Trait**
When using the `?` operator, the error value automatically passes through the **`from` function** defined in the standard library's `std::convert::From` trait. If a function returns a custom error type `E`, any sub-error type that implements `From<SubError> for E` will be automatically converted to `E` by the `?` operator.

#### **E. Restrictions on the `?` Operator**
The `?` operator **can only be used in functions that return a compatible type** (such as `Result` or a type implementing `std::ops::Try`). Attempting to use `?` inside a standard `fn main()` that returns unit `()` produces a compile-time error.

---

### **4. Guidelines: To `panic!` or Not to `panic!`**

Returning `Result` is the recommended default because it gives the calling code options rather than forcing a process halt. However, panicking is preferred in specific contexts:

1. **Examples, Prototypes, and Tests**: Using `unwrap()` or `expect()` in code examples keeps them clean. In tests, panicking is the mechanism used to mark a test as failed.
2. **When You Have More Information Than the Compiler**: If you can prove logically that an operation will never fail, calling `.unwrap()` is acceptable.

```rust
use std::net::IpAddr;

// The compiler doesn't know "127.0.0.1" is valid, but we do
let home: IpAddr = "127.0.0.1".parse().unwrap();
```
*(Source: Having More Information Than Compiler)*

3. **Invariants, Contracts, and Bad States**: Code should `panic!` when an assumption, contract, or invariant has been violated by the caller (e.g., passing invalid or out-of-range inputs to library functions), as this indicates a caller-side bug that cannot be recovered from at runtime.

---

### **5. Creating Custom Types for Validation**

Instead of scattering repetitive error-checking `if` statements throughout code, idiomatic Rust uses the type system to create **custom validation types**.

##### **Code Example: Encapsulated `Guess` Type (`src/lib.rs`)**
```rust
pub struct Guess {
    value: u32, // Private field prevents external code from bypassing validation
}

impl Guess {
    pub fn new(value: u32) -> Guess {
        if value < 1 || value > 100 {
            panic!("Guess value must be between 1 and 100, got {}.", value);
        }

        Guess { value }
    }

    // Public getter method to access the private field
    pub fn value(&self) -> u32 {
        self.value
    }
}
```
*(Source: Listing 9-9)*

* **Encapsulation & Safety**: By keeping `value` private, external code **must** use `Guess::new()` to construct instances. Once a `Guess` instance is instantiated, any function receiving it can proceed with its logic confident that the value has already been validated between 1 and 100.

---
🦀 

## References
