2024/07/27  -  20:04

Tags: [[js]] [[modules]] 

Modules require two key characteristics:
1. an outer wrapping function being invoked, to create the enclosing scope.
2. the return value of the wrapping function must include reference to at least one inner function that then has closure over the private inner scope of the wrapper.
# References
