2025/09/14  -  18:50
status: 

Tags: [[js]] 

## `<script> `

Most browser-viewed websites/applications have more than one file that contains their code, and it's common to have a few or several elements in the page that load these files separately, and even a few inline-code elements as well. But do these separate files/code snippets constitute separate programs or are they collectively one JS program? The (perhaps surprising) reality is they act more like independent JS programs in most, but not all, respects. The one thing they share is the single global object ( window in the browser), which means multiple files can append their code to that shared namespace and they can all interact.

So, if one script element defines a global function foo() , when a second runs, it can access and call foo() just as if it had defined the function itself. script later But global variable scope hoisting (see the Scope & Closures title of this series) does not occur across these boundaries, so the following code would not work (because foo() 's declaration isn't yet declared), regardless of if they are (as shown) inline elements or externally loaded files.

We know and can rely upon the fact that the JS language itself has one standard and is predictably implemented by all the modern browsers/engines. This is a very good thing! But JavaScript rarely runs in isolation. It runs in an environment mixed in with code from third-party libraries, and sometimes it even runs in engines/environments that differ from those found in browsers. Paying close attention to these issues improves the reliability and robustness of your code.





# References
