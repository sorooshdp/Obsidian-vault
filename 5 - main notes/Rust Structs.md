---
created: 2026-10-14 15:47
updated: 2026-10-14 15:47
type: note
status: seed
tags:
  - rust
aliases: []
---

## Summary


## Notes

In **Chapter 5: Using Structs to Structure Related Data**, *The Rust Programming Language* introduces custom data structures that allow developers to group and label related values into meaningful domain types.

---

### **1. Defining and Instantiating Structs**

A **struct** (structure) is a custom data type that packages together related pieces of data. Unlike tuples, each piece of data in a struct is explicitly named so its meaning is clear, removing reliance on element ordering.

#### **Defining a Struct & Creating Instances**
Structs are declared using the **`struct`** keyword followed by field names and their types. Instances are created by supplying key-value pairs inside curly brackets.

```rust
// Defining the struct
struct User {
    username: String,
    email: String,
    sign_in_count: u64,
    active: bool,
}

fn main() {
    // Instantiating the struct
    let user1 = User {
        email: String::from("someone@example.com"),
        username: String::from("someusername123"),
        active: true,
        sign_in_count: 1,
    };
}
```
*(Source: Listing 5-1 & Listing 5-2)*

#### **Field Access & Mutability Rules**
* **Dot Notation**: Fields are accessed using dot notation.
* **Instance-Level Mutability**: To modify any field within a struct instance, the entire instance must be declared mutable using **`let mut`**. **Rust does not allow marking only specific fields as mutable**.

```rust
let mut user1 = User {
    email: String::from("someone@example.com"),
    username: String::from("someusername123"),
    active: true,
    sign_in_count: 1,
};

[suspicious link removed] = String::from("anotheremail@example.com");
```
*(Source: Listing 5-3)*

---

### **2. Ergonomic Struct Syntaxes**

#### **A. Field Init Shorthand**
When function parameter names match struct field names exactly, you can omit the repetitive `field_name: field_name` syntax.

```rust
fn build_user(email: String, username: String) -> User {
    User {
        email,    // Shorthand for email: email
        username, // Shorthand for username: username
        active: true,
        sign_in_count: 1,
    }
}
```
*(Source: Listing 5-5)*

#### **B. Struct Update Syntax (`..`)**
To create a new struct instance that reuses most fields from an existing instance, use **struct update syntax** with **`..`** to fill in unstated fields.

```rust
let user2 = User {
    email: String::from("another@example.com"),
    username: String::from("anotherusername567"),
    ..user1 // Copies/moves remaining fields from user1
};
```
*(Source: Listing 5-7)*

---

### **3. Tuple Structs and Unit-Like Structs**

#### **Tuple Structs**
**Tuple structs** have a name that adds semantic meaning, but their fields do not have individual names—only types. 

* **Type Safety Distinction**: Every tuple struct you define creates its own unique type, even if their inner field types are identical. A function expecting a `Color` cannot accept a `Point` argument.
* Fields are accessed via destructuring or dot indexing (`.0`, `.1`).

```rust
struct Color(i32, i32, i32);
struct Point(i32, i32, i32);

let black = Color(0, 0, 0);
let origin = Point(0, 0, 0);
```
*(Source: Tuple Struct Definitions)*

#### **Unit-Like Structs**
**Unit-like structs** have no fields at all. They behave similarly to the unit type `()` and are useful when you need to implement a trait on a type without needing to store data.

```rust
struct AlwaysEqual;
```

---

### **4. Ownership of Struct Data & Lifetimes**

In standard struct definitions, using owned types like **`String`** ensures the struct owns all of its internal data, keeping that data valid for as long as the struct instance exists.

If a struct stores references to data owned by something else (e.g., `&str`), Rust requires explicit **lifetime specifiers**. Lifetimes guarantee that the data referenced by the struct outlives the struct instance itself to prevent dangling pointers.

```rust
// THIS WILL NOT COMPILE WITHOUT LIFETIME ANNOTATIONS
struct User {
    username: &str, // Compiler Error: missing lifetime specifier
    email: &str,
    sign_in_count: u64,
    active: bool,
}
```
*(Source: Missing Lifetime Specifier)*

---

### **5. Refactoring Example & Debug Formatting**

To demonstrate structs in practice, consider a program that calculates the area of a rectangle.

#### **Refactoring from Tuples to Named Structs**
Using a struct groups related dimensions into a clear type rather than relying on tuple indexes like `.0` and `.1`.

```rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

fn main() {
    let rect1 = Rectangle { width: 30, height: 50 };

    println!(
        "The area of the rectangle is {} square pixels.",
        area(&rect1) // Borrowing rect1 immutably
    );
}

fn area(rectangle: &Rectangle) -> u32 {
    rectangle.width * rectangle.height
}
```
*(Source: Listing 5-10)*

#### **Derived Traits: Debug Printing (`#[derive(Debug)]`)**
By default, custom structs cannot be formatted with `println!("{}" ...)` because they do not implement the `Display` trait.
* Adding **`#[derive(Debug)]`** above the struct definition opts into the `Debug` trait.
* Use **`{:?}`** for single-line debug printing or **`{:#?}`** for multiline pretty-printing.

```rust
println!("rect1 is {:?}", rect1);  // Outputs: rect1 is Rectangle { width: 30, height: 50 }
println!("rect1 is {:#?}", rect1); // Outputs formatted multi-line struct layout
```
*(Source: Listing 5-12)*

---

### **6. Method Syntax & Implementation (`impl`) Blocks**

Methods are functions declared inside an **`impl` (implementation) block** associated with a struct. Unlike standard functions, a method's first parameter is always **`self`**, representing the instance the method is called on.

#### **Method Receiver Variants (`self`, `&self`, `&mut self`)**
* **`&self`**: Borrows the struct instance immutably (used for reading data).
* **`&mut self`**: Borrows the struct instance mutably (used to modify fields).
* **`self`**: Takes ownership of the instance, consuming it (rare; used when transforming `self` into another type).

#### **Defining Methods & Passing Arguments**
```rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

impl Rectangle {
    // Method taking immutable borrow of self
    fn area(&self) -> u32 {
        self.width * self.height
    }

    // Method with additional parameters
    fn can_hold(&self, other: &Rectangle) -> bool {
        self.width > other.width && self.height > other.height
    }
}

fn main() {
    let rect1 = Rectangle { width: 30, height: 50 };
    let rect2 = Rectangle { width: 10, height: 40 };

    println!("Area: {}", rect1.area());
    println!("Can rect1 hold rect2? {}", rect1.can_hold(&rect2));
}
```
*(Source: Listing 5-13 & Listing 5-15)*

---

### **7. Rust-Specific Concept: Automatic Referencing and Dereferencing**

In C and C++, calling methods directly on an object uses the dot operator (`.`), while calling methods on a pointer requires the arrow operator (`->`) to dereference it first.

**Rust does not have a `->` operator.** Instead, Rust features **automatic referencing and dereferencing**. When calling a method with `object.something()`, Rust automatically inserts `&`, `&mut`, or `*` so `object` matches the method signature:

```rust
p1.distance(&p2);
// Rust automatically converts this to:
(&p1).distance(&p2);
```
*(Source: Automatic Referencing)*

Because methods have a clear receiver type (`self`), Rust can determine whether the method is reading (`&self`), mutating (`&mut self`), or consuming (`self`), making method calls ergonomic.

---

### **8. Associated Functions and Multiple `impl` Blocks**

#### **Associated Functions**
Functions defined inside an `impl` block that **do not take `self` as a parameter** are called **associated functions**. They are associated with the struct type rather than a specific instance, making them ideal for constructors.

Associated functions are invoked using the **`::`** syntax (e.g., `Rectangle::square(3)` or `String::from(...)`).

```rust
impl Rectangle {
    // Associated constructor function creating a square Rectangle
    fn square(size: u32) -> Rectangle {
        Rectangle { width: size, height: size }
    }
}

fn main() {
    let sq = Rectangle::square(3);
}
```
*(Source: Associated Functions)*

#### **Multiple `impl` Blocks**
Rust allows a single struct to have multiple `impl` blocks. Splitting methods across multiple blocks is syntactically valid and useful when combining generic types and traits.

```rust
impl Rectangle {
    fn area(&self) -> u32 {
        self.width * self.height
    }
}

impl Rectangle {
    fn can_hold(&self, other: &Rectangle) -> bool {
        self.width > other.width && self.height > other.height
    }
}
```
*(Source: Listing 5-16)*

---
🦀 
## References
