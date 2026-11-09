---
created: 2026-10-15 17:01
updated: 2026-10-15 17:01
type: note
status: seed
tags:
  - rust
aliases: []
---

## Summary


## Notes

In **Chapter 12: An I/O Project: Building a Command Line Program**, *The Rust Programming Language* synthesizes concepts from earlier chapters—such as modules, collections, error handling, traits, lifetimes, and testing—by building a custom, simplified command-line utility called **`minigrep`** (a lightweight version of `grep`).

---

### **1. Accepting Command-Line Arguments**

To accept command-line arguments, Rust's standard library provides the **`std::env::args`** function, which returns an iterator over the arguments passed to the program. Calling **`.collect()`** turns this iterator into a vector of strings (`Vec<String>`).

#### **Handling Invalid Unicode**
* **`std::env::args`**: Panics if any argument contains invalid Unicode.
* **`std::env::args_os`**: Use this alternative if your application must accept non-Unicode command-line arguments. It returns an iterator producing **`OsString`** values instead of `String`.

#### **First Code Block: Collecting Arguments (`src/main.rs`)**
```rust
use std::env;

fn main() {
    let args: Vec<String> = env::args().collect();

    let query = &args;
    let filename = &args;

    println!("Searching for {}", query);
    println!("In file {}", filename);
}
```
*(Source: Listing 12-1 & Listing 12-2)*

* **Index 0 (`args`)**: Contains the path/name of the compiled executable binary (matching standard C behavior).
* **Indexes 1 & 2 (`args`, `args`)**: Store the search query string and the target file path, respectively.

---

### **2. Reading a File**

File operations require importing **`std::fs::File`** and bringing file I/O traits into scope via **`std::io::prelude::*`**.

#### **Second Code Block: Reading File Contents (`src/main.rs`)**
```rust
use std::env;
use std::fs::File;
use std::io::prelude::*;

fn main() {
    let args: Vec<String> = env::args().collect();
    let query = &args;
    let filename = &args;

    let mut f = File::open(filename).expect("file not found");

    let mut contents = String::new();
    f.read_to_string(&mut contents)
        .expect("something went wrong reading the file");

    println!("With text:\n{}", contents);
}
```
*(Source: Listing 12-4)*

---

### **3. Refactoring for Modularity & Error Handling**

Placing command-line parsing, file reading, and error handling all inside `main` creates multiple responsibilities, making testing and maintenance difficult. 

#### **Separation of Concerns for Binary Projects**
To organize command-line applications, the Rust community adheres to a standard guideline:
1. **Split the binary**: Divide the program into **`src/main.rs`** and **`src/lib.rs`**.
2. **`src/main.rs` Responsibilities**: Limited to calling argument parsing logic, setting up configuration, calling a `run` function in `lib.rs`, and handling any errors returned by `run`.
3. **`src/lib.rs` Responsibilities**: Holds all core logic, data structures, file processing, and search algorithms so they can be unit-tested directly.

---

### **4. Structuring Configuration & Returning `Result`**

#### **A. Grouping Configuration into a Struct (`Config`)**
Instead of returning individual primitive variables or tuples, configuration values should be encapsulated inside a **`Config` struct**.

#### **B. The Trade-Offs of `.clone()`**
When constructing `Config`, calling `.clone()` on the argument strings creates owned copies of the data. While slightly less performant than managing borrowed references (`&str`), cloning avoids complex lifetime management on configuration data and keeps ownership logic straightforward.

#### **C. Replacing `panic!` with `Result` & Using `unwrap_or_else`**
Calling `panic!` on bad user inputs displays noisy thread backtraces. Returning a **`Result<Config, &'static str>`** allows `main` to handle failures gracefully. The **`.unwrap_or_else()`** method executes a closure (anonymous function) to print a clean error message and terminate the process using **`std::process::exit(1)`**.

#### **Third Code Block: `Config::new` and Clean Error Handling (`src/main.rs`)**
```rust
use std::env;
use std::process;

fn main() {
    let args: Vec<String> = env::args().collect();

    let config = Config::new(&args).unwrap_or_else(|err| {
        println!("Problem parsing arguments: {}", err);
        process::exit(1);
    });

    // --snip--
}

struct Config {
    query: String,
    filename: String,
}

impl Config {
    fn new(args: &[String]) -> Result<Config, &'static str> {
        if args.len() < 3 {
            return Err("not enough arguments");
        }

        let query = args.clone();
        let filename = args.clone();

        Ok(Config { query, filename })
    }
}
```
*(Source: Listings 12-9 & 12-10)*

---

### **5. Extracting Core Logic to a Library Crate (`src/lib.rs`)**

#### **A. Trait Objects for Flexible Error Handling (`Box<Error>`)**
The core execution logic is extracted into a `run` function. Returning **`Result<(), Box<Error>>`** leverages a **trait object** (`Box<Error>`), allowing `run` to propagate different types of standard library errors (such as file I/O errors) using the `?` operator without committing to a single concrete error type.

#### **Fourth Code Block: The Modular Library (`src/lib.rs`) & Binary (`src/main.rs`)**

##### **`src/lib.rs`**
```rust
use std::error::Error;
use std::fs::File;
use std::io::prelude::*;

pub struct Config {
    pub query: String,
    pub filename: String,
}

impl Config {
    pub fn new(args: &[String]) -> Result<Config, &'static str> {
        if args.len() < 3 {
            return Err("not enough arguments");
        }

        let query = args.clone();
        let filename = args.clone();

        Ok(Config { query, filename })
    }
}

pub fn run(config: Config) -> Result<(), Box<Error>> {
    let mut f = File::open(config.filename)?;

    let mut contents = String::new();
    f.read_to_string(&mut contents)?;

    println!("With text:\n{}", contents);

    Ok(()) // Returns unit type inside Ok
}
```
*(Source: Listing 12-13)*

##### **`src/main.rs`**
```rust
extern crate minigrep;

use std::env;
use std::process;
use minigrep::Config;

fn main() {
    let args: Vec<String> = env::args().collect();

    let config = Config::new(&args).unwrap_or_else(|err| {
        println!("Problem parsing arguments: {}", err);
        process::exit(1);
    });

    if let Err(e) = minigrep::run(config) {
        println!("Application error: {}", e);
        process::exit(1);
    }
}
```
*(Source: Listing 12-14)*

---

### **6. Developing Search Logic with Test-Driven Development (TDD)**

The project uses **Test-Driven Development (TDD)** to implement the string searching logic:
1. Write a failing unit test.
2. Implement just enough code to compile and pass the test.
3. Refactor while maintaining passing tests.

#### **Explicit Lifetime Annotations (`'a`)**
The `search` function returns a vector of string slices (`Vec<&'a str>`) pointing to lines inside `contents`. Rust requires an explicit lifetime annotation **`'a`** connecting the `contents` argument to the returned slices, ensuring that returned references cannot outlive the text buffer they reference.

#### **Fifth Code Block: Search Function and Unit Test (`src/lib.rs`)**
```rust
pub fn search<'a>(query: &str, contents: &'a str) -> Vec<&'a str> {
    let mut results = Vec::new();

    for line in contents.lines() { // Iterates line-by-line
        if line.contains(query) {  // Checks string containment
            results.push(line);
        }
    }

    results
}

#[cfg(test)]
mod test {
    use super::*;

    #[test]
    fn one_result() {
        let query = "duct";
        let contents = "\
Rust:
safe, fast, productive.
Pick three.";

        assert_eq!(
            vec!["safe, fast, productive."],
            search(query, contents)
        );
    }
}
```
*(Source: Listings 12-15, 12-19)*

---

### **7. Working with Environment Variables**

To allow users to enable case-insensitive searching across terminal sessions, `minigrep` checks for an environment variable named `CASE_INSENSITIVE`.

* **`std::env::var("CASE_INSENSITIVE")`**: Returns `Ok(value)` if set, or `Err` if unset.
* **`env::var(...).is_err()`**: Evaluates to `true` if the variable is unset, defaulting to case-sensitive search.

#### **Sixth Code Block: Case-Insensitive Search & Environment Checks (`src/lib.rs`)**
```rust
use std::env;

pub struct Config {
    pub query: String,
    pub filename: String,
    pub case_sensitive: bool,
}

impl Config {
    pub fn new(args: &[String]) -> Result<Config, &'static str> {
        if args.len() < 3 {
            return Err("not enough arguments");
        }

        let query = args.clone();
        let filename = args.clone();

        // Checks if CASE_INSENSITIVE environment variable is absent
        let case_sensitive = env::var("CASE_INSENSITIVE").is_err();

        Ok(Config { query, filename, case_sensitive })
    }
}

pub fn search_case_insensitive<'a>(query: &str, contents: &'a str) -> Vec<&'a str> {
    let query = query.to_lowercase(); // Allocates a lowercase String
    let mut results = Vec::new();

    for line in contents.lines() {
        if line.to_lowercase().contains(&query) {
            results.push(line);
        }
    }

    results
}
```
*(Source: Listings 12-21 & 12-23)*

---

### **8. Writing Errors to Standard Error (`eprintln!`)**

Terminals distinguish between **standard output (`stdout`)** for regular output and **standard error (`stderr`)** for diagnostics. 

* Using `println!` sends error messages to `stdout`. Redirecting program output to a file (`cargo run > output.txt`) would accidentally capture error text in the output file.
* **`eprintln!`**: Prints directly to standard error (`stderr`), ensuring error messages appear on-screen even when standard output is redirected.

#### **Seventh Code Block: Redirecting Errors to Stderr (`src/main.rs`)**
```rust
fn main() {
    let args: Vec<String> = env::args().collect();

    let config = Config::new(&args).unwrap_or_else(|err| {
        eprintln!("Problem parsing arguments: {}", err); // Writes to stderr
        process::exit(1);
    });

    if let Err(e) = minigrep::run(config) {
        eprintln!("Application error: {}", e);           // Writes to stderr
        process::exit(1);
    }
}
```
*(Source: Listing 12-24)*

---
🦀 
## References
