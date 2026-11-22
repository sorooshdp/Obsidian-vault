---
created: 2026-11-10 13:54
updated: 2026-11-10 13:54
type: moc
status: mature
tags:
  - js
aliases:
  - JavaScript MOC
  - YDKJS
---

# JavaScript & YDKJS Hub

## 1. Types, Values & Grammar
- [[YDKJS intro to js]] — Overview of typed values, truthy/falsy, polyfills, and transpiling
- [[YDKJS Types]] — The 7 built-in types, `typeof` quirks, `undefined` vs. undeclared
- [[YDKJS Values]] — Arrays, strings, `NaN`, `-0`, and value-copy vs. reference
- [[YDKJS Natives]] — Boxing wrappers (`String()`, `Boolean()`), unboxing, and object wrapper gotchas
- [[YDKJS Coercion]] — Abstract operations (`ToString`, `ToNumber`, `ToBoolean`), `==` vs. `===`
- [[YDKJS Grammar]] — Statements vs. expressions, operator precedence, ASI, and TDZ
- [[nullish coalescing operator]] — Fallback handling with `??` for `null` and `undefined`

## 2. Scope, Closures & Modules
- [[YDKJS scope & closures]] — Compiler theory, Engine/Scope interaction, and nested scopes
- [[LHS and RHS Lookups]] — Assignment targets vs. value retrieval and `ReferenceError` mechanics
- [[YDKJS lexical scope]] — Write-time scope resolution, shadowing, `eval`, and `with`
- [[Dynamic scope]] — Runtime call-stack scope resolution vs. lexical scope
- [[YDKJS Function vs. Block scope]] — IIFEs, `try/catch`, `let`, and `const`
- [[YDKJS Hoisting]] — Compilation vs. execution phases, function vs. variable hoisting
- [[YDKJS Closures]] — Lexical environment retention, loop pitfalls, and the module pattern
- [[YDKJS Modules]] — Classic closure-based module characteristics
- [[JavaScript Modules]] — CommonJS (`require`) vs. ES Modules (`import`/`export`) and tree-shaking

## 3. `this`, Objects & Prototypes
- [[YDKJS this]] — Call-site inspection, the 4 binding rules, DMZ objects, and lexical arrow `this`
- [[YDKJS Objects]] — Property descriptors, shallow immutability (`freeze`/`seal`), `[[Get]]` & `[[Put]]`
- [[Deep Copy vs. Shallow Copy]] — Reference sharing vs. recursive cloning in JS
- [[YDKJS Class Objects]] — Classical inheritance vs. JS mixins and parasitic inheritance
- [[YDKJS Prototypes]] — `[[Prototype]]` chain, shadowing traps, `.constructor`, and `Object.create()`

## 4. Runtime, Memory & Utilities
- [[Memory Management in JS]] — Memory lifecycle, reference counting, and mark-and-sweep GC
- [[Data structures aiding memory management in JS]] — `WeakMap`, `WeakSet`, and ephemerons
- [[YDKJS Mixed Environment JavaScript]] — Global namespace sharing across `<script>` tags and host environments
- [[Regex Cheat Sheet]] — JS/PCRE regular expressions, lookarounds, flags, and common recipes