2024/07/29  -  10:51

Tags: [[scope]]  [[js]]

dynamic scope doesn't concern itself by how and where functions are scoops are declared, but rather where they are called from. in other words, the scope chain is based on the call-stack, not the nesting of scopes in code.

```js
function foo() {
	console.log( a ); // 3
}

function bar() {
	var a = 3;
	foo();
}

var a = 2;

bar();
```
in dynamic scope we walk up the call-stack, to find where `foo()` was called from. Since `foo()` was called from `bar()` , it checks the variables in scope for `bar()` , and finds an a there with value `3` .

**lexical scope is write-time, whereas dynamic scope (and `this` !) are runtime**

# References
