---
date: 2023-04-17T11:13:25-04:00
draft: false
tags: [evm, CTF]
title: "Day10 - Same bytecode, different results"
---

Continuing with the [EVM Through CTFs](https://www.evmthroughctfs.com/) challenges, today I was able to solve a pretty interesting challenge.

# The challenge

In this case the challenge consisted on winning a game agains a bot smart contract *(khabib)* at the game [0xships](https://0xship.vercel.app/).

The thing is that the bot *khabib* is quite a good player and his code is pretty complex, so trying to build your own bot was completely out of the equation.

This is *khabib*'s code:

```javascript
0x608060405234801561001057600080fd5b50600436106100415760003560e01c806306fdde0314610046578063436cc795146100775780638da5cb5b14610098575b600080fd5b604080518082018252600681526535b430b134b160d11b6020820152905161006e919061065b565b60405180910390f35b61008a6100853660046106a9565b6100b3565b60405190815260200161006e565b6000546040516001600160a01b03909116815260200161006e565b60008360ff036100cf576100c8868585610289565b905061027f565b600684901c6000819003610119576100e987866001610310565b15610101576100f9878686610289565b91505061...
```

Hehehehe, yeah, thats part of the challenge, you have to think of a way to beat him without having access to his high level code.

As we are doing a great job at understanding the **EVM**, this is not going to be a problem, right?

Well... let's see.

# The solution

First we need to learn how to make a smart contract have the same bytecode as another one.

```javascript
pragma solidity ^0.8.17;

contract ShipSolution {
    constructor() {
        bytes memory bytecode = hex"60806040523480156100...";
        assembly {
            return(bytecode, mload(bytecode))
        }
    }
}
```

This is how you do it, you just need to copy the bytecode of the contract you want to reproduce and then return it in the constructor, but there is a *catch*, can you see it?

Well, take into account that we are doing low level stuff, therefor we must be very explicit in how we want our code to behave.

### The catch

Ask yourself, **who is the owner of the contract?**

We need to trace down the owner function in the original bytecode from *khabib* and see how it behaves.

Go and copy the bytecode from *khabib*'s contract and paste it in [evm.codes](https://evm.codes/), then search for the owner function and you will find that the owner function is simply returning the *0th slot* of *storage*.

To fix this we need to slightly modify our code:

```javascript
pragma solidity ^0.8.17;

contract ShipSolution {
    address public owner;

    constructor() {
        owner = msg.sender;
        bytes
            memory bytecode = hex"60806040523480156100...";
        assembly {
            return(add(bytecode, 0x20), mload(bytecode))
        }
    }
}
```
### Explanation

So, why is this the solution?

It is not difficult to understand if you think about it, the owner function is returning the *0th slot* of *storage*, so we need to make sure that the *0th slot* of *storage* is the address of the contract creator, being an address means that it is **20 bytes** long.

### Important note

Something that I didn't mention is, we were able to take the bytecode from *khabib* and paste it into [evm.codes](https://evm.codes/) and work with it because of the fact that it was an **unverified** contract.

**Unverified** only show the runtime bytecode (which in this case it is what we need). **Verified** contracts on the other hand, show both initialization and runtime bytecode and it is not possible to distinguish between them (at least as far as I know).

## Wrapping up

Challenges from [evm through ctf](https://www.evmthroughctfs.com/) are getting harder and harder, but I'm really enjoying them, I'm learning a lot and I'm sure I'll be able to apply this knowledge in the future.

In case you wonder how are this special contracts called, they are called **metamorphic contracts** and if you found this interesting, I encourage you to read more about them.