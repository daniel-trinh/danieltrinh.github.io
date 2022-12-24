---
categories: scala snippet
comments: true
date: "2013-11-16T00:00:00Z"
published: false
title: Debugging Scala Initialization Order Problems
---
explain problem
example of code that causes initialization order problem (with exception/trace)
example of slightly tweaked code that doesn't have the problem
list alternate ways of solving
mention compiler flag
give build scala example with compiler flag 
give example of the compiler finding an initialization order problem
link to paulp article



In the absence of "early definitions" (see below), initialization of strict vals is done in the following order.

    Superclasses are fully initialized before subclasses.
    Otherwise, in declaration order.

superclass = traits


``
Something I found interesting is that calling `Future(...)` (or `future(...)`) is significantly slower than calling `Future.successful(...)` on an already completed block of code (over a magnitude slower when `...` is a primitive type, see below).

Calling `Future(...)` will schedule the block of code to be executed asynchronously,
whereas calling `Future.successful(...)` will simply take the value of the `...` code and place it in 
a Future object -- there's no scheduling overhead with using `.successful`. In the example above,
`future(List.empty[T])` can become `Future.succesful(List.empty[T])`, since the empty list is
already completed and doesn't need to be run asynchronously.

Anyway, these are some benchmark timings showing `Future(<Int goes here>)` vs `Future.successful(<Int goes here>)` on my machine, run 10, 100, 1000, and 10000 times:

```
       benchmark length        ns linear runtime
          Future     10    2548.9 =
          Future    100   11883.9 =
          Future   1000  157872.0 ==
          Future  10000 1945644.0 ==============================
FutureSuccessful     10      92.2 =
FutureSuccessful    100     821.0 =
FutureSuccessful   1000   11269.2 =
FutureSuccessful  10000  116668.8 =
```

The overhead is barely noticeable if it's just called once, but it can add up
if done in a critical loop.