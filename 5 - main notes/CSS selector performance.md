2024/08/08  -  14:04

Tags: [[CSS]] [[Performance]]

## recalculation 
Whenever a part of your page changes, the browser engine that’s running it needs to take a look at the new DOM tree, and figure how to style it based on the available CSS stylesheets. This operation of matching styles to DOM nodes is called a style recalculation.

the browser engine needs to look at all your rules and make decisions as to which ones apply to a given element. engine needs to look at the rule selector, and this happens from right to left.

For example, when the engine sees a selector like `.wrapper .section .title .link` it will try to match the `link` class with the element first, and if that matches, then go up the chain from right to left to find an ancestor element with class `title`, then one with class `section`, and finally one with class `wrapper`.

`[class*="icon-"]`This type of selector requires the browser engine to not only check if the element has a class attribute but also whether the value of this attribute contains the substring `icon-`.

Overly complex CSS selectors, coupled with a huge DOM tree that changes a lot could very well lead to bad performance
# References
https://blogs.windows.com/msedgedev/2023/01/17/the-truth-about-css-selector-performance/