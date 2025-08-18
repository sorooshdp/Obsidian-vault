2024/08/02  -  19:36

Tags: [[js]] [[wrappers]]

The result of the constructor form of value creation `new String("abc")` is an object wrapper around the primitive `( "abc" )` value.
### Natives
In JavaScript, several built-in functions, commonly referred to as "natives," provide essential functionality for the language. These include `String()`, `Number()`, `Boolean()`, `Array()`, `Object()`, `Function()`, `RegExp()`, `Date()`, `Error()`, and `Symbol()`. Each of these natives can be used as constructors, but they often create objects rather than simple primitives.
# Boxing wrappers
Primitive values don't have properties or methods, so to access `.length` or `.toString()` you need an object wrapper around the value. Thankfully, JS will automatically box (aka wrap) the primitive value to fulfill such accesses.
```js
var a = "abc"; console.log(a.length); // 3 
console.log(a.toUpperCase()); // "ABC"
```
***never ever, under any circumstances***, should you intentionally create and use these exotic empty-slot arrays. Just don't do it. They're nuts.

#### Object Wrapper Gotchas

Using object wrappers directly can introduce gotchas. For instance:
```js
var a = new Boolean(false);
if (!a) {
  console.log("Oops"); // This will never run
}
```
Here, `a` is an object wrapper around `false`, but objects are truthy, so the condition behaves oppositely to what one might expect. To get the underlying primitive value from an object wrapper, use the `valueOf()` method: 

```js 
var a = new String("abc"); 
var b = new Number(42); 
var c = new Boolean(true); 

console.log(a.valueOf()); // "abc" 
console.log(b.valueOf()); // 42 
console.log(c.valueOf()); // true
```
Unboxing can also happen implicitly when using an object wrapper in a way that requires the primitive value.

Also, be very careful not to use Array.prototype as a default value that will subsequently be modified. In this example, vals is used read-only, but if you were to instead make in place changes to vals , you would actually be modifying Array.prototype itself, which would lead to the gotchas mentioned earlier!

In conclusion, while JavaScript's native constructors provide powerful functionality, their direct use can lead to unexpected results. It's generally better to use primitive values and let JavaScript handle the boxing and unboxing implicitly.
# References
