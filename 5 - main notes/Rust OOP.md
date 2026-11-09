---
created: 2026-10-17 18:33
updated: 2026-10-17 18:33
type: note
status: seed
tags:
  - rust
aliases: []
---

## Summary


## Notes

In **Chapter 17: Object-Oriented Programming Features of Rust**, *The Rust Programming Language* examines how common Object-Oriented Programming (OOP) concepts translate into idiomatic Rust, how **trait objects** provide dynamic polymorphism, and how to balance traditional OO design patterns against idiomatic Rust type-system designs.

---

### **1. Characteristics of Object-Oriented Languages in Rust**

Rust is influenced by many paradigms, including OOP. Whether Rust is "object-oriented" depends on how OOP characteristics are defined:

* **Objects Contain Data and Behavior**: Structs and enums hold data, while **`impl` blocks** define methods on those types. According to the Gang of Four definition of objects, Rust structs and enums provide the same functionality.
* **Encapsulation That Hides Implementation Details**: Rust controls visibility using the **`pub`** keyword. By default, fields, structs, modules, and methods are private. Encapsulation allows internal implementations to change without breaking external caller code.
* **Inheritance**: Rust **does not have traditional class-based inheritance**—a struct cannot inherit fields or method implementations from a parent struct. Instead, Rust achieves OOP goals through alternatives:
  * **Code Reuse**: Replaced by **default trait method implementations**.
  * **Polymorphism**: Replaced by **generics with trait bounds** (*bounded parametric polymorphism*) and **trait objects**.

#### **Encapsulation Example: `AveragedCollection` (`src/lib.rs`)**
The `AveragedCollection` struct encapsulates its `list` and cached `average` fields as private so that external code cannot mutate `list` without automatically recomputing `average`.

```rust
pub struct AveragedCollection {
    list: Vec<i32>,
    average: f64,
}

impl AveragedCollection {
    pub fn add(&mut self, value: i32) {
        self.list.push(value);
        self.update_average();
    }

    pub fn remove(&mut self) -> Option<i32> {
        let result = self.list.pop();
        match result {
            Some(value) => {
                self.update_average();
                Some(value)
            },
            None => None,
        }
    }

    pub fn average(&self) -> f64 {
        self.average
    }

    fn update_average(&mut self) {
        let total: i32 = self.list.iter().sum();
        self.average = total as f64 / self.list.len() as f64;
    }
}
```
*(Source: Listing 17-1 & Listing 17-2)*

---

### **2. Trait Objects & Dynamic Dispatch**

Vectors in Rust normally store elements of a single homogenous type. To allow collections of differing concrete types that share a common behavior, Rust uses **trait objects**.

#### **A. Generics vs. Trait Objects**
* **Generics (`Screen<T: Draw>`)**: Use **monomorphization** to generate concrete code at compile time, achieving **static dispatch** with zero runtime performance cost. However, a generic `Vec<T>` must be **homogeneous** (e.g., all `Button` or all `TextField`).
* **Trait Objects (`Box<Draw>` / `Box<dyn Draw>`)**: Pointer-based abstractions (`Box<Draw>` or `&Draw`) that point to instances implementing the trait. They allow **heterogeneous collections** holding different concrete types in the same `Vec`.

#### **B. Dynamic Dispatch & "Duck Typing"**
Because the compiler cannot know all concrete types used behind trait objects at compile time, it uses **dynamic dispatch**. At runtime, Rust uses pointers inside the trait object (*vtable*) to look up which method to call. This incurs a slight runtime lookup cost and prevents method inlining, but provides runtime flexibility similar to **duck typing**, while remaining fully type-checked at compile time.

#### **Code Example: A GUI Library Crate (`src/lib.rs`)**
```rust
pub trait Draw {
    fn draw(&self);
}

pub struct Screen {
    // Vector storing heterogeneous trait objects that implement Draw
    pub components: Vec<Box<Draw>>,
}

impl Screen {
    pub fn run(&self) {
        for component in self.components.iter() {
            component.draw();
        }
    }
}

pub struct Button {
    pub width: u32,
    pub height: u32,
    pub label: String,
}

impl Draw for Button {
    fn draw(&self) {
        // Code to draw a button
    }
}
```
*(Source: Listings 17-3, 17-4, 17-5, 17-7)*

#### **Extending the Library with a Custom Type (`src/main.rs`)**
External consumers can define new types like `SelectBox`, implement `Draw`, and pass them to `Screen` without modifying the library.

```rust
extern crate gui;
use gui::{Screen, Button, Draw};

struct SelectBox {
    width: u32,
    height: u32,
    options: Vec<String>,
}

impl Draw for SelectBox {
    fn draw(&self) {
        // Code to draw a select box
    }
}

fn main() {
    let screen = Screen {
        components: vec![
            Box::new(SelectBox {
                width: 75,
                height: 10,
                options: vec![
                    String::from("Yes"),
                    String::from("Maybe"),
                    String::from("No"),
                ],
            }),
            Box::new(Button {
                width: 50,
                height: 10,
                label: String::from("OK"),
            }),
        ],
    };

    screen.run();
}
```
*(Source: Listings 17-8 & 17-9)*

---

#### **C. Object Safety Rules for Trait Objects**
You can only create trait objects from **object-safe traits**. A trait is object-safe if all methods defined in the trait meet two criteria:
1. **The return type is not `Self`**.
2. **There are no generic type parameters**.

If a trait method returns `Self` (such as `Clone::clone(&self) -> Self`), the trait object forgets the original concrete type, making it impossible for Rust to know what type `Self` should resolve to. Attempting to use `Box<Clone>` triggers compile error **`E0038`**.

---

### **3. Implementing the Object-Oriented State Pattern**

The **State Pattern** is an OO design pattern where an object's behavior changes based on its internal state, represented by a set of state objects.

#### **The Blog Post Workflow Requirements:**
1. A post starts as an empty `Draft`.
2. Requesting review moves the post to `PendingReview`.
3. Approving the post publishes it (`Published`).
4. Only `Published` posts return non-empty text via `.content()`.

#### **Code Example: OO State Pattern Implementation (`src/lib.rs`)**
```rust
pub struct Post {
    state: Option<Box<State>>,
    content: String,
}

impl Post {
    pub fn new() -> Post {
        Post {
            state: Some(Box::new(Draft {})),
            content: String::new(),
        }
    }

    pub fn add_text(&mut self, text: &str) {
        self.content.push_str(text);
    }

    pub fn content(&self) -> &str {
        self.state.as_ref().unwrap().content(self)
    }

    pub fn request_review(&mut self) {
        if let Some(s) = self.state.take() { // Takes ownership of old state
            self.state = Some(s.request_review())
        }
    }

    pub fn approve(&mut self) {
        if let Some(s) = self.state.take() {
            self.state = Some(s.approve())
        }
    }
}

trait State {
    fn request_review(self: Box<Self>) -> Box<State>;
    fn approve(self: Box<Self>) -> Box<State>;
    fn content<'a>(&self, post: &'a Post) -> &'a str {
        ""
    }
}

struct Draft {}

impl State for Draft {
    fn request_review(self: Box<Self>) -> Box<State> {
        Box::new(PendingReview {})
    }

    fn approve(self: Box<Self>) -> Box<State> {
        self
    }
}

struct PendingReview {}

impl State for PendingReview {
    fn request_review(self: Box<Self>) -> Box<State> {
        self
    }

    fn approve(self: Box<Self>) -> Box<State> {
        Box::new(Published {})
    }
}

struct Published {}

impl State for Published {
    fn request_review(self: Box<Self>) -> Box<State> {
        self
    }

    fn approve(self: Box<Self>) -> Box<State> {
        self
    }

    fn content<'a>(&self, post: &'a Post) -> &'a str {
        &post.content
    }
}
```
*(Source: Listings 17-12 through 17-18)*

* **`self.state.take()`**: Extracts the `Some(state)` out of `Post` while leaving a temporary `None`, allowing `Post` to consume and transform the old state without violating ownership rules.

#### **Trade-offs of the OO State Pattern:**
* **Pros**: Encapsulates state rules in separate state structs. Adding a new state requires modifying only a single struct.
* **Cons**: Duplication of delegating methods, tight coupling between state transitions, runtime allocation overhead, and reliance on runtime `unwrap()` checks.

---

### **4. Idiomatic Rust Alternative: Encoding States as Types**

Rather than managing state changes inside a mutable container at runtime, idiomatic Rust can encode states **directly into distinct types**. This turns invalid state operations into **compile-time errors**.

#### **Code Example: Type-State Pattern (`src/lib.rs` & `src/main.rs`)**
```rust
pub struct Post {
    content: String,
}

pub struct DraftPost {
    content: String,
}

pub struct PendingReviewPost {
    content: String,
}

impl Post {
    pub fn new() -> DraftPost {
        DraftPost {
            content: String::new(),
        }
    }

    pub fn content(&self) -> &str {
        &self.content
    }
}

impl DraftPost {
    pub fn add_text(&mut self, text: &str) {
        self.content.push_str(text);
    }

    // Takes ownership of DraftPost and transforms it into PendingReviewPost
    pub fn request_review(self) -> PendingReviewPost {
        PendingReviewPost {
            content: self.content,
        }
    }
}

impl PendingReviewPost {
    // Takes ownership of PendingReviewPost and transforms it into Post
    pub fn approve(self) -> Post {
        Post {
            content: self.content,
        }
    }
}
```
*(Source: Listings 17-19 & 17-20)*

#### **Usage in Main (`src/main.rs`)**
```rust
extern crate blog;
use blog::Post;

fn main() {
    let mut post = Post::new();

    post.add_text("I ate a salad for lunch today");

    let post = post.request_review(); // DraftPost moved into PendingReviewPost
    let post = post.approve();        // PendingReviewPost moved into Post

    assert_eq!("I ate a salad for lunch today", post.content());
}
```
*(Source: Listing 17-21)*

#### **Why the Type-State Approach Is Preferred in Rust:**
* Neither `DraftPost` nor `PendingReviewPost` define a `.content()` method. Attempting to call `.content()` on an unapproved post causes a **compile-time error**, preventing bugs before code ever reaches production.
* Method calls take ownership (`self`), invalidating old state variables so they cannot be reused.

---
🦀 
## References
