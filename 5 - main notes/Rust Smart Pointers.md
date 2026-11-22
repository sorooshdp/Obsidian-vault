---
created: 2026-10-17 18:02
updated: 2026-10-17 18:02
type: note
status: seed
tags:
  - rust
aliases: []
topic: "[[Rust]]"
---

## Summary


## Notes

In **Chapter 15: Smart Pointers**, *The Rust Programming Language* explores specialized data structures that act like pointers while offering metadata and additional capabilities. While ordinary references (`&`) merely borrow data without overhead, smart pointers frequently **own the data they point to**. Standard library smart pointers are typically built using structs that implement the **`Deref`** and **`Drop`** traits.

---

### **1. Using `Box<T>` to Point to Data on the Heap**

The most straightforward smart pointer is **`Box<T>`**, which allocates values on the heap while keeping a pointer to that heap memory on the stack.

#### **A. Primary Use Cases for `Box<T>`**
1. **Unknown Sizes at Compile Time**: When a type's exact size cannot be determined at compile time (such as recursive types), but a context requires a known fixed size.
2. **Transferring Ownership of Large Data**: Copying large data on the stack takes time; placing it inside a `Box` ensures only the small pointer is copied on the stack while heap data stays in place.
3. **Trait Objects**: Owning a value that implements a specific trait rather than a concrete type.

#### **B. Enabling Recursive Types (The Cons List)**
At compile time, Rust must know how much memory a type occupies. A recursive type, where a value contains another value of the same type as part of itself, theoretically requires infinite memory.

By inserting a `Box<T>` into a recursive definition, the compiler knows the size of the variant because a pointer has a fixed size regardless of the data it points to.

##### **Recursive Cons List Code Example (`src/main.rs`)**
```rust
enum List {
    Cons(i32, Box<List>), // Box breaks the infinite size recursion
    Nil,
}

use List::{Cons, Nil};

fn main() {
    let list = Cons(1,
        Box::new(Cons(2,
            Box::new(Cons(3,
                Box::new(Nil))))));
}
```
*(Source: Listing 15-5)*

---

### **2. Treating Smart Pointers Like References with `Deref`**

Implementing the **`Deref` trait** allows a smart pointer to customize the behavior of the dereference operator (`*`), enabling smart pointers to be treated like standard references.

#### **A. How `Deref` Works Under the Hood**
When you apply `*y` to a custom smart pointer `y`, Rust automatically converts `*y` into `*(y.deref())`. The `.deref()` method returns a reference (`&T`) to the inner value so that ownership is not moved out of the smart pointer.

##### **Implementing `Deref` on a Custom Smart Pointer (`src/main.rs`)**
```rust
use std::ops::Deref;

struct MyBox<T>(T);

impl<T> MyBox<T> {
    fn new(x: T) -> MyBox<T> {
        MyBox(x)
    }
}

impl<T> Deref for MyBox<T> {
    type Target = T; // Associated type defining the target type

    fn deref(&self) -> &T {
        &self.0 // Returns reference to inner field
    }
}

fn main() {
    let x = 5;
    let y = MyBox::new(x);

    assert_eq!(5, x);
    assert_eq!(5, *y); // Dereferences MyBox using Deref trait
}
```
*(Source: Listing 15-8, 15-10)*

---

#### **B. Implicit Deref Coercion**
**Deref coercion** is an automatic convenience performed on arguments passed to functions or methods. It converts a reference to a type implementing `Deref` into a reference to a type that `Deref` can convert it into (e.g., converting `&MyBox<String>` into `&String`, and then into `&str`).

```rust
fn hello(name: &str) {
    println!("Hello, {}!", name);
}

fn main() {
    let m = MyBox::new(String::from("Rust"));
    hello(&m); // Automatically coerced from &MyBox<String> to &str
}
```
*(Source: Listing 15-11, 15-12)*

##### **Deref Coercion and Mutability Rules:**
* From `&T` to `&U` when `T: Deref<Target=U>`.
* From `&mut T` to `&mut U` when `T: DerefMut<Target=U>`.
* From `&mut T` to `&U` when `T: Deref<Target=U>`.
*(Immutable references will **never** coerce to mutable references because borrowing rules require mutable references to be exclusive)*.

---

### **3. Running Cleanup Code with the `Drop` Trait**

The **`Drop` trait** lets you customize what code runs automatically when a value is about to go out of scope, releasing resources such as heap memory, files, or network connections.

##### **Implementing `Drop` (`src/main.rs`)**
```rust
struct CustomSmartPointer {
    data: String,
}

impl Drop for CustomSmartPointer {
    fn drop(&mut self) {
        println!("Dropping CustomSmartPointer with data `{}`!", self.data);
    }
}

fn main() {
    let c = CustomSmartPointer { data: String::from("my stuff") };
    let d = CustomSmartPointer { data: String::from("other stuff") };
    println!("CustomSmartPointers created.");
} // d is dropped first, then c (reverse order of creation)
```
*(Source: Listing 15-14)*

#### **Early Cleanup with `std::mem::drop`**
Directly calling `c.drop()` is a compile-time error (`explicit use of destructor method`) because Rust would still call `drop` automatically at scope end, causing a **double free error**. To force early cleanup, call the prelude function **`std::mem::drop(c)`**.

---

### **4. `Rc<T>`: The Reference Counted Smart Pointer**

To support multiple owners of shared heap data in **single-threaded scenarios**, Rust provides **`Rc<T>`** (Reference Counting). `Rc<T>` tracks the number of active references; when the count reaches zero, the heap memory is freed.

* **`Rc::clone(&a)`**: Increments the reference count without making a deep copy of the underlying data.
* **`Rc::strong_count(&a)`**: Returns the current number of strong references.

##### **Sharing Data across Lists with `Rc<T>` (`src/main.rs`)**
```rust
enum List {
    Cons(i32, Rc<List>),
    Nil,
}

use List::{Cons, Nil};
use std::rc::Rc;

fn main() {
    let a = Rc::new(Cons(5, Rc::new(Cons(10, Rc::new(Nil)))));
    println!("count after creating a = {}", Rc::strong_count(&a)); // 1

    let b = Cons(3, Rc::clone(&a));
    println!("count after creating b = {}", Rc::strong_count(&a)); // 2

    {
        let c = Cons(4, Rc::clone(&a));
        println!("count after creating c = {}", Rc::strong_count(&a)); // 3
    }
    println!("count after c goes out of scope = {}", Rc::strong_count(&a)); // 2
}
```
*(Source: Listing 15-18, 15-19)*

---

### **5. `RefCell<T>` and the Interior Mutability Pattern**

**Interior mutability** is a design pattern in Rust that allows mutating data even when immutable references to that data exist.

#### **A. Compile-Time vs. Runtime Borrow Checking**
* **Standard References & `Box<T>`**: Borrowing rules are enforced at **compile time** (errors caught early, zero runtime overhead).
* **`RefCell<T>`**: Borrowing rules are enforced at **runtime**. Violating borrowing rules (such as creating two active `borrow_mut()` references in the same scope) causes the program to **panic** at runtime with a `BorrowMutError`.
* **Thread Safety**: Like `Rc<T>`, `RefCell<T>` is strictly for **single-threaded** contexts.

#### **B. Summary Comparison of Smart Pointers**
* **`Box<T>`**: Single ownership; immutable or mutable borrows checked at **compile time**.
* **`Rc<T>`**: Multiple ownership; **immutable** borrows checked at **compile time**.
* **`RefCell<T>`**: Single ownership; immutable or mutable borrows checked at **runtime**.

#### **C. Combining `Rc<T>` and `RefCell<T>` for Multiple Owners of Mutable Data**
By wrapping a `RefCell<T>` inside an `Rc<T>` (`Rc<RefCell<T>>`), you can construct values that have multiple owners and can also be mutated.

##### **Code Example: `Rc<RefCell<i32>>` (`src/main.rs`)**
```rust
#[derive(Debug)]
enum List {
    Cons(Rc<RefCell<i32>>, Rc<List>),
    Nil,
}

use List::{Cons, Nil};
use std::rc::Rc;
use std::cell::RefCell;

fn main() {
    let value = Rc::new(RefCell::new(5));

    let a = Rc::new(Cons(Rc::clone(&value), Rc::new(Nil)));
    let b = Cons(Rc::new(RefCell::new(6)), Rc::clone(&a));
    let c = Cons(Rc::new(RefCell::new(10)), Rc::clone(&a));

    *value.borrow_mut() += 10; // Mutates inner value from 5 to 15

    println!("a after = {:?}", a); // Reflects value 15 across all shared lists
}
```
*(Source: Listing 15-24)*

---

### **6. Reference Cycles and Preventing Memory Leaks with `Weak<T>`**

Rust allows memory leaks when `Rc<T>` and `RefCell<T>` form cyclic references where items point to each other in a loop. Because the reference count never reaches zero, neither item is dropped.

#### **A. Weak References (`Weak<T>`)**
To prevent reference cycles, convert an `Rc<T>` into a **`Weak<T>`** by calling **`Rc::downgrade(&rc)`**.
* **`strong_count`**: Tracks shared ownership. Must be 0 for the heap memory to be cleaned up.
* **`weak_count`**: Tracks non-owning references. Does not prevent the value from being dropped when `strong_count` reaches 0.
* **`upgrade()` Method**: Because the referenced data may have been dropped, calling `.upgrade()` on a `Weak<T>` returns an **`Option<Rc<T>>`** (`Some(Rc)` if valid, `None` if dropped).

#### **B. Building a Tree Data Structure (Child-to-Parent References)**
Parent nodes should own child nodes (`children: RefCell<Vec<Rc<Node>>>`), but child nodes should only hold weak references to their parents (`parent: RefCell<Weak<Node>>`) to avoid reference cycles.

##### **Tree Node Code Example (`src/main.rs`)**
```rust
use std::rc::{Rc, Weak};
use std::cell::RefCell;

#[derive(Debug)]
struct Node {
    value: i32,
    parent: RefCell<Weak<Node>>,
    children: RefCell<Vec<Rc<Node>>>,
}

fn main() {
    let leaf = Rc::new(Node {
        value: 3,
        parent: RefCell::new(Weak::new()),
        children: RefCell::new(vec![]),
    });

    let branch = Rc::new(Node {
        value: 5,
        parent: RefCell::new(Weak::new()),
        children: RefCell::new(vec![Rc::clone(&leaf)]),
    });

    *leaf.parent.borrow_mut() = Rc::downgrade(&branch); // Creates non-owning weak link

    println!("leaf parent = {:?}", leaf.parent.borrow().upgrade()); // Upgrades to Some(branch)
}
```
*(Source: Listing 15-27, 15-28)*

---
🦀 
## References
