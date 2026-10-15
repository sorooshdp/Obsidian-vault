---
created: 2026-10-15 13:18
updated: 2026-10-15 13:18
type: note
status: seed
tags:
  - rust
aliases: []
---

## Summary


## Notes

In **Chapter 6: Enums and Pattern Matching**, _The Rust Programming Language_ introduces enumerations (enums) and pattern matching constructs (`match` and `if let`). Enums allow you to construct custom types by listing every possible variant a value can assume.

---

### **1. Defining an Enum and Attaching Associated Data**

#### **A. Enums vs. Structs**

An **enum** defines a custom data type that can be one of a set of enumerated variants. Unlike structs (which group multiple values together simultaneously), an enum instance can only be **one of its variants at any given time**.

#### **B. Embedding Data Directly inside Variants**

A major Rust-specific feature is that **enum variants can store associated data directly** without needing an auxiliary struct. Furthermore, each variant can hold **different types and amounts of data**:

- A variant can hold no data at all (like a unit struct).
- A variant can hold primitive values, tuples, strings, structs, or even other enums.

#### **C. Methods on Enums (`impl`)**

Just like structs, enums can have methods and associated functions defined on them using an **`impl` block**. Inside methods, **`self`** refers to the specific enum variant instance on which the method was invoked.

```rust
// Defining an Enum with variants containing different data types
enum Message {
    Quit,                             // No associated data
    Move { x: i32, y: i32 },          // Anonymous struct fields
    Write(String),                    // Single String
    ChangeColor(i32, i32, i32),       // Tuple of three i32 values
}

// Defining a method on an enum
impl Message {
    fn call(&self) {
        // Method body executes using self
    }
}

fn main() {
    let m = Message::Write(String::from("hello"));
    m.call();
}
```

_(Source: Listing 6-2 & Enum Methods)_

---

### **2. The `Option<T>` Enum: Replacing Null with Type Safety**

#### **A. Eliminating the "Billion-Dollar Mistake"**

Rust **does not have a `null` feature**. In languages with `null`, variables can always be in a `null` or `not-null` state, leading to runtime crashes when attempting to use a `null` value as if it were valid.

#### **B. The `Option<T>` Definition**

To represent the concept of a value being present or absent safely, Rust provides the generic **`Option<T>`** `enum` in its standard library prelude:

```rust
enum Option<T> {
    Some(T),
    None,
}
```

_(Source: Standard Library `Option<T>` Definition)_

#### **C. Strict Type Separation**

Because **`Option<T>` and `T` are entirely different types**, the compiler prevents treating an `Option<T>` as a valid value directly. You cannot add an `i8` to an `Option<i8>` without explicitly handling the `Option` wrapper first, ensuring that code handles missing values at compile time.

```rust
let some_number = Some(5);
let some_string = Some("a string");

// Explicit type annotation is required when using None
let absent_number: Option<i32> = None;
```

_(Source: Using `Option<T>`)_

---

### **3. The `match` Control Flow Operator**

The **`match`** operator compares a value against a sequence of **patterns** and executes the code block associated with the first matching pattern.

#### **A. Patterns That Bind to Values**

Match arms can **bind to variables** inside enum variants, making it easy to extract and use inner data stored inside a variant.

```rust
#[derive(Debug)]
enum UsState {
    Alabama,
    Alaska,
    // --snip--
}

enum Coin {
    Penny,
    Nickel,
    Dime,
    Quarter(UsState),
}

fn value_in_cents(coin: Coin) -> u32 {
    match coin {
        Coin::Penny => 1,
        Coin::Nickel => 5,
        Coin::Dime => 10,
        Coin::Quarter(state) => {
            println!("State quarter from {:?}!", state);
            25
        },
    }
}
```

_(Source: Listing 6-4 & Patterns That Bind to Values)_

#### **B. Matching with `Option<T>`**

The standard way to extract the inner `T` value from `Some(T)` is by pattern matching with `match`:

```rust
fn plus_one(x: Option<i32>) -> Option<i32> {
    match x {
        None => None,
        Some(i) => Some(i + 1),
    }
}

let five = Some(5);
let six = plus_one(five);  // Returns Some(6)
let none = plus_one(None); // Returns None
```

_(Source: Listing 6-5)_

#### **C. Matches Are Exhaustive**

In Rust, **matches are strictly exhaustive**. Every possible variant or outcome must be accounted for in a `match` expression; omitting a variant (such as forgetting `None`) causes a compile-time error.

#### **D. The `_` Catchall Placeholder**

To satisfy exhaustiveness when you only care about specific values, you can place the **`_` placeholder** as the final match arm to match all remaining cases. `()` represents the unit value to perform no operation.

```rust
let some_u8_value = 0u8;
match some_u8_value {
    1 => println!("one"),
    3 => println!("three"),
    5 => println!("five"),
    7 => println!("seven"),
    _ => (), // Catchall for all other u8 values (0..=255)
}
```

_(Source: The `_` Placeholder)_

---

### **4. Concise Control Flow with `if let`**

#### **A. Syntactic Sugar for Single-Pattern Matching**

When you only care about handling one specific pattern and ignoring all others, a full `match` expression requires verbose boilerplate like `_ => ()`. The **`if let`** construct combines `if` and `let` into a concise shorthand:

```rust
let some_u8_value = Some(0u8);

// Concise syntax replacing a match for a single variant
if let Some(3) = some_u8_value {
    println!("three");
}
```

_(Source: Listing 6-6 & `if let`)_

#### **B. Adding `else` Blocks**

An `else` block can be attached to an `if let` statement to execute code for unhandled variants (behaving identically to the `_` wildcard arm in a `match`):

```rust
let mut count = 0;

if let Coin::Quarter(state) = coin {
    println!("State quarter from {:?}!", state);
} else {
    count += 1;
}
```

_(Source: `if let` with `else`)_

---
🦀 
## References
