---
layout: post
title: "Coroutine"
description: "Coroutine"
categories: [concurrency]
tags: [concurrency]
redirect from:
  - /2018/11/11/
---
## Coroutine
coroutine is control structure- similar to python yield
When the "yield" occurs, the current state of the function is saved and control is returned to the calling function. The calling function can then transfer execution back to the yielding function and its state will be restored to the point where the "yield" was encountered and execution will continue. 

In python, a line such as `yield item` produces a value that is received by the caller of `next()` and it also gives way, suspending the execution of the generator so that the caller may proceed until its ready to consume another value by invoking `next()` again. 
Four states for a coroutine:
(1) created: waiting to start execution
(2) running: being executed
(3) suspend: suspend at a yield expression
(4) close: complete execution
only make a `send` call to coroutine when it is suspended. 

`yield from` main feature is to open a bidirectional channel from the outermost caller to the innermost subgenerator, so that values can be sent and yielded back and forth directly from them, and exceptions can be thrown all the way. 