---
date: 2022-08-01T15:29:24+02:00
draft: true
tags: [concept, programming, JavaScript]
title: "Callback Vs. Promise functions in JavaScript"
---
## Diferencias entre Callback y Promise en Js

```js
const operation = (n1, n2, op) => {
	return op(n1, n2)
}

operation (1, 2, (a, b) => a + b)
```
In this case we can see a *Callback function* being this, the function **op** which is then passed as a parameter to the **operation** function. Nowadays is uncommon to use callbacks unless they are required by a library or it is legacy code. For example the function *map* is a callback.

```js
const promises = (n1, n2) => {
	const reusltado = n1 + n2
	return new Promise(resolve => {
		setTimeout(() => {
			resolve(resultado)
		}, 500)	
	})
}

promises(1,3)
	.then(result => console.log(result))
```
When talking about promises, even though you write longer code, when consuming the functions, you have much more options for customizing the behavior of the promise.