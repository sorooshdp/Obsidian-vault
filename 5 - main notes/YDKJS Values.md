2024/07/29  -  11:58

Tags: [[js]] [[values]] 

Warning: Using delete on an array value will remove that slot from the array , but even if you remove the final element, it does not update the length property, so be careful!

a gotcha to be aware of is that if a string value intended as a key can be coerced to a standard base-10 number , then it is assumed that you wanted to use it as a number index rather than as a string key!

```js
var a = [ ];
a["13"] = 42;
a.length; // 14
```
# Strings
JavaScript `string` s are immutable, while `array` s are quite mutable. none of the `string` methods that alter its contents can modify in-place, but rather must create and return new `string` s. By contrast, many of the methods that change `array` contents actually do modify in-place.

# The not number, number
Any mathematic operation you perform without both operands being `number` s will result in the operation failing to produce a valid number , in which case you will get the `NaN` value.

```js
if (!Number.isNaN) { 
	Number.isNaN = function(n) { 
		return ( 
			typeof n === "number" && window.isNaN( n ) 
		);
	}; 
} 
var a = 2 / "foo";
var b = "foo";
Number.isNaN( a ); // true 
Number.isNaN( b ); // false -- phew!
```

# Zeros

JavaScript has both a normal zero (otherwise known as a positive zero +0 ) and a negative zero -0. There are certain applications where developers use the magnitude of a value to represent one piece of information (like speed of movement per animation frame) and the sign of that number to represent another piece of information (like the direction of that movement). In those applications, as one example, if a variable arrives at zero and it loses its sign, then you would lose the information of what direction it was moving in before it arrived at zero. Preserving the sign of the zero prevents potentially unwanted information loss.
# Value vs. Reference
Remember: you cannot directly control/override value-copy vs. reference -- those semantics are controlled entirely by the type of the underlying value.

A reference in JS points at a (shared) value, so if you have 10 different references, they are all always distinct references to a single shared value; none of them are references/pointers to each other.
Simple values (aka scalar primitives) are always assigned/passed by value-copy: `null` , `undefined` , `string` , `number` , `boolean` , and ES6's `symbol` .
Compound values -- `object` s and `function` s -- always create a copy of the reference on assignment or passing.
# References
