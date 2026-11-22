---
created: 2026-10-13 20:36
updated: 2026-10-13 20:36
type: note
status: seed
tags:
  - rust
aliases: []
topic: "[[Rust]]"
---

## Summary


## Notes
In **Chapter 3: Common Programming Concepts**, *The Rust Programming Language* details the foundational constructs that form the backbone of almost every Rust program. It covers variable mutability, data types, functions, statements versus expressions, and control flow constructs.

---

### **1. Variables, Mutability, Constants, and Shadowing**

#### **Variables & Mutability**
By default, variables in Rust are **immutable**. Once a value is bound to a name, it cannot be reassigned. This safety guarantee prevents subtle bugs where one part of the code mutates state unexpectedly.

To opt into mutability, you must explicitly use the **`mut`** keyword.

```rust
fn main() {
    let mut x = 5;
    println!("The value of x is: {}", x);
    x = 6;
    println!("The value of x is: {}", x);
}
```
#### **Differences Between Variables and Constants**
Constants resemble immutable variables, but with key distinctions:
1. You **cannot** use `mut` with constants—they are permanently immutable.
2. Constants are declared with **`const`** instead of `let`, and their type **must** be explicitly annotated.
3. Constants can be declared in any scope, including the global scope.
4. Constants can only be set to **constant expressions** known at compile time, not runtime function evaluations.

```rust
const MAX_POINTS: u32 = 100_000;
```

#### **Shadowing**
Rust allows you to declare a new variable with the same name as a previous variable, **shadowing** the original variable. 

* **Difference from `mut`**: Re-declaring a variable with `let` creates an entirely new variable. This allows you to **change the type** of the bound value while preserving the same variable name.

```rust
fn main() {
    let x = 5;
    let x = x + 1; // x becomes 6
    let x = x * 2; // x becomes 12
    println!("The value of x is: {}", x);

    let spaces = "   ";        // String slice (&str)
    let spaces = spaces.len(); // Shadowed to an integer (usize)
}
```

---

### **2. Data Types**

Rust is a **statically typed** language, meaning the compiler must know the type of every variable at compile time. Types are divided into two main subsets: **scalar** and **compound**.

#### **A. Scalar Types**
A scalar type represents a single value:

1. **Integers**: Numbers without fractional components.
   * **Signed (`i8`, `i16`, `i32`, `i64`, `isize`)** and **Unsigned (`u8`, `u16`, `u32`, `u64`, `usize`)**.
   * Default integer type is **`i32`**. `isize` and `usize` depend on the host machine architecture and are used for indexing collections.
   * Supports literals in Decimal (`98_222`), Hex (`0xff`), Octal (`0o77`), Binary (`0b1111_0000`), and Byte (`b'A'`).

2. **Floating-Point Numbers**:
   * Primitive float types are **`f32`** (single precision) and **`f64`** (double precision, default).

3. **Booleans**:
   * Represented by **`bool`**, taking values `true` or `false`.

4. **Characters**:
   * The **`char`** type uses single quotes (`'z'`, `'😻'`) and represents a **Unicode Scalar Value** (4 bytes in size).

```rust
fn main() {
    // Integer and Float
    let x: i32 = -42;
    let y: f64 = 3.14;

    // Boolean and Character
    let is_active: bool = true;
    let emoji: char = '😻';
}
```

#### **B. Compound Types**
Compound types group multiple values into one structure:

1. **The Tuple Type**:
   * Fixed-length group of values that can have **different types**.
   * Destructured using pattern matching or accessed via dot notation (`.0`, `.1`).

```rust
fn main() {
    let tup: (i32, f64, u8) = (500, 6.4, 1);

    let (x, y, z) = tup; // Destructuring pattern
    println!("The value of y is: {}", y);

    let five_hundred = tup.0; // Index access
}
```

2. **The Array Type**:
   * Fixed-length collection where **every element must be of the same type** and stored on the stack.
   * Elements are accessed using square brackets (`a`). Out-of-bounds access triggers a runtime panic.

```rust
fn main() {
    let a =;
    let first = a;
}
```

---

### **3. Functions, Statements, and Expressions**

Functions are declared using the **`fn`** keyword, and names follow **`snake_case`** convention.

#### **Function Parameters**
Parameters require explicit type annotations in the function signature.

```rust
fn another_function(x: i32, y: i32) {
    println!("The value of x is: {}", x);
    println!("The value of y is: {}", y);
}
```

#### **Statements vs. Expressions**
Function bodies consist of statements optionally ending in an expression:
* **Statements**: Instructions that perform an action and **do not return a value** (e.g., `let x = 6;`).
* **Expressions**: Evaluate to a resulting value. A block `{}` is an expression.
* **Rule**: Expressions **do not** include ending semicolons. Adding a semicolon turns an expression into a statement, discarding its return value.

```rust
fn main() {
    let y = {
        let x = 3;
        x + 1 // Expression (no semicolon) evaluates to 4
    };

    println!("The value of y is: {}", y);
}
```

#### **Functions with Return Values**
Return types are declared after an arrow **`->`**. Functions implicitly return the value of their final expression, or early via the `return` keyword.

```rust
fn five() -> i32 {
    5 // Final expression returned implicitly
}

fn plus_one(x: i32) -> i32 {
    x + 1 // Adding a semicolon here would cause a compile error
}
```

---

### **4. Comments**

In Rust, standard comments start with two slashes (`//`) and extend to the end of the line.

```rust
// This is a simple line comment in Rust
let x = 5; // Can also be placed inline
```

---

### **5. Control Flow**

#### **`if` Expressions**
An `if` expression evaluates a condition to branch execution. The condition **must be a `bool`**; Rust does not convert non-Boolean types automatically.

Because `if` is an expression, it can be used on the right side of a `let` statement. However, all potential branches must evaluate to the **same data type**.

```rust
fn main() {
    let number = 6;

    if number % 4 == 0 {
        println!("number is divisible by 4");
    } else if number % 3 == 0 {
        println!("number is divisible by 3");
    } else {
        println!("number is not divisible by 4 or 3");
    }

    // Using `if` in a `let` statement
    let condition = true;
    let number = if condition { 5 } else { 6 };
}
```

#### **Repetition with Loops**

1. **`loop`**: Executes a block infinitely until stopped manually or exited using **`break`**.

```rust
fn main() {
    let mut count = 0;
    loop {
        count += 1;
        if count == 3 {
            break; // Exits the infinite loop
        }
    }
}
```

2. **`while`**: Evaluates a condition before each iteration, terminating when the condition evaluates to `false`.

```rust
fn main() {
    let mut number = 3;

    while number != 0 {
        println!("{}!", number);
        number -= 1;
    }

    println!("LIFTOFF!!!");
}
```

3. **`for`**: Safely and efficiently iterates over elements of a collection or a range.

```rust
fn main() {
    let a =;

    // Iterating over an array safely
    for element in a.iter() {
        println!("the value is: {}", element);
    }

    // Iterating over a reversed range (3 down to 1)
    for number in (1..4).rev() {
        println!("{}!", number);
    }
    println!("LIFTOFF!!!");
}
```


---

🦀 

## References
