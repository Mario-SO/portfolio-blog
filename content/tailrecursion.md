---
date: 2022-07-30T15:35:06+02:00
draft: false
tags: [concept, JavaScript, programming, algorithms]
title: "Tail recursion concept"
---

### Tail Recursion

We call **tail recursion** to a function which has its last instruction being a recursive call. **They are recursive functions that can be used without the fear of suffering a stack overflow.**

Normal recursion ❌

```python
def factorial (n):
	if n == 1:
		return 1
	return n * factorial(n-1)
```
This is an example of a normal recursion, because the last operation is not the recursive call, even though it seems that it is the last thing you do is the recursive call, the very last thing this code does is the multiplication.
 
Tail recursion ✅

```python
def fact (n, a = 1):
	if (n == 1) :
		return a
	return fact (n - 1, n * a)
```
In a tail recursion, you need to include the initial values in the first call.

Not every language accepts this programming scheme, in Python you can get it by importing external libraries. It is more normal that functional programming languages like haskell support this by default.