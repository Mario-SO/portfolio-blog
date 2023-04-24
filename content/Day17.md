---
date: 2023-04-24T15:09:21-04:00
draft: false
tags: [huff]
title: "Day17 - Huff magic continues"
---

Hey there, huff enthusiasts! I'm back with more huff adventures and some insights into storage in Ethereum smart contracts. I can't wait to share them with you! I'm still on cloud nine from my recent progress in understanding huff and the EVM. So, today, let's dive into storage and how it works in Ethereum smart contracts.

# Ethereum Smart Contract Storage

In Ethereum, smart contract storage is a key-value store that allows you to store and access data persistently. The storage is part of the blockchain's state, meaning that once you store data in it, it will remain there indefinitely until you explicitly modify or delete it. However, keep in mind that modifying storage is costly in terms of gas, so it's **essential to optimize storage usage in your smart contracts**.

Ethereum smart contract storage is organized as a sparse `32-byte` key-value store, which means that it can handle `2^256` different keys. In practice, the number of keys a contract can use is far less than this theoretical maximum, but it's still quite large.

In Solidity, you can interact with storage using for example state variables. These high-level construct abstracts away the details of storage and automatically generate EVM bytecode to handle storage operations using `SSTORE` and `SLOAD` opcodes.

In huff, we use `SSTORE` and `SLOAD` to directly interact with storage.

# The challenge

The task is to write within the `MAIN` macro 2 functions...
    - One named `store()` that takes one function argument and stores it in storage slot 0,
    - the second named `read()` that simply returns what is stored at storage slot 0.

### Storing data in contract storage

To store something in contract storage, we first load it from calldata, add the `SLOT0` pointer to the stack, and then use the `SSTORE` opcode:

```javascript
store:
    pop
    0x04 calldataload // [uint256]
    [SLOT0] sstore    // [SLOT0, uint256] -> []
    stop
```

### Reading data from contract storage

To read from storage, we use the `SLOAD` opcode to load the value. This will add the value to the stack, and then we can use the `MSTORE` opcode and return the value like a pro:

```javascript
read:
    pop
    [SLOT0] sload     // [uint256]
    0x00 mstore       // []
    0x20 0x00 return
```

### What's that [SLOT0]?

Good question, I'm glad you asked! The `SLOT0` is a **FREE STORAGE POINTER**. It's a special pointer that points to the first storage slot of the contract. It's a constant value that can be used to access the storage of the contract.

You can take a look at huff docs to learn more about it.

The pop statement is there for the same reason as in the previous challenge, go check it out if you haven't already!

So the final code will look like this:

```javascript
#define function store(uint256) payable returns()
#define function read() payable returns(uint256)

// To work with storage we need to declare a FREE STORAGE POINTER
#define constant SLOT0 = FREE_STORAGE_POINTER()

#define macro MAIN() = takes(0) returns(0) {
    // Get function selector
    0x00 calldataload
    0xe0 shr
    dup1 __FUNC_SIG(store) eq store jumpi
    dup1 __FUNC_SIG(read) eq read jumpi
    0x00 0x20 revert

    // To store someting in storage we need first to load from calldata
    // then add to the stack the SLOT0 pointer and then use the SSTORE opcode
    store:
        pop
        0x04 calldataload // [uint256]
        [SLOT0] sstore    // [SLOT0, uint256] -> []
        stop

    // To read from storage we need to load the value with SLOAD
    // This will add it to the stack and then we can use the MSTORE opcode
    // and return as usual (remember that return, returns from memory)
    read:
        pop
        [SLOT0] sload     // [uint256]
        0x00 mstore       // []
        0x20 0x00 return
}
```

## Wrapping Up

My huff skills are leveling up with each challenge, and although I'm not quite at the "production-ready" stage, I'm certainly getting closer. I hope these posts are as enjoyable for you as they are for me, and that we can learn together!

Hopefully I can start bringing back some more solidity and foundry tips, I also really enjoy that, but as that's not as new as huff to me, well I'm enjoying huff more for now hehe.