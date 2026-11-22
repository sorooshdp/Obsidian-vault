---
created: 2026-10-15 13:48
updated: 2026-10-15 13:48
type: note
status: seed
tags:
  - rust
aliases: []
topic: "[[Rust]]"
---

## Summary


## Notes
In **Chapter 8: Common Collections**, *The Rust Programming Language* examines three essential data structures provided by the standard library: **Vectors**, **Strings**, and **Hash Maps**. Unlike primitive arrays or tuples whose sizes are fixed at compile time, these collections store their contents on the heap, allowing them to dynamically grow or shrink as your program executes.

---

### **1. Storing Lists of Values with Vectors (`Vec<T>`)**

A **vector** (`Vec<T>`) places multiple values of the same type next to each other in heap memory.

#### **A. Creating and Updating Vectors**
Vectors can be initialized empty using `Vec::new()` (requiring a type annotation if no elements are added immediately) or populated using the **`vec!`** macro. Adding items requires declaring the vector as **`mut`** and using the **`.push()`** method.

```rust
// Explicit type annotation for an empty vector
let v: Vec<i32> = Vec::new();

// Using the vec! macro with type inference
let v = vec![1,2,3];

// Modifying a vector
let mut v = Vec::new();
v.push(5);
v.push(6);
```
*(Source: Listings 8-1, 8-2, 8-3)*

When a vector goes out of scope, it is dropped along with all of its elements.

#### **B. Reading Elements & Borrow Checker Protections**
Rust offers two ways to access vector elements: **indexing syntax** (`&v[i]`) and the **`.get(i)`** method.

```rust
let v = vec![1,2,3];

// Method 1: Indexing syntax (returns &T, panics if index is out of bounds)
let third: &i32 = &v[3];

// Method 2: .get() method (returns Option<&T>, handles out-of-bounds gracefully)
let third: Option<&i32> = v.get(2);
```
*(Source: Listing 8-5)*

* **Panic vs. `None`**: Indexing past the vector's length causes the program to panic, whereas `.get()` returns `None` without crashing.
* **Reallocation Safety**: The borrow checker prevents holding a reference to a vector element while attempting to push new items to the vector. Pushing an element may force the vector to allocate a larger memory block on the heap and copy existing elements over, which would invalidate any existing references.

```rust
let mut v = vec![1,2,3,4,5];
let first = &v; // Immutable borrow
v.push(6);         // Compile error! Cannot borrow `v` as mutable while immutably borrowed.
```
*(Source: Listing 8-7)*

#### **C. Iterating Over Vectors**
Iterating over elements can be done immutably or mutably. When mutating items during iteration, the **dereference operator (`*`)** must be used to mutate the underlying value.

```rust
let mut v = vec!;

// Mutable iteration using the dereference operator
for i in &mut v {
    *i += 50;
}
```
*(Source: Listing 8-9)*

#### **D. Storing Multiple Types with Enums**
To store elements of different types inside a single vector, you can define an **enum** whose variants hold the desired data types. Because enum variants belong to the same parent enum type, the vector remains homogeneous at the type level while accommodating diverse data.

```rust
enum SpreadsheetCell {
    Int(i32),
    Float(f64),
    Text(String),
}

let row = vec![
    SpreadsheetCell::Int(3),
    SpreadsheetCell::Text(String::from("blue")),
    SpreadsheetCell::Float(10.12),
];
```
*(Source: Listing 8-10)*

---

### **2. Storing UTF-8 Encoded Text with Strings**

String handling in Rust exposes the inherent complexities of text encodings to guarantee correctness at compile time.

#### **A. What is a String?**
Rust has only one string type in the core language: the **string slice (`str` or `&str`)**, which is a borrowed reference to UTF-8 text data. The **`String`** type is provided by the standard library as a growable, mutable, owned, UTF-8 encoded string buffer. Additional specialized string types exist in the standard library, such as `OsString`/`OsStr` and `CString`/`CStr`.

#### **B. Creating and Updating Strings**
`String` instances can be created using `String::new()`, `.to_string()`, or `String::from()`. Because strings are UTF-8 encoded, any validly encoded text in any human language can be stored.

```rust
let s1 = String::from("Hello");
let s2 = "initial contents".to_string();

let mut s = String::from("foo");
s.push_str("bar"); // Appends a string slice (&str) without taking ownership
s.push('!');       // Appends a single char
```
*(Source: Listings 8-11, 8-12, 8-15, 8-17)*

#### **C. Concatenation: The `+` Operator vs. `format!` Macro**
Combining two `String` values with the **`+` operator** calls an `add` function with the signature `fn add(self, s: &str) -> String`.

```rust
let s1 = String::from("Hello, ");
let s2 = String::from("world!");
let s3 = s1 + &s2; // s1 is MOVED and can no longer be used!
```
*(Source: Listing 8-18)*

* **Deref Coercion**: The second argument `&s2` (type `&String`) is automatically coerced into `&str` by the compiler via **deref coercion** (`&s2[..]`).
* **Ownership Transfer**: The `+` operator takes ownership of `s1`, appends `s2`'s contents, and returns the result without creating extra heap allocations.
* **Formatting Macro**: For concatenating multiple strings without transferring ownership, the **`format!`** macro provides cleaner code.

```rust
let s1 = String::from("tic");
let s2 = String::from("tac");
let s3 = String::from("toe");

let s = format!("{}-{}-{}", s1, s2, s3); // Does not take ownership of s1, s2, or s3
```
*(Source: String Concatenation)*

#### **D. Why String Indexing (`s`) Is Forbidden**
Attempting to access individual characters in a `String` via numerical index (`s[1]`) results in a **compile-time error**. This restriction exists for three reasons:

1. **Internal Representation**: A `String` is a wrapper over a **`Vec<u8>`**. A single Unicode scalar value can take between 1 and 4 bytes in UTF-8. Returning a single byte at index `0` would often yield an invalid fragment of a character.
2. **Three Views of Text**: Rust interprets text in three valid ways—as **raw bytes** (`u8`), **Unicode scalar values** (`char`), and **grapheme clusters** (what humans consider letters).
3. **No O(1) Performance Guarantee**: String indexing cannot guarantee constant-time execution because Rust would have to traverse the string from the beginning to find valid character boundaries.

#### **E. Slicing Strings and Iteration Methods**
To slice a string, you must specify a range of **byte boundaries**. If a range cuts through the middle of a multi-byte UTF-8 character, Rust panics at runtime.

```rust
let hello = "Здравствуйте";
let s = &hello[0..4]; // Creates a &str containing "Зд" (each character takes 2 bytes)
```
*(Source: String Slices)*

To iterate through string content explicitly, use **`.chars()`** or **`.bytes()`**:

```rust
// Iterating over Unicode scalar values (chars)
for c in "नमस्ते".chars() {
    println!("{}", c); // Prints 6 char items
}

// Iterating over raw bytes
for b in "नमस्ते".bytes() {
    println!("{}", b); // Prints 18 u8 byte values
}
```
*(Source: Methods for Iterating over Strings)*

---

### **3. Storing Keys with Associated Values in Hash Maps (`HashMap<K, V>`)**

The type **`HashMap<K, V>`** stores key-value pairs using a hashing function to map keys of type `K` to values of type `V`.

#### **A. Importing and Creation**
Because `HashMap` is the least frequently used of the three collections, it is **not included in Rust's prelude** and must be brought into scope explicitly with `use std::collections::HashMap;`.

```rust
use std::collections::HashMap;

let mut scores = HashMap::new();
scores.insert(String::from("Blue"), 10);
scores.insert(String::from("Yellow"), 50);
```
*(Source: Listing 8-20)*

Like vectors, hash maps store their data on the heap and are **homogeneous**: all keys must share the same type, and all values must share the same type.

#### **B. Constructing via `collect`**
A hash map can also be created by zipping two vectors into a vector of tuples and calling **`.collect()`**:

```rust
let teams = vec![String::from("Blue"), String::from("Yellow")];
let initial_scores = vec![10,50];

// HashMap<_, _> annotation directs collect() to gather tuples into a HashMap
let scores: HashMap<_, _> = teams.iter().zip(initial_scores.iter()).collect();
```
*(Source: Listing 8-21)*

#### **C. Hash Maps and Ownership**
Types that implement the `Copy` trait (like `i32`) are copied into the hash map. Owned values (like `String`) are **moved into the map**, transferring ownership so that the map becomes their owner.

```rust
let field_name = String::from("Favorite color");
let field_value = String::from("Blue");

let mut map = HashMap::new();
map.insert(field_name, field_value);
// field_name and field_value are moved and no longer valid here!
```
*(Source: Listing 8-22)*

#### **D. Accessing Values & Iteration**
Calling **`.get(&key)`** returns an **`Option<&V>`**. If the key does not exist, it returns `None`.

```rust
let team_name = String::from("Blue");
let score: Option<&i32> = scores.get(&team_name);

// Iterating over key-value pairs
for (key, value) in &scores {
    println!("{}: {}", key, value);
}
```
*(Source: Listing 8-23 & Accessing Values)*

#### **E. Updating Strategies**

1. **Overwriting Existing Values**: Calling `.insert()` with an existing key overwrites its associated value.
2. **Inserting Only If Key Is Absent (`entry` API)**: The **`.entry()`** method returns an `Entry` enum. Calling **`.or_insert()`** on the entry inserts the default value only if the key does not already exist, and returns a mutable reference (`&mut V`) to the value.

```rust
let mut scores = HashMap::new();
scores.insert(String::from("Blue"), 10);

// "Yellow" gets inserted with 50; "Blue" is ignored because it exists
scores.entry(String::from("Yellow")).or_insert(50);
scores.entry(String::from("Blue")).or_insert(50);
```
*(Source: Listing 8-25)*

3. **Updating a Value Based on Old Value**: Calling `.or_insert()` returns a mutable reference (`&mut V`), which can be dereferenced (`*`) to modify the stored value in place.

```rust
let text = "hello world wonderful world";
let mut map = HashMap::new();

for word in text.split_whitespace() {
    let count = map.entry(word).or_insert(0);
    *count += 1; // Dereference mutable reference to increment value
}
// Outputs: {"world": 2, "hello": 1, "wonderful": 1}
```
*(Source: Listing 8-26)*

#### **F. Hashing Functions & `DoS` Resistance**
By default, `HashMap` uses a cryptographically secure hashing algorithm (**`SipHash`**) designed to mitigate Denial of Service (`DoS`) attacks involving hash collisions. If higher speed is required, you can replace the default hasher by providing a custom type that implements the **`BuildHasher`** trait.

---
🦀 

## References
