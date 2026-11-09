---
created: 2025/12/18  -  17:54
updated: 2026-11-09 20:56
type: note
status: seed
topic:
tags:
  - web/javascript
  - memory
  - garbage_collection
aliases: []
---


Although JavaScript does not directly expose the garbage collector API, the language offers several data structures that indirectly observe garbage collection and can be used to manage memory usage.

### [WeakMaps and WeakSets](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Memory_management#weakmaps_and_weaksets)

[`WeakMap`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/WeakMap) and [`WeakSet`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/WeakSet) are data structures whose APIs closely mirror their non-weak counterparts: [`Map`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map) and [`Set`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Set). `WeakMap` allows you to maintain a collection of key-value pairs, while `WeakSet` allows you to maintain a collection of unique values, both with performant addition, deletion, and querying.

`WeakMap` and `WeakSet` got the name from the concept of _weakly held_ values. If `x` is weakly held by `y`, it means that although you can access the value of `x` via `y`, the mark-and-sweep algorithm won't consider `x` as reachable if nothing else _strongly holds_ to it. Most data structures, except the ones discussed here, strongly holds to the objects passed in so that you can retrieve them at any time. The keys of `WeakMap` and `WeakSet` can be garbage-collected (for `WeakMap` objects, the values would then be eligible for garbage collection as well) as long as nothing else in the program is referencing the key. This is ensured by two characteristics:

- `WeakMap` and `WeakSet` can only store objects or symbols. This is because only objects are garbage collected — primitive values can always be forged (that is, `1 === 1` but `{} !== {}`), making them stay in the collection forever. [Registered symbols](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Symbol#shared_symbols_in_the_global_symbol_registry) (like `Symbol.for("key")`) can also be forged and thus not garbage collectable, but symbols created with `Symbol("key")` are garbage collectable. [Well-known symbols](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Symbol#well-known_symbols) like `Symbol.iterator` come in a fixed set and are unique throughout the lifetime of the program, similar to intrinsic objects such as `Array.prototype`, so they are also allowed as keys.
- `WeakMap` and `WeakSet` are not iterable. This prevents you from using `Array.from(map.keys()).length` to observe the liveliness of objects, or get hold of an arbitrary key which should otherwise be eligible for garbage collection. (Garbage collection should be as invisible as possible.)

In typical explanations of `WeakMap` and `WeakSet` (such as the one above), it's often implied that the key is garbage-collected first, freeing the value for garbage collection as well. However, consider the case of the value referencing the key:

```javascript
const wm = new WeakMap();
const key = {};
wm.set(key, { key });
// Now `key` cannot be garbage collected,
// because the value holds a reference to the key,
// and the value is strongly held in the map!
```

If `key` is stored as an actual reference, it would create a cyclic reference and make both the key and value ineligible for garbage collection, even when nothing else references `key` — because if `key` is garbage collected, it means that at some particular instant, `value.key` would point to a non-existent address, which is not legal. To fix this, the entries of `WeakMap` and `WeakSet` aren't actual references, but [ephemerons](https://dl.acm.org/doi/pdf/10.1145/263700.263733), an enhancement to the mark-and-sweep mechanism. [Barros et al.](https://www.jucs.org/jucs_14_21/eliminating_cycles_in_weak/jucs_14_21_3481_3497_barros.pdf) offers a good summary of the algorithm (page 4). To quote a paragraph:

> Ephemerons are a refinement of weak pairs where neither the key nor the value can be classified as weak or strong. The connectivity of the key determines the connectivity of the value, but the connectivity of the value does not affect the connectivity of the key. […] when the garbage collection offers support to ephemerons, it occurs in three phases instead of two (mark and sweep).


# References
