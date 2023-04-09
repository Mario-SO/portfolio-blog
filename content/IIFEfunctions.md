---
date: 2022-08-01T14:39:32+02:00
draft: true
tags: [Concept, JavaScript, Programming]
title: "IIFE Functions in JavaScript"
---

## IIFE - Immediately Invoked Function Expression

```js
(() => {
	console.log('Hello World')
})()
```
**IIFE** functions look like that, they are wrapped between parentheses and at the end they are wrapped back again by two parentheses this time invoking the function.


## Why would you use an IIFE?

### Reason 1: Async - Await

```js
(async () => {
	await fetch('https://mariodev.xyz')
})();
```
This kind of functions were used in the past to make the *async - await* work in node, but nowadays it is not necessary anymore because **top-level** await is supported in modern versions of node and most runtimes.

### Reason 2: Variable Scope

```js
let a = "Hello";
(() => {
	let b = " World"
	console.log(a + b)
})();
```
In this block of code we have a variable *b* and a *console.log()* function which can access both the variable *a* and the variable *b* but a function outside the **IIFE** won't be able to access the variable *b* because the IIFE has its own **scope** (the variables declared inside are not accessible outside).

### Reason 3: Threads (*Not an important one nowadays*)

A function declared normally `const f = () => {}`, depending on what it does, when it is called (since it is executed immediately) could block the **main thread** execution and slow down the program (we talk about nanosegs) but the IIFEs wouldn't have that problem.

## Extra

When using **IIFE** it is very important to add the **;** before and after the IIFE, because otherwise, the interpreter will see everything as if it was in the same line, and won't be able to run properly and it will 💥.
