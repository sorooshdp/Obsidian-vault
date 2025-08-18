2025/04/10  -  18:36

Tags: [[js]] [[software engineering]]

## Shallow Copy Defined

A shallow copy creates a new object that stores exact copies of the original object's values. However, for nested objects or arrays, it only copies their references rather than creating new instances of the nested structures[1](https://www.geeksforgeeks.org/difference-between-shallow-and-deep-copy-of-a-class/)[2](https://medium.com/@manjuladube/understanding-deep-and-shallow-copy-in-javascript-13438bad941c). In essence, a shallow copy duplicates the top-level structure while maintaining references to the same nested objects as the original.

When shallow copying is performed, both the original and copied object will share references to the same underlying nested data structures. This means that modifying a nested object in either the original or the copy will affect both objects[2](https://medium.com/@manjuladube/understanding-deep-and-shallow-copy-in-javascript-13438bad941c).

## Deep Copy Defined

A deep copy creates a completely independent duplicate of an object, including all nested objects and arrays[1](https://www.geeksforgeeks.org/difference-between-shallow-and-deep-copy-of-a-class/)[3](https://dev.to/jahid6597/beyond-cloning-sculpting-javascript-mirrors-with-the-art-of-shallow-copy-and-the-depth-of-deep-copy-3p64). It recursively copies all levels of the data structure, allocating new memory for each element rather than preserving references to the original data[1](https://www.geeksforgeeks.org/difference-between-shallow-and-deep-copy-of-a-class/). This results in two entirely separate objects with identical values but no shared references.

Deep copies ensure complete isolation between the original and copied data structures, so changes to one will never affect the other[2](https://medium.com/@manjuladube/understanding-deep-and-shallow-copy-in-javascript-13438bad941c).

## Implementation Approaches

```js
const shallowCopy = { ...originalObject };
const shallowCopy = Object.assign({}, originalObject);
const shallowArrayCopy = originalArray.slice(); 
const anotherShallowCopy = [].concat(originalArray);
```


```js
const deepCopy = JSON.parse(JSON.stringify(originalObject));

function deepCopy(obj) {   
	if (typeof obj !== "object" || obj === null) {return obj;}  
	
	const newObj = Array.isArray(obj) ? [] : {}; 
	 
	for (const key in obj) {    
		if (obj.hasOwnProperty(key)) {      
		newObj[key] = deepCopy(obj[key]);
		}  
	}  
	
	return newObj; 
}
```

This approach provides more control and can handle complex structures[3](https://dev.to/jahid6597/beyond-cloning-sculpting-javascript-mirrors-with-the-art-of-shallow-copy-and-the-depth-of-deep-copy-3p64).

# References
