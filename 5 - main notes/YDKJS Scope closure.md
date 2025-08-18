2024/07/27  -  11:54

Tags: [[closures]] [[js]] 

# Closures
Closure is when a function is able to remember and access its lexical scope even when that function is executing outside its lexical scope.

```js
for ( var i=1 ; i<=5 ; i++ ) {
	setTimeout( function timer () {
		console.log( i );
	}, i*1000);
}
```
the timeout function calls are running strictly after the completion of the loop and so they print 6 each time. the way scope works, all 5 those functions, though they are defined separately in each iteration, all are closed over the same shared global scope, which has only `i` in it.

```js
for (var i=1; i<=5; i++) { 
	(function(){ 
		setTimeout( function timer(){ 
			console.log( i );
		}, i*1000 ); 
	})();
}
```
this doesn't work either, because of the IIFE now function callbacks are closed over their own per-iteration scope, but that scope is empty, it needs its own variable with a copy of the `i` value a each iteration.

```js
// it works!
for (var i=1; i<=5; i++) { 
	(function(){ 
		var j = i;
		setTimeout( function timer(){ 
			console.log( j );
		}, j*1000 ); 
	})();
}

// A slight variation some prefer is:
for (var i=1; i<=5; i++) { 
	(function(j){ 
		setTimeout( function timer(){ 
			console.log( j );
		}, j*1000 ); 
	})( i );
}

// let essentially turns a block into a scope that we can close over.
for ( let i=1; i<=5 ; i++) {
	setTimeout ( function timer() {
		console.log( j );	
	}, i*1000)
}

```
# References
