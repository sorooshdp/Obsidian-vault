2025/08/18  -  00:30
status: 

Tags: [[js]] [[syntax]]

A "sentence" is one	complete	formation of words	that expresses	a thought. It's comprised of one	or	more "phrases," each of	which can be connected	with	punctuation	marks or conjunction words	("and,"	"or,"	etc).	A	phrase	can	itself	be	made	up	of	smaller	phrases. Some phrases are incomplete and don't	accomplish	much	by	themselves,	while	other phrases	can stand on their	own. These	rules are	collectively	called	the	grammar	of	the English	language. And	so	it	goes	with	JavaScript	grammar.	Statements	are sentences, expressions	are phrases,	and	operators are conjunctions/punctuation.

It's a fairly little known fact that statements all have completion values (even if that value is just `undefined`) 

```js
var b;
if (true) { 
	 b = 4 + 38; 
}
```

If you typed that into your console/REPL, you'd probably see completion value of the 42 reported, since 42 is the if block, which took on the completion value of its last assignment expression statement b = 4 + 38 . In other words, the completion value of a block is like an implicit return of the last statement value in the block. 

```js
var a = 42; 
a++; // 42 
a; // 43 
++a; // 44 
a; // 44
```

When ++ is used in the prefix position as ++a , its side effect (incrementing a ) happens before the value is returned from the expression, rather than after as with a++ .

```js
// assume there's a `bar()` function defined 
{ foo: bar() }
```

A lot of developers assume that the { .. } pair is just a standalone doesn't get assigned anywhere. But it's actually entirely different. Here, object literal that { .. } is just a regular code block. It's not very idiomatic in JavaScript (much more so in other languages!) to have a standalone { .. } block like that, but it's perfectly valid JS grammar. It can be especially helpful when combined with let block-scoping declarations (see the Scope & Closures title in this series).

```js
function foo() { // `bar` labeled-block
	bar: { 
		console.log( "Hello" ); 
		break bar; 
		console.log( "never runs" ); 
	} 
	console.log( "World" ); 
}
foo(); 
// Hello 
// World
```

A label can apply to a non-loop block, but only break can reference such a non-loop label. You can do a labeled break ___ out of any labeled block, but you cannot continue ___ a non-loop label, nor can you do a non-labeled break out of a block.

It's a common misconception that JavaScript has an else if clause, because you can do:
```js
if (a) { 
	// .. 
} else if (b) { 
	// .. 
} else { 
// .. 
}
```
But there's a hidden characteristic of the JS grammar here: there is no else if . But if and else statements are allowed to omit the { } around their attached block if they only contain a single statement.

## Precedence 

`a && b || c ? c || b ? a : c && b : a`
Is that more like this: 
`a && b || (c ? c || (b ? a : c) && b : a)` 
or this?
`(a && b || c) ? (c || b) ? a : (c && b) : a`
The answer is the second one. But why?
Because && is more precedent than || , and So, the expression || is more precedent than (a && b || c) is evaluated first before the ? : . ? : it participates in. Another way this is commonly explained is that && and || "bind more tightly" than If the reverse was true, then ? : . c ? c... would bind more tightly, and it would behave (as the first choice) like a && b || (c ? c..) .

ASI (Automatic Semicolon Insertion) is when JavaScript assumes a ; in certain places in your JS program even if you didn't put one there. It's important to note that ASI will only take effect in the presence of a newline (aka line break). Semicolons are not inserted in the middle of a line.

## Using Variables Too Early 
ES6 defines a (frankly confusingly named) new concept called the TDZ ("Temporal Dead Zone"). The TDZ refers to places in code where a variable reference cannot yet be made, because it hasn't reached its required initialization.

```js
{ a = 2; // ReferenceError! 
let a;
}
```

## try {} finally

A return inside a finally has the special ability to override a previous return from the try or catch clause, but only if return is explicitly called:

The finally clause attached to a try (or try..catch ) offers some very interesting quirks in terms of execution processing order. Some of these quirks can be helpful, but it's possible to create lots of confusion, especially if combined with labeled blocks. As always, use finally to make code better and clearer, not more clever or confusing.
# References
