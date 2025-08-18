
tags: [[react]] [[DOM]]

## reconciliation
is the algorithm behind virtual DOM. react uses nodes to create an entire state of app, called current tree and then it gives it to rendering environment. when there is a change in our DOM react creates another tree called work-in-progress tree, it comperes these tress and only renders the different parts, then work in progress tree becomes current tree.

## prioritizing DOM operations 
react splits interface update into small units and prioritize them for example a hover is more important than some info that user cannot see.

```js
window.requestAnimationFrame(callback); // for more important tasks

window.requestIdleCallback(callback[,options]); // for less important tasks 
```

# References
 