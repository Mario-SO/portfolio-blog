---
date: 2023-04-15T14:11:20-04:00
draft: false
tags: [evm, CTF]
title: "Day08 - Going deeper into the EVM"
---

# As we were saying...

Today I went on and tried the [EVM Through CTFs](https://www.evmthroughctfs.com/) challenges and oh god are they good.

Even though the first challenge is a bit of a warmup (especially if you read my previous post [Day04 - Calldata & Foundry](https://blog.mariodev.xyz/day04/)) it rapidly gets harder and harder, I'm actually stuck in the last part of challenge one, but I'll get there.

But today I wanted to dive deeper into `opcodes` and `stack` as in the 3rd part of this first challenge.

# Opcodes & Stack resources

The first thing I did was to look at the [EVM Opcodes](https://ethervm.io/) page and [evm.codes](https://www.evm.codes/) too, I think they are the best and only resource you really need to understand how evm works.

Personally found the [playground](https://www.evm.codes/playground) very useful, but the other webpage is also great for reference.

# The challenge

You are presented with this unverified [contract](https://etherscan.io/address/0x36ce5aa25b99cf6eb019aafd149b97b32cdd4a5b#code) and you need to understand what it does and finally build your own `calldata` so the contract returns a certain value.

# The solution

### Testing the grounds

To understand what the contract does you need to understand the `opcodes` and the `stack` and so for that we have the [evm.codes playgroung](https://www.evm.codes/playground).

Simply paste the *bytecode* of the contract, change the dropdown option to `mnemonic` and you'll see the `opcodes` that result from that *bytecode*, paste the `calldata` you want to test and hit run, you'll see the `stack` and the `memory` after each `opcode` is executed.

When pasting the `calldata`

```javascript
0xb40d7b75
0000000000000000000000000000000000000000000000000000000000000003
0000000000000000000000000000000000000000000000000000000000000004
```

I was getting the value `0x1c` which in decimal is `28`, but still this value could be achieved by many different ways so I needed to see how it was getting there. (I won't go into the details of how I built the `calldata`, see my previous post [Day04 - Calldata & Foundry](https://blog.mariodev.xyz/day04/) for that)

### Understanding the contract

I don't think it makes sense here to go `opcode` by `opcode` and explain what they do, If you want you can take the contract and try to understand it yourself by doing what I said in the previous section.

The contract bascially run a series of operations to properly get the arguments we passed in the `calldata` and do some operation with them. In this case the formula it was running to get the result was `a * 8 + b` -> `3 * 8 + 4 = 28`.

So to get the value `13` we need to pass `a = 1` and `b = 5` in the `calldata`.

``` javascript
0x
b40d7b75
0000000000000000000000000000000000000000000000000000000000000001
0000000000000000000000000000000000000000000000000000000000000005
```

## Wrapping up

Sorry if you expected a super detailed description on how each `opcode` works, but I think the best way to learn is to try it yourself and see how it works.

Today was quite a good day for me, I now feel more confident with the EVM and how to interact with low level solidity.