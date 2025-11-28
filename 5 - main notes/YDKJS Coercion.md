2024/08/08  -  02:09

Tags: [[js]] [[coercion]] [[types]] 

## coercion 
Converting a value from one type to another is often called "type casting," when done explicitly, and "coercion" when done implicitly. JavaScript coercions always result in one of the scalar primitive values, like `string` , `number` , or `boolean` . There is no coercion that results in a complex value like `object` or `function`. 

Another way these terms are often distinguished is as follows: "type casting" (or "type conversion") occur in statically typed languages at compile time, while "type coercion" is a runtime conversion for dynamically typed languages.

## `ToString` 
 When any non- string value is coerced to a string representation, the conversion is handled by the `ToString` abstract operation in section 9.8 of the specification. 
 
 `function` s, (ES6+) `symbol` s, and `undefined` s, `object` s with circular references (where property references in an object structure create a never-ending cycle through each other). These are all illegal values for a standard JSON structure, mostly because they aren't portable to other languages that consume JSON values.

## JSON Stringification

Another task that seems awfully related to `ToString` is when you use the `JSON.stringify(..)` utility to serialize a value to a JSON-compatible string value. The `JSON.stringify(..)` utility will automatically omit undefined , function , and symbol values when it comes across them. undefined s, function s, (ES6+) symbol s, and object s with circular references (where property references in an object structure create a never-ending cycle through each other) are not considered JSON-safe values. 

```js
JSON.stringify( undefined ); // undefined 
JSON.stringify( function(){} ); // undefined 
JSON.stringify( [1,undefined,function(){},4] ); // "[1,null,null,4]"
JSON.stringify( { a:2, b:function(){} } ); // "{"a":2}"
```
It's a very common misconception that `toJSON()` should return a JSON stringification representation. `toJSON()` should be interpreted as "to a JSON-safe value suitable for stringification," not "to a JSON string" as many developers mistakenly assume.

## `ToNumber` 
If any non- number value is used in a way that requires it to be a number , such as a mathematical operation, the ES5 spec defines the `ToNumber` abstract operation in section 9.3. For example, `true` becomes `1` and `false` becomes `0` . undefined becomes `NaN` , but (curiously) `null` becomes `0`.
Objects will first be converted to their **primitive values** using the `ToPrimitive` operation. JavaScript will try to call `valueOf()`, and if that returns a primitive, it will be used. If not, it will try `toString()`. If the string contains valid numeric characters, it is converted to a number, otherwise, it becomes `NaN`.

## `ToBoolean`

anything not explicitly on the `falsy` list is therefore truthy. a value is truthy if it's not on the `falsy` list.
- undefined 
- null 
- false 
- +0 
- -0 
- `NaN`
- ""


## `Falsy` objects

A "`falsy` object" is a value that looks and acts like a normal object (properties, etc.), but when you coerce it to a `boolean` , it coerces to a false value.
Be aware of edge cases like `document.all`, which is considered a **`falsy` object** even though it’s technically an object.

## truthy objects

```js
var a = []; // empty array -- truthy or falsy?
var b = {}; // empty object -- truthy or falsy? 
var c = function(){}; // empty function -- truthy or falsy? 
var d = Boolean( a && b && c ); 

d; //true
```

## Explicitly: Strings <--> Numbers

```js
var a = 42;
var b = a.toString();
var c = "3.14";
var d = +c;
b; // "42"
d; // 3.14
```

Calling `a.toString()` is ostensibly explicit (pretty clear that "`toString`" means "to a string"), but there's some hidden implicitness here. like `toString()` cannot be called on a primitive value 42 . So JS automatically "boxes" (see Chapter 3) 42 in an object wrapper, so that `toString()` can be called against the object. In other words, you might call it "explicitly implicit." +c here is showing the unary operator form (operator with only one operand) of the + operator. Instead of performing mathematic addition (or string concatenation -- see below), the unary + explicitly coerces its operand ( c ) to a number value.

## ~ 

So, let's turn our attention back to ~ . The ~ operator first "coerces" to a 32-bit value, and then performs a bitwise negation (flipping each bit's parity). Note: This is very similar to how ! not only coerces its value to parity (see discussion of the "unary ! " later).
How ~~ works is that the first ~ applies the ToInt32 "coercion" and does the bitwise flip, and then the second ~ does another bitwise flip, flipping all the bits back to the original state. The end result is just the ToInt32 "coercion" (aka truncation).

## Implicit coercion 

Implicit coercion refers to type conversions that are hidden, with non-obvious side-effects that implicitly occur from other actions. In other words, implicit coercions are any type conversions that aren't obvious (to you).

Many developers believe that if a mechanism can do some useful thing A but can also be abused or misused to do some awful thing Z, then we should throw out that mechanism altogether, just to be safe.

The value produced by a && or || operator is not necessarily of type Boolean. The value produced will always be the value of one of the two operand expressions.

```js
var a = 42;
var b = "abc";
var c = null;
a || b; // 42 
a && b; // "abc"
c || b; // "abc"
c && b; // null
```

Both || and && operators perform a boolean test on the first operand ( a or c ). If the operand is not already boolean (as it's not, here), a normal ToBoolean coercion occurs, so that the test can be performed. For the || operator, if the test is true , the || expression results in the value of the first operand ( a or c ). If the test is false , the || expression results in the value of the second operand ( b ). Inversely, for the && operator, if the test is true , the && expression results in the value of the second operand ( b ). If the test is false , the && expression results in the value of the first operand ( a or c ).

```js
a || b; // roughly equivalent to: a ? a : b;
a && b; // roughly equivalent to: a ? b : a;
```

This || idiom is extremely common, and quite helpful, but you have to use it only in cases where all falsy values should be skipped. Otherwise, you'll need to be more explicit in your test, and probably use a ? : ternary instead.

## Symbol Coercion

symbol values cannot coerce to number at all (throws an error either way), but strangely they can both explicitly and implicitly coerce to `boolean` (always true ).

```js
var s1 = Symbol( "cool" );
String( s1 ); // "Symbol(cool)"
var s2 = Symbol( "not cool" );
s2 + "";  // TypeError

```
## Loose Equals vs. Strict Equals
A very common misconception about these two operators is: " == checks values for equality and === checks both values and types for equality." While that sounds nice and reasonable, it's inaccurate. Countless well-respected JavaScript books and blogs have said exactly that, but unfortunately they're all wrong. The correct description is: " == allows coercion in the equality comparison and disallows coercion."

In the first explanation, it seems obvious that === is doing more work than == , because it has to also check the type. In the second explanation, == is the one doing more work because it has to follow through the steps of coercion if the types are different. Don't fall into the trap, as many have, of thinking this has anything to do with performance, though, as if == is going to be slower than === in any relevant way. While it's measurable that coercion does take a little bit of processing time, it's mere microseconds (yes, that's millionths of a second!). If you're comparing two values of different types, the performance isn't the important factor. What you should be asking yourself is: when comparing these two values, do I want coercion or not? If you want coercion, use equality.

It's a very little known fact that == and 42 === behave identically in the case where two object s are being compared!

```js
var a = 42;
var b = "42";
a === b; // false
a == b; // true
```
1. If `Type(x)` is Number and `Type(y)` is String, return the result of the comparison x == `ToNumber(y)`. 
2. If `Type(x)` is String and `Type(y)` is Number, return the result of the comparison `ToNumber(x) == y`.

## Comparing: anything to `boolean`

```js
var a = "42"; 
var b = true; 
a == b; // false
```

Let's again quote the spec, clauses 11.9.3.6-7: 
1. If `Type(x)` is Boolean, return the result of the comparison `ToNumber(x) == y`. 
2. If `Type(y)` is Boolean, return the result of the comparison `x == ToNumber(y)`.
The Type(x) is indeed Boolean , so it performs Now, ToNumber(x) , which coerces true to 1 . 1 == "42" is evaluated. The types are still different, so (essentially recursively) we reconsult the algorithm, which just as above will coerce "42" to 42 , and 1 == 42 is clearly false.

"42" is indeed truthy, but "42" == true is not performing a `boolean` test/coercion at all, no matter what your brain says. "42" is not being coerced to a `boolean` ( true ), but instead true is being coerced to a 1 , and then "42" is being coerced to 42.

```js
var a = "42";

// bad (will fail!): 
if (a == true) { // .. 
} 

// also bad (will fail!):
if (a === true) { // .. 
} 

// good enough (works implicitly): 
if (a) { // .. 
} 

// better (works explicitly):
if (!!a) { // .. 
} 

// also great (works explicitly):
if (Boolean( a )) { // .. 
}
```

## Comparing: `object` s to `non- object` s

If an object / function / array is compared to a simple scalar primitive ( string , number , or `boolean` ), the ES5 spec says in clauses 11.9.3.8-9: Coercion 238 
1. If Type(x) is either String or Number and Type(y) is Object, return the result of the comparison x == `ToPrimitive(y)`. 
2. If Type(x) is Object and Type(y) is either String or Number, return the result of the comparison `ToPrimitive(x) == y`.

```js
"0" == null; // false 
"0" == undefined; // false
"0" == false; // true -- UH OH!
"0" == NaN; // false 
"0" == 0; // true
"0" == ""; // false
false == null; // false 
false == undefined; // false 
false == NaN; // false 
false == 0; // true -- UH OH!
false == ""; // true -- UH OH!
false == []; // true -- UH OH!
false == {}; // false
"" == null; // false 
"" == undefined; // false
"" == NaN; // false 
"" == 0; // true -- UH OH! 
"" == []; // true -- UH OH!
"" == {}; // false 
0 == null; // false 
0 == undefined; // false 
0 == NaN; // false 
0 == []; // true -- UH OH!
0 == {}; // false
```

![[JavaScript-Equality-Table.png]]

## Abstract Relational Comparison 

The algorithm is only defined for a < b . So, a > b is handled as b < a .

The algorithm first calls `ToPrimitive` coercion on both values, and if the return result of either call is not a string , then both values are coerced to number values using the `ToNumber` operation rules, and compared numerically.
# References
