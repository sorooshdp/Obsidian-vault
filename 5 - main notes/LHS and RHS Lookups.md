2025/04/07  -  18:08

Tags: [[js]] [[scope]]  

When the JavaScript Engine executes code, it needs to find variables. This process involves different types of lookups: LHS (Left-hand Side) and RHS (Right-hand Side). These terms originate from the sides of an assignment operation.

• LHS Lookup: An LHS lookup occurs when a variable is the target of an assignment. The Engine's goal in an LHS lookup is to find the variable container itself so that a value can be assigned to it. Examples include `a = 2` (looking for the container `a`) and the implicit assignment to function parameters like a in `function foo(a) { ... } foo(2)`. Even `var b = a;` involves an LHS lookup for `b`.

• RHS Lookup: An RHS lookup happens when a variable appears on the right-hand side of an assignment or simply when the value of a variable is being retrieved. The Engine's aim here is to get the value of the variable. Examples include `console.log(a)` (retrieving the value of a) and `var b = a;` (retrieving the value of a to assign to `b`). The function call `foo(2)` also requires an RHS lookup for foo to retrieve its function value for execution.

It's important to note that LHS and RHS don't always literally mean the left or right of the = operator; conceptually, LHS is about finding the assignment target, and RHS is about getting the source value.

## Interaction with Scope

LHS and RHS lookups are crucial for how JavaScript resolves variable references through scope. Scope is the set of rules that determines where and how a variable can be looked up. Scopes are often nested, forming "bubbles" where identifiers (variables, functions) are declared.

When the Engine encounters an LHS or RHS reference, it starts the lookup in the currently executing scope. If the variable is not found in the current scope, the Engine moves up to the next outer (enclosing) scope, and this process continues up the scope chain until the variable is found or the global scope is reached. Scope lookup stops as soon as the first match is found.

## Errors Resulting from Lookups

The outcome of an LHS or RHS lookup depends on whether the variable is found in the scope chain.

• Failed RHS Lookup: If an RHS lookup fails to find a variable in any scope up to the global scope, the Engine throws a `ReferenceError`.

• Failed LHS Lookup (Non-Strict Mode): In non-"Strict Mode," if an LHS lookup for a previously undeclared variable reaches the global scope without finding it, the Engine automatically creates a new global variable with that name.

• Failed LHS Lookup (Strict Mode): In "Strict Mode," a failed LHS lookup for an undeclared variable will also result in a `ReferenceError`.

## Nested Scope and Lookups

Nested scopes mean that inner scopes have access to variables declared in their outer scopes due to the scope chain. During the execution of a function within another function, RHS lookups for variables not found in the inner function's scope will continue to search in the outer function's scope, and so on, until the global scope. This process of traversing nested scopes is fundamental to how JavaScript resolves variable references.

## LHS/RHS Lookups and Closures

The concept of closure is deeply intertwined with lexical scope and thus with LHS and RHS lookups. Closure occurs when a function remembers and retains access to its lexical scope (the scope in which it was declared) even when that function is executed outside of that original scope.

Consider a scenario where an inner function declared within an outer function accesses a variable from the outer function's scope. Even if the inner function is later executed in a different scope (e.g., passed as a callback or returned from the outer function), it will still have access to the variable from its original lexical scope via the scope chain and the necessary RHS lookups.

The power of closures lies in this ability of the inner function to "close over" its surrounding lexical environment, remembering the variables within it. When the inner function is executed, it uses LHS and RHS lookups to access these captured variables, even if the outer function has already finished executing. The module pattern, a common JavaScript idiom for creating encapsulation and data privacy, heavily relies on closures, where inner functions retain access to private variables through these lexical scope lookups.

In summary, LHS and RHS lookups are the fundamental mechanisms by which the JavaScript Engine resolves variable references. They operate within the rules of scope, traversing the scope chain to find variables. Closures, a powerful feature of JavaScript, depend on these lexical scoping rules and lookups to enable functions to maintain access to variables from their declaration-time scope, even when executed elsewhere.
# References
