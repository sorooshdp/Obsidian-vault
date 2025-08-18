2024/07/26  -  19:01
status: 

Tags: [[js]] [[closures]] [[scope]]

# lexical scope
lexical scope is based on where you define the variables and blocks at write time. engine looks for variables in the scope and if it can't find it in that scope, goes one level up and stops once it finds the first match.  
The same identifier name can be specified at multiple layers of nested scope, which is called "shadowing"

Two mechanisms in JavaScript can "cheat" lexical scope: eval(..) and with . The former can modify existing lexical scope (at runtime) by evaluating a string of "code" which has one or more declarations in it. The latter essentially creates a whole new lexical scope (again, at runtime) by treating an object reference as a "scope" and that object's properties as scoped identifiers.
# References
