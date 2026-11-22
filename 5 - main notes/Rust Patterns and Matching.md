---
created: 2026-10-17 20:29
updated: 2026-10-17 20:29
type: note
status: seed
tags:
  - rust
aliases: []
topic: "[[Rust]]"
---

## Summary


## Notes
In **Chapter 18: Patterns and Matching**, *The Rust Programming Language* provides a comprehensive reference on patterns—special syntax in Rust for matching against the structure of simple and complex types. Patterns allow fine-grained control over program flow by matching data shapes against literals, destructured types, variables, wildcards, and placeholders.

---

### **1. All the Places Patterns Can Be Used**

Patterns appear throughout Rust, often in constructs you use every day:

#### **A. `match` Arms**
`match` expressions use patterns in each arm:
```rust
match VALUE {
    PATTERN => EXPRESSION,
    PATTERN => EXPRESSION,
}
```
* **Exhaustiveness**: `match` expressions must account for every possible value. The wildcard `_` matches any value without binding to a variable, making it ideal as a catchall last arm.

#### **B. Conditional `if let` Expressions**
`if let` expressions provide a concise way to write a single-pattern match. They can be combined with `else if`, `else if let`, and `else` blocks:

```rust
fn main() {
    let favorite_color: Option<&str> = None;
    let is_tuesday = false;
    let age: Result<u8, _> = "34".parse();

    if let Some(color) = favorite_color {
        println!("Using favorite color: {}", color);
    } else if is_tuesday {
        println!("Tuesday is green day!");
    } else if let Ok(age) = age {
        if age > 30 {
            println!("Using purple");
        } else {
            println!("Using orange");
        }
    } else {
        println!("Using blue");
    }
}
```
*(Source: Listing 18-1)*

* **Trade-off**: Unlike `match`, `if let` does **not** enforce compile-time exhaustiveness checking.

#### **C. `while let` Conditional Loops**
A `while let` loop continues running for as long as a pattern continues to match.

```rust
let mut stack = Vec::new();
stack.push(1);
stack.push(2);
stack.push(3);

while let Some(top) = stack.pop() {
    println!("{}", top);
}
```
*(Source: Listing 18-2)*

#### **D. `for` Loops**
The value directly following the `for` keyword is a pattern. For instance, calling `.enumerate()` produces a tuple that can be destructured directly in the loop head:

```rust
let v = vec!['a', 'b', 'c'];

for (index, value) in v.iter().enumerate() {
    println!("{} is at index {}", value, index);
}
```
*(Source: Listing 18-3)*

#### **E. `let` Statements**
Standard variable declarations formally use pattern matching:
```rust
let PATTERN = EXPRESSION;
```
For example, a tuple can be destructured into multiple variables in a single `let` statement:

```rust
let (x, y, z) = (1, 2, 3);
```
*(Source: Listing 18-4)*

#### **F. Function and Closure Parameters**
Function signatures and closure parameter lists use patterns to destructure incoming arguments:

```rust
fn print_coordinates(&(x, y): &(i32, i32)) {
    println!("Current location: ({}, {})", x, y);
}

fn main() {
    let point = (3, 5);
    print_coordinates(&point);
}
```
*(Source: Listing 18-7)*

---

### **2. Refutability: Whether a Pattern Might Fail to Match**

Patterns exist in two forms based on whether they can fail to match:

* **Irrefutable Patterns**: Patterns that **will match for any possible value** passed and cannot fail (e.g., `x` in `let x = 5;`).
  * **Required Contexts**: Function parameters, `let` statements, and `for` loops **must** use irrefutable patterns because execution cannot handle a failure state.
* **Refutable Patterns**: Patterns that **can fail to match** for some possible values (e.g., `Some(x)` in `if let Some(x) = a_value`, which fails if `a_value` is `None`).
  * **Accepted Contexts**: `if let`, `while let`, and `match` arms accept refutable patterns because conditional branching is designed to handle failure.

Attempting to use a refutable pattern in a `let` statement (e.g., `let Some(x) = some_option;`) causes a compile error because `None` is not covered. Conversely, using an irrefutable pattern in `if let` (e.g., `if let x = 5`) triggers a compiler warning/error because the conditional check is redundant.

---

### **3. Comprehensive Pattern Syntax**

#### **A. Matching Literals and Named Variables**
You can match patterns directly against literal values. However, matching against named variables introduces **variable shadowing** because `match` creates a new scope.

```rust
fn main() {
    let x = Some(5);
    let y = 10;

    match x {
        Some(50) => println!("Got 50"),
        Some(y) => println!("Matched, y = {:?}", y), // Shadows outer y with 5!
        _ => println!("Default case, x = {:?}", x),
    }

    println!("at the end: x = {:?}, y = {:?}", x, y); // Prints outer y = 10
}
```
*(Source: Listing 18-11)*

#### **B. Multiple Patterns (`|`) and Ranges (`...`)**
* **Multiple Patterns**: Use the `|` (OR) operator to match one of several pattern options:
  ```rust
  let x = 1;
  match x {
      1 | 2 => println!("one or two"),
      _ => println!("anything"),
  }
  ```
  *(Source: Multiple Patterns)*

* **Ranges**: Use the `...` syntax to match an inclusive range of numbers or characters:
  ```rust
  let x = 'c';
  match x {
      'a' ... 'j' => println!("early ASCII letter"),
      'k' ... 'z' => println!("late ASCII letter"),
      _ => println!("something else"),
  }
  ```
  *(Source: Matching Ranges)*

---

#### **C. Destructuring Values**

1. **Destructuring Structs**: Extract fields into named variables using matching field syntax or struct field shorthand:
   ```rust
   struct Point { x: i32, y: i32 }

   let p = Point { x: 0, y: 7 };

   // Shorthand syntax
   let Point { x, y } = p;

   // Destructuring with literal match conditions
   match p {
       Point { x, y: 0 } => println!("On x axis at {}", x),
       Point { x: 0, y } => println!("On y axis at {}", y),
       Point { x, y } => println!("On neither axis: ({}, {})", x, y),
   }
   ```
   *(Source: Listing 18-13 & 18-14)*

2. **Destructuring Enums**: Matches the structure of the data inside enum variants (unit, struct-like, or tuple-like):
   ```rust
   enum Message {
       Quit,
       Move { x: i32, y: i32 },
       Write(String),
       ChangeColor(i32, i32, i32),
   }

   match msg {
       Message::Quit => println!("Quit"),
       Message::Move { x, y } => println!("Move x: {}, y: {}", x, y),
       Message::Write(text) => println!("Text: {}", text),
       Message::ChangeColor(r, g, b) => println!("Color: {}, {}, {}", r, g, b),
   }
   ```
   *(Source: Listing 18-15)*

3. **Destructuring References**: Specify `&` in the pattern to extract the value the reference points to directly:
   ```rust
   let points = vec![Point { x: 0, y: 0 }, Point { x: 1, y: 5 }];
   let sum_of_squares: i32 = points
       .iter()
       .map(|&Point { x, y }| x * x + y * y)
       .sum();
   ```
   *(Source: Listing 18-16)*

4. **Destructuring Nested Structures**: Mix and match patterns to unpack nested tuples and structs simultaneously:
   ```rust
   let ((feet, inches), Point { x, y }) = ((3, 10), Point { x: 3, y: -10 });
   ```
   *(Source: Destructuring Structs and Tuples)*

---

#### **D. Ignoring Values in a Pattern**

1. **Ignoring Entire Values (`_`)**: Use `_` as a wildcard parameter or catchall arm that does not bind to the value.
2. **Ignoring Nested Parts**: Place `_` inside tuples or structs to evaluate specific elements while ignoring others:
   ```rust
   let numbers = (2, 4, 8, 16, 32);
   match numbers {
       (first, _, third, _, fifth) => println!("{}, {}, {}", first, third, fifth),
   }
   ```
   *(Source: Listing 18-19)*
3. **Ignoring Unused Variables (`_name` vs `_`)**: Prefixing a variable name with an underscore (`_x`) suppresses compiler unused variable warnings.
   * **Crucial Difference**: `_x` **still binds/moves** the value into the variable, whereas `_` **does not bind at all**.
4. **Ignoring Remaining Parts (`..`)**: Expands to ignore any unspecified fields or tuple values:
   ```rust
   struct Point { x: i32, y: i32, z: i32 }
   let origin = Point { x: 0, y: 0, z: 0 };

   match origin {
       Point { x, .. } => println!("x is {}", x),
   }
   ```
   *(Source: Listing 18-23)*
   * **Ambiguity Constraint**: `..` can be used **only once** per tuple or tuple struct pattern; multiple `..` in a single pattern causes a compile error.

---

#### **E. Creating References in Patterns (`ref` and `ref mut`)**

In a pattern, specifying `&` matches an existing reference. To **borrow a reference** to a value instead of moving ownership into the pattern variable, use the **`ref`** keyword. Use **`ref mut`** to create a mutable reference.

```rust
let mut robot_name = Some(String::from("Bors"));

match robot_name {
    // Borrows a mutable reference without moving ownership out of robot_name
    Some(ref mut name) => *name = String::from("Another name"),
    None => (),
}

println!("robot_name is: {:?}", robot_name); // Prints: Some("Another name")
```
*(Source: Listings 18-27 & 18-28)*

---

#### **F. Match Guards**

A **match guard** is an additional `if` condition specified after a pattern in a match arm that must evaluate to `true` for that arm to be selected.

##### **Solving Pattern Shadowing with Match Guards**
By replacing `Some(y)` with `Some(n) if n == y`, the match guard checks equality with the outer `y` variable without shadowing it:

```rust
fn main() {
    let x = Some(5);
    let y = 10;

    match x {
        Some(50) => println!("Got 50"),
        Some(n) if n == y => println!("Matched, n = {:?}", n), // Checks outer y!
        _ => println!("Default case, x = {:?}", x),
    }
}
```
*(Source: Listing 18-30)*

* **Operator Precedence**: When combining multiple patterns (`|`) with a match guard (`4 | 5 | 6 if y`), the match guard applies to the **entire pattern** (`(4 | 5 | 6) if y`).

---

#### **G. `@` Bindings**

The **`@` (at) operator** lets you test a value against a pattern while simultaneously capturing that value inside a variable.

```rust
enum Message {
    Hello { id: i32 },
}

let msg = Message::Hello { id: 5 };

match msg {
    // Tests if id is in range 3...7 AND binds the matched value to id_variable
    Message::Hello { id: id_variable @ 3...7 } => {
        println!("Found an id in range: {}", id_variable);
    },
    Message::Hello { id: 10...12 } => {
        println!("Found an id in another range"); // Value of id is not bound
    },
    Message::Hello { id } => {
        println!("Found some other id: {}", id);
    },
}
```
*(Source: Listing 18-32)*

---
🦀 

## References
