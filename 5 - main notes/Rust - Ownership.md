---
created: 2026-10-14 14:20
updated: 2026-10-14 14:20
type: note
status: seed
tags:
  - rust
aliases: []
---

## Summary


## Notes
Chapter 4 of *The Rust Programming Language*, titled **Understanding Ownership**, introduces Rust’s most defining feature: its compile-time memory management model. Ownership allows Rust to guarantee complete memory safety without relying on a garbage collector.

---

### **1. What is Ownership & The Ownership Rules**

In traditional languages, memory is managed either manually (`malloc`/`free`) or automatically via dynamic garbage collection. Rust uses a third approach: memory is managed through an **ownership system** governed by strict rules that the compiler checks at compile time with zero runtime overhead.

#### **The Three Rules of Ownership:**
1. **Each value in Rust has a variable called its owner**.
2. **There can be only one owner at a time**.
3. **When the owner goes out of scope, the value will be dropped**.

#### **Variable Scope and RAII**
A variable is valid from the moment it is declared until its scope ends. When a complex data type allocated on the heap (such as `String`) goes out of scope, Rust automatically calls a special function named **`drop`** at the closing brace (`}`) to return that memory to the operating system. This pattern is known as **Resource Acquisition Is Initialization (RAII)**.

---

### **2. Memory Allocation: Move, Clone, and Copy**

#### **Stack vs. Heap**
* **Stack**: Stores values with a known, fixed size at compile time (e.g., integers, booleans, floating points) in a fast, last-in-first-out structure.
* **Heap**: Allocates memory at runtime for dynamic data whose size is unknown at compile time. The stack stores metadata (a pointer to the heap, length, and capacity).

```
Stack (s1)                  Heap
+-----------+-------+       +---+-----+
| name      | value |       |idx|value|
+-----------+-------+       +---+-----+
| ptr       | ------+------>| 0 | 'h' |
| len       |   5   |       | 1 | 'e' |
| capacity  |   5   |       | 2 | 'l' |
+-----------+-------+       | 3 | 'l' |
                            | 4 | 'o' |
                            +---+-----+
```
*(Source: Figure 4-1)*

#### **A. Move Semantics (Heap Data)**
Assigning one heap variable to another copies the pointer, length, and capacity on the stack, but **does not copy the heap data**. To prevent a **double free error** (where both variables attempt to free the same heap memory upon going out of scope), Rust invalidates the original variable. This operation is called a **move** rather than a shallow copy.

##### **Code Example: Move Operation**
```rust
let s1 = String::from("hello");
let s2 = s1; // s1 is MOVED into s2 and invalidated!

// println!("{}, world!", s1); // Compile error: use of moved value `s1`
```
*(Source: Memory Allocation - Move)*

#### **B. Clone (Deep Copy)**
To deeply copy heap data rather than transferring ownership, call the **`.clone()`** method.

```rust
let s1 = String::from("hello");
let s2 = s1.clone(); // Deeply copies heap data

println!("s1 = {}, s2 = {}", s1, s2); // Both variables remain valid
```
*(Source: Memory Allocation - Clone)*

#### **C. The `Copy` Trait (Stack Data)**
Types stored entirely on the stack implement the **`Copy` trait**. Reassigning stack-only values creates an independent copy without invalidating the original variable.

Types that implement `Copy` include:
* All integer types (e.g., `i32`, `u32`)
* Booleans (`bool`) and Characters (`char`)
* Floating-point numbers (`f64`, `f32`)
* Tuples containing only `Copy` types (e.g., `(i32, i32)`)

#### **Ownership and Functions**
Passing a variable to a function uses the same move or copy semantics as assignment. Returning values from functions transfers ownership back to the caller.

##### **Code Example: Functions and Ownership**
```rust
fn main() {
    let s = String::from("hello"); // s comes into scope

    takes_ownership(s); // s's value moves into the function...
                        // ... and is no longer valid here!

    let x = 5;          // x comes into scope
    makes_copy(x);      // x is Copy, so it remains valid here
}

fn takes_ownership(some_string: String) { // some_string comes into scope
    println!("{}", some_string);
} // some_string goes out of scope and drop is called; memory is freed

fn makes_copy(some_integer: i32) { // some_integer comes into scope
    println!("{}", some_integer);
} // some_integer goes out of scope; nothing special happens
```
*(Source: Listing 4-3)*

---

### **3. References and Borrowing**

Transferring ownership with every function call can be cumbersome. Rust provides **references** (`&`) to grant access to data without taking ownership—a concept called **borrowing**.

#### **A. Immutable References (`&T`)**
By default, references are **immutable** (`&s`). They allow code to read data without modifying it or claiming ownership.

```rust
fn main() {
    let s1 = String::from("hello");
    let len = calculate_length(&s1); // Pass immutable reference
    println!("The length of '{}' is {}.", s1, len); // s1 remains valid
}

fn calculate_length(s: &String) -> usize { // s is a reference to a String
    s.len()
} // s goes out of scope, but does not own the String, so nothing is dropped
```
*(Source: References and Borrowing)*

#### **B. Mutable References (`&mut T`)**
To modify borrowed data, pass a **mutable reference** (`&mut s`) from a variable declared with `let mut`.

```rust
fn main() {
    let mut s = String::from("hello");
    change(&mut s);
}

fn change(some_string: &mut String) {
    some_string.push_str(", world");
}
```
*(Source: Mutable References)*

#### **C. The Rules of References & Data Race Prevention**
To eliminate data races and prevent memory corruption, the compiler enforces two fundamental reference rules:

1. **At any given time, you can have either one mutable reference OR any number of immutable references in a given scope**.
2. **References must always be valid** (preventing dangling pointers).

##### **Preventing Data Races:**
A data race occurs when two or more pointers access the same memory location simultaneously, at least one is writing, and there is no synchronization. Rust prevents data races at compile time by disallowing multiple active mutable borrows, or mixing mutable and immutable borrows in the same scope.

##### **Preventing Dangling References:**
The compiler ensures that referenced data will not go out of scope before the reference pointing to it does.

```rust
// THIS CODE WILL NOT COMPILE
fn dangle() -> &String {
    let s = String::from("hello");
    &s // Error: returns a reference to data that is dropped at function exit!
}
```
*(Source: Dangling References)*

---

### **4. The Slice Type (`&str` and `&[T]`)**

Slices are references to a contiguous sequence of elements within a collection without taking ownership of the collection.

#### **A. String Slices (`&str`)**
A string slice stores a starting pointer and a length.

```rust
let s = String::from("hello world");

let hello: &str = &s[0..5];  // Slice from index 0 up to 5
let world: &str = &s[6..11]; // Slice from index 6 up to 11
```
*(Source: String Slices)*

##### **Range Syntax Shortcuts:**
* `&s[0..2]` can be written as `&s[..2]`.
* `&s[3..len]` can be written as `&s[3..]`.
* `&s[0..len]` can be written as `&s[..]`.

#### **B. String Literals Are Slices**
String literals (e.g., `let s = "Hello, world!";`) are stored directly inside the compiled binary text. The type of `s` is **`&str`**, an immutable slice pointing to that binary memory address.

#### **C. Idiomatic Function Parameters (`&str`)**
Instead of declaring functions with `&String` parameters, idiomatic Rust accepts `&str`. This allows the function to accept both `&String` (via deref coercion/slicing) and string literals directly.

```rust
fn first_word(s: &str) -> &str {
    let bytes = [suspicious link removed]_bytes();

    for (i, &item) in bytes.iter().enumerate() {
        if item == b' ' {
            return &s[0..i];
        }
    }

    &s[..]
}

fn main() {
    let my_string = String::from("hello world");

    // Works on slices of `String`s
    let word = first_word(&my_string[..]);

    let my_string_literal = "hello world";

    // Works on string literals directly
    let word = first_word(my_string_literal);
}
```
*(Source: Listing 4-9 & String Slices as Parameters)*

#### **D. General Array Slices (`&[T]`)**
Slice syntax applies to all contiguous collections, such as arrays:

```rust
let a =;
let slice: &[i32] = &a[1..3]; // References elements at index 1 and 2
```
*(Source: Other Slices)*

---
🦀

## References
