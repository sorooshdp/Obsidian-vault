---
created: 2026-10-17 18:21
updated: 2026-10-17 18:21
type: note
status: seed
tags:
  - rust
aliases: []
topic: "[[Rust]]"
---

## Summary


## Notes

In **Chapter 16: Fearless Concurrency**, *The Rust Programming Language* explores how Rust's ownership and type systems shift concurrent and parallel programming bugs from difficult-to-debug runtime issues into immediate compile-time errors. By leveraging the compiler, Rust allows developers to write multithreaded code fearlessly without risking data races or invalid memory access.

---

### **1. Using Threads to Run Code Simultaneously**

An operating system runs program code in a process, and within a program, independent tasks can run concurrently in **threads**.

#### **A. The 1:1 Threading Model**
Programming languages implement threads in two main ways:
* **1:1 Model (OS Threads)**: One operating system thread corresponds to one language thread.
* **M:N Model (Green Threads)**: \\(M\\) green threads managed by a language runtime are mapped onto \\(N\\) OS threads.

Rust's standard library uses the **1:1 model** to avoid requiring a large language runtime, prioritizing minimal performance overhead.

---

#### **B. Creating Threads with `thread::spawn`**
To spin up a new thread, call **`thread::spawn`** and pass a closure containing the code to execute in parallel. When the main thread terminates, spawned threads stop running regardless of whether they have finished.

```rust
use std::thread;
use std::time::Duration;

fn main() {
    thread::spawn(|| {
        for i in 1..10 {
            println!("hi number {} from the spawned thread!", i);
            thread::sleep(Duration::from_millis(1));
        }
    });

    for i in 1..5 {
        println!("hi number {} from the main thread!", i);
        thread::sleep(Duration::from_millis(1));
    }
}
```
*(Source: Listing 16-1)*

---

#### **C. Waiting for Threads to Finish with `JoinHandle`**
`thread::spawn` returns an owned **`JoinHandle`**. Calling **`.join()`** on the handle blocks the currently executing thread until the spawned thread completes.

```rust
use std::thread;
use std::time::Duration;

fn main() {
    let handle = thread::spawn(|| {
        for i in 1..10 {
            println!("hi number {} from the spawned thread!", i);
            thread::sleep(Duration::from_millis(1));
        }
    });

    for i in 1..5 {
        println!("hi number {} from the main thread!", i);
        thread::sleep(Duration::from_millis(1));
    }

    handle.join().unwrap(); // Main thread blocks until spawned thread finishes
}
```
*(Source: Listing 16-2)*

---

#### **D. Transferring Ownership with `move` Closures**
Because the compiler cannot predict how long a spawned thread will run, it disallows spawned threads from borrowing variables from the environment by default. Annotating the closure with the **`move`** keyword forces the closure to take ownership of captured variables.

```rust
use std::thread;

fn main() {
    let v = vec!;

    // `move` forces ownership of `v` to transfer to the thread
    let handle = thread::spawn(move || {
        println!("Here's a vector: {:?}", v);
    });

    handle.join().unwrap();
}
```
*(Source: Listing 16-5)*

---

### **2. Message-Passing Concurrency with Channels (`mpsc`)**

Message passing follows the Go language philosophy: *"Do not communicate by sharing memory; instead, share memory by communicating."*.

Rust provides channels via **`std::sync::mpsc`**, which stands for **Multiple Producer, Single Consumer**. A channel can have multiple sending ends (transmitters), but only one receiving end (receiver).

#### **A. Sending and Receiving Data**
Calling `mpsc::channel()` returns a tuple `(tx, rx)` containing the transmitter (`tx`) and receiver (`rx`).

* **`tx.send(val)`**: Moves ownership of `val` down the channel. Attempting to use `val` in the sending thread after calling `send` causes a compile error, preventing data races.
* **`rx.recv()`**: Blocks the current thread until a value arrives.
* **`rx.try_recv()`**: Non-blocking call returning immediately with `Ok` or `Err`.
* **Iterator Interface**: Treating `rx` as an iterator inside a `for` loop automatically receives values until the channel closes.

```rust
use std::thread;
use std::sync::mpsc;
use std::time::Duration;

fn main() {
    let (tx, rx) = mpsc::channel();

    thread::spawn(move || {
        let vals = vec![
            String::from("hi"),
            String::from("from"),
            String::from("the"),
            String::from("thread"),
        ];

        for val in vals {
            tx.send(val).unwrap(); // Transfers ownership of each string
            thread::sleep(Duration::from_secs(1));
        }
    });

    for received in rx { // Receives messages continuously until channel closes
        println!("Got: {}", received);
    }
}
```
*(Source: Listing 16-10)*

---

#### **B. Creating Multiple Producers (`mpsc::Sender::clone`)**
To allow multiple spawned threads to send messages to the same receiver, clone the transmitter using **`mpsc::Sender::clone(&tx)`**.

```rust
use std::thread;
use std::sync::mpsc;
use std::time::Duration;

fn main() {
    let (tx, rx) = mpsc::channel();

    // Clone the transmitter for the first thread
    let tx1 = mpsc::Sender::clone(&tx);
    thread::spawn(move || {
        let vals = vec![String::from("hi"), String::from("from")];
        for val in vals {
            tx1.send(val).unwrap();
            thread::sleep(Duration::from_secs(1));
        }
    });

    // Pass original transmitter to second thread
    thread::spawn(move || {
        let vals = vec![String::from("more"), String::from("messages")];
        for val in vals {
            tx.send(val).unwrap();
            thread::sleep(Duration::from_secs(1));
        }
    });

    for received in rx {
        println!("Got: {}", received);
    }
}
```
*(Source: Listing 16-11)*

---

### **3. Shared-State Concurrency (`Mutex<T>` and `Arc<T>`)**

Shared-state concurrency allows multiple threads to read and modify the same memory location simultaneously.

#### **A. Mutual Exclusion with `Mutex<T>`**
A **mutex** (mutual exclusion) guards data behind a locking mechanism, allowing only one thread to access data at a time.

* **Acquiring Locks**: Calling **`.lock()`** blocks execution until the lock is acquired, returning a smart pointer named **`MutexGuard<T>`**.
* **Automatic Unlocking**: `MutexGuard<T>` implements `Deref` (to access inner data) and `Drop`. When `MutexGuard<T>` goes out of scope, the lock is automatically released.
* **Interior Mutability**: Like `RefCell<T>`, `Mutex<T>` provides interior mutability.

#### **B. Thread Safety with Atomic Reference Counting (`Arc<T>`)**
`Rc<T>` cannot be shared across threads because its reference-counting operations are not thread-safe.

To share a `Mutex<T>` across multiple threads, wrap it inside **`Arc<T>`** (Atomic Reference Counting). Atomics enforce thread-safe reference counts with a slight performance cost.

```rust
use std::sync::{Mutex, Arc};
use std::thread;

fn main() {
    // Wrap Mutex in an Arc for thread-safe shared ownership
    let counter = Arc::new(Mutex::new(0));
    let mut handles = vec![];

    for _ in 0..10 {
        let counter = Arc::clone(&counter); // Clone Arc reference handle
        let handle = thread::spawn(move || {
            let mut num = counter.lock().unwrap(); // Acquire lock
            *num += 1;                             // Mutate inner value
        });                                         // Lock automatically dropped here
        handles.push(handle);
    }

    for handle in handles {
        handle.join().unwrap();
    }

    println!("Result: {}", *counter.lock().unwrap()); // Prints: Result: 10
}
```
*(Source: Listing 16-15)*

---

### **4. Extensible Concurrency with `Send` and `Sync` Traits**

Unlike many languages, Rust's concurrency primitives are mostly defined in the standard library rather than hardcoded into language syntax. However, two fundamental concurrency concepts are built directly into the language as **marker traits** in `std::marker`:

#### **A. The `Send` Marker Trait**
* Indicates that **ownership of a type can be transferred between threads**.
* Almost all primitive types implement `Send`.
* `Rc<T>` does **not** implement `Send` because concurrent reference counting can cause data corruption.
* Any custom type composed entirely of `Send` types automatically implements `Send`.

#### **B. The `Sync` Marker Trait**
* Indicates that it is **safe for a type to be referenced from multiple threads**.
* Formally, a type `T` is `Sync` if and only if `&T` (an immutable reference to `T`) is `Send`.
* `RefCell<T>` and `Rc<T>` are **not** `Sync`. `Mutex<T>` **is** `Sync`.
* Implementing `Send` or `Sync` manually requires writing `unsafe` Rust.

---
🦀 
## References
