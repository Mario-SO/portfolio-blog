---
date: 2023-04-21T17:23:27-04:00
draft: false
tags: [huff]
title: "Day14 - Moar huffing"
---

I couldn't get too much done today, made a very hard and sad decision and... well, I needed to rest and take some time for myself.

So I can only show what I did today, which is not much, and actually has been between today and yesterday.

# More huff challenges

I was only able to finish the *Multiply* challenge from the repo I shared in the previous post.

he task is to write within the `MAIN` macro below, a function named `multiply` that takes in 2 uint256s, and returns their product. Be sure to revert on overflow.

```javascript
#define function multiply(uint256, uint256) payable returns(uint256)


#define macro MAIN() = takes(0) returns(0) {
    0x04 calldataload // [arg1]
    0x24 calldataload // [arg2, arg1]
    mul               // [arg1 * arg2]

    dup1              // [arg1 * arg2, arg1 * arg2]
    iszero            // [arg1 * arg2 == 0 ? 1 : 0, arg1 * arg2]
    multiply          // [multiply, arg1 * arg2 == 0 ? 1 : 0, arg1 * arg2]
    jumpi             // if arg1 * arg2 == 0, jump to multiply
                      // [arg1 * arg2]

    dup1              // [arg1 * arg2, arg1 * arg2]
    0x04 calldataload // [arg1, arg1 * arg2, arg1 * arg2]
    swap1             // [arg1 * arg2, arg1, arg1 * arg2]
    div               // [arg1 * arg2 / arg1, arg1 * arg2]
    0x24 calldataload // [arg2, arg1 * arg2 / arg1, arg1 * arg2]
    eq                // [arg1 * arg2 / arg1 == arg2 ? 1 : 0, arg1 * arg2]
    multiply          // [multiply, arg1 * arg2 / arg1 == arg2 ? 1 : 0, arg1 * arg2]
    jumpi             // if arg1 * arg2 / arg1 == arg2, jump to multiply
                      // [arg1 * arg2]
                      
    0x00 mstore
    0x20 0x00 revert

    multiply:
        0x00 mstore
        0x20 0x00 return
}
```

This one was a hell of a ride, and hard af to solve, it really took some time. It was key to be really careful with the order of the operations, and to understand what was going on, with the comments I added.

Still, the repo, when running the tests for this challenge, has 1 passing test and another failing one that has to do with the function selector. I can guess why it is failing but don't know how to solve it.

## Wrapping up

Well not much today, but I'm happy with what I did. I'm still learning a lot, and I'm really enjoying this. I'm also happy with the progress I'm making, and I'm really looking forward to the next days when hopefully I'll be able to do some pair auditing.

See you soon!