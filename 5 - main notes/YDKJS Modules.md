---
created: 2024/07/27  -  20:04
updated: 2026-11-09 22:48
type: note
status: seed
topic:
tags:
  - web/javascript
  - modules
aliases: []
---

Modules require two key characteristics:
1. an outer wrapping function being invoked, to create the enclosing scope.
2. the return value of the wrapping function must include reference to at least one inner function that then has closure over the private inner scope of the wrapper.
# References
