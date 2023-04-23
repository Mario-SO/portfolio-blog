---
date: 2023-04-23T12:00:02-04:00
draft: false
tags: [huff]
title: "Day16 - They see me huffing, they hatin'."
---

Hehe sorry, but I can't stop playing with huff, It is cool when you start to be able to fix your own mistakes, I think that's a point in which you are starting to understand new stuff.

So today I bring two more challenges from the repo I shared in a previous post.

# Non-Payable

The task is to write within the `MAIN` macro, huff code that reverts if ether is sent with the transaction. In solidity terms, revert if `msg.value > 0`.

```javascript
#define macro MAIN() = takes(0) returns(0) {
    callvalue iszero   // [callvalue == 0 ? 1 : 0]
    ok                 // [ok, callvalue == 0 ? 1 : 0]
    jumpi              // []

    0x20 0x00 revert   // Revert, callvalue != 0

    ok:
        0x01 0x00 return // Return 1, callvalue == 0
}
```

Not very difficult, but this is meant to be an introduction to the **OPCODE** `jumpi` (jump if), which is the equivalent to solidity's `if` statement (there is also the `jump` **OPCODE** which is a forced jump)

If you see my previous post on the *Multiply* challenge, you'll see that I needed to separate each instruction with a new line, now I'm starting to keep some instructions together in the same line because I'm getting more familiar with some combinations of **OPCODEs** and how they interact.

### Understadning JUMPI

It is really simple, just like many **OPCODEs** like `retur`, `iszero`... `jumpi` takes two arguments, the first one is the `JUMPDEST` (the jump destination/label) and the second one should be either `0` or `1`, if it is `1` it will jump to the `JUMPDEST` and if it is `0` it will continue with the next instruction.

So it is crucial that the instruction right before the `jumpi` is a label and there is already in the stack either a `0` or `1` resulting from instructions `iszero`, `eq`...

# FooBar

The task is to write within the `MAIN` macro below, huff code that mimics 2 solidity functions.

- One named `foo()` that simply returns 2,
- the second named `bar()` that simply returns 3.

```javascript
#define function foo() payable returns(uint256)
#define function bar() payable returns(uint256)


#define macro MAIN() = takes(0) returns(0) {
    // Get function signature
    0x00 calldataload   // [calldata]
    0xe0 shr            // [__FUNC_SIG]

    // dup1 __FUNC_SIG(foo) eq foo jumpi
    dup1                // [__FUNC_SIG, __FUNC_SIG]
    __FUNC_SIG(foo)     // [foo, __FUNC_SIG, __FUNC_SIG]
    eq                  // [foo == __FUNC_SIG, __FUNC_SIG]
    foo                 // [foo, __FUNC_SIG]
    jumpi

    // dup1 __FUNC_SIG(bar) eq bar jumpi
    dup1                // [__FUNC_SIG, __FUNC_SIG]
    __FUNC_SIG(bar)     // [bar, __FUNC_SIG, __FUNC_SIG]
    eq                  // [bar == __FUNC_SIG, __FUNC_SIG]
    bar                 // [bar, __FUNC_SIG]
    jumpi

    0x00 0x20 revert    // Revert if unrecognized function is called

    foo:
        pop
        0x02 0x00 mstore
        0x20 0x00 return
    bar:
        pop
        0x03 0x00 mstore
        0x20 0x00 return
}
```

Uh pretty scary right? No worries, let's break it down.

### Getting the function signature

```javascript
// Get function signature
0x00 calldataload
0xe0 shr            // [__FUNC_SIG]
```
Here we are getting the first 4 bytes of the `calldata` (the data sent to the contract) which is the function signature, and we are storing it in the stack. 

The `shr` **OPCODE** is a bit tricky, it shifts bits to the right.

We obtain *calldata* which is `32` bytes long, and we want to get the first 4 bytes, so we need to shift `28` bytes to the right, which is `28 * 8 = 224` bits = `0xe0` in hex, and that gets stored in the stack.

### Jumping to the right function

```javascript
dup1 __FUNC_SIG(foo) eq foo jumpi
dup1 __FUNC_SIG(bar) eq bar jumpi
```

In the code I broke it down so it is easier to follow the stack state, but look how simple it is when you put it all together, rigth?

The thing to take from this one is that you wil be using `dup1` **OPCODE** a lot, it duplicates the first element in the stack and this is key because when you perform an operation like `eq` or `iszero`, this will take values off the stack and you need to keep the original value to perform the jump.

### Returning values

```javascript
foo:
    pop
    0x02 0x00 mstore
    0x20 0x00 return
bar:
    pop
    0x03 0x00 mstore
    0x20 0x00 return
```

This is pretty straight forward, we are just storing the value we want to return in memory and then returning it depending on the function signature. we obtained previously.

## Wrapping up

I'm starting to get the hang of it, I'm still not able to write *"production ready"* (even though huff devs don't recomend it but you get my point) code, but I'm getting there.

Hopefully I can start writting about my experience with the Secureum RACE I just joined plus some POCs, I think those are very interesting to share and also I'm aware that huff posts are not very popular, but I'm having fun and I hope you are too.