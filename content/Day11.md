---
date: 2023-04-18T12:05:22-04:00
draft: false
tags: [evm, CTF]
title: "Day11 - Code is not law"
---

Going back again with yet another challenge from [EVM Through CTFs](https://www.evmthroughctfs.com/), but this time, I don't have a solution yet, I'm still working on it.

Still, will share the point at which I'm at right now so in the future I can look back and see how stupid I was hehe.

# The challenge

Today's challenge consists on getting `CodeIsNotLaw` tokens from a minter contract, but it is not as straight forward as calling the mint function.

Here is the contract:

```javascript
contract CodeIsNotLaw is ERC20("CodeIsNotLaw", "CINL", 18) {
    address public immutable onlyImAllowedToHaveTokens;
    bytes32 public immutable correctCodeHash;

    constructor() {
        onlyImAllowedToHaveTokens = address(new OnlyImAllowedToHaveTokens());
        correctCodeHash = getContractCodeHash(onlyImAllowedToHaveTokens);
    }

    function mint(address receiver) external {
        require(
            getContractCodeHash(receiver) == correctCodeHash,
            "code hash does not match"
        );
        if (balanceOf[receiver] == 0) {
            _mint(receiver, 1);
        }
    }
```

We need to have the correct code hash to be able to mint tokens, therefor, we cannot call this function from our EOA, we need to call it from a contract, plus, this contract must have the exact same bytecode as the `OnlyImAllowedToHaveTokens` contract, which is this one:

```javascript
contract OnlyImAllowedToHaveTokens {
    function bye() external {
        selfdestruct(payable(msg.sender));
    }
}
```

I you read my [previous post](https://blog.mariodev.xyz/day10/) you may have an idea on how to first approach the challenge, and you would be correct but still there is the tricky part of, once you have the right code hash, how do you call the mint function from a contract with the same bytecode as the `OnlyImAllowedToHaveTokens` contract?

# The solution

I know it must be from the constructor where we need to call the mint function, but I don't know why it is failing...

Will keep working on it and update this post once I have a solution.