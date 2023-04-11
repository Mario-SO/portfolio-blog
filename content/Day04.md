---
date: 2023-04-11T10:53:41-04:00
draft: false
tags: [evm]
title: "Day04 - Calldata & Foundry"
---

The other day, I stumbled upon a tweet from [@DeGatchi](https://twitter.com/DeGatchi) promoting his latest blog post about [Reversing the EVM: Raw Calldata](https://degatchi.com/articles/reading-raw-evm-calldata) and I thought it was so helpful that I decided to write a post about it, not only to share it with you (rando reading this) but also to have a reference for myself.

It includes a `foundry` CLI mini-masterclass (if that's a thing).

# Calldata 101
Super brief *ELI5* explanation of calldata:

`Calldata` is the encoded data passed to a funtion in the form of bytes, and there are two types, *dynamic* and *static*.

## Setting up the playground

I recently learned how powerful foundry can be, not only for testing but also because of its different CLI tools.

1. Start by running `anvil` to launch a local node.
2. I recommend exporting a couple private keys and addresses to your environment variables.
```bash
export PK1=0x00000000
export PK2=0x00000000
export addr1=0x23618e81E3f5cdF7f54C3d65f7FBc0aBf5B21E8f
export addr2=0xa0Ee7A142d267C1f36714E4a8F75612F20a79720
```
3. Leave the node running and open a new terminal window to run `forge init <project-name>`.
4. Finally `cd` into `<project-name>` and open your fav code editor.

---

### Encoding calldata

Imagine we have this contract:

```javascript
interface IDay04 {
    function transfer(uint256[] memory ids, address to) external;
}

contract Day04 {
    function checkCalldata(uint256[] memory ids, address to) external pure returns (bytes memory) {
        return abi.encodeWithSelector(IDay04.transfer.selector, ids, to);
    }
}
```

Let's deploy the contract with the following command:

```javascript
forge create --private-key $PK1 --rpc-url http://127.0.0.1:8545 src/Day04.sol:Day04 
```
You will now see some stuff in the `anvil` terminal window, that's the transaction details for the contract creation, take the contract address and run:

```javascript
cast send <contract-address> "checkCalldata(uint256[],address)" "[1,2,3]" $addr1 --from $addr2 
```
This is the way we send a transaction to the contract in our local node.

To send a transaction to a real world contract *(testnet or mainnet)* you will need to specify a couple more parameters *(maybe next post)*, but for now this is enough.

```javascript
cast tx <tx-hash>
```

This will let us see the transaction details.

In the `input` field you'll find something like this *(depending on the parameters you used, you may see slightly different values)*:
    
```javascript
0x8229ffb6
0000000000000000000000000000000000000000000000000000000000000040
0000000000000000000000009965507d1a55bcc2695c58ba16fb37d819b0a4dc
0000000000000000000000000000000000000000000000000000000000000003
0000000000000000000000000000000000000000000000000000000000000001
0000000000000000000000000000000000000000000000000000000000000002
0000000000000000000000000000000000000000000000000000000000000003
```

This is the raw calldata, it is scary af, it was for me too before reading about how to interpret it.

---

### Understanding the output

- The first **4 bytes** *(8 bits)* are the function selector, which is the first **4 bytes** of the `keccak256` hash of the function signature. In this case, the function signature is `transfer(uint256[],address)` and the `keccak256` hash of it is `0x8229ffb6`.

- The next **32 bytes** *(64 bits)* are the offset to the start of the `ids` array. In this case, the offset is `0x40` which is 64 in decimal.

- The next **32 bytes** *(64 bits)* are the address of the `to` address. In this case, the address is `0x9965507d1a55bcc2695c58ba16fb37d819b0a4dc`.

- The next **32 bytes** *(64 bits)* are the length of the `ids` array. In this case, the length is `0x3` which is 3 in decimal (As you can see, this part is dynamic depending on the length of the array).

- The next **32 bytes** *(64 bits)* are the first element of the `ids` array. In this case, the first element is `0x1` which is 1 in decimal.

- The next **32 bytes** *(64 bits)* are the second element of the `ids` array. In this case, the second element is `0x2` which is 2 in decimal.

- The next **32 bytes** *(64 bits)* are the third element of the `ids` array. In this case, the third element is `0x3` which is 3 in decimal.

### Creating our own calldata

Now that we know how to read calldata, we should be able to create our own.

Let's try to create the calldata for the `mint` function:

```javascript
interface IDay04 {
    function mint(uint8 amount, uint256 id, address to) external;
}
```

Take into account the `uint8` type. **What do you think the calldata will look like?**

```javascript
Parameters:
amount: 1
id: 23
addr: 0x9965507d1a55bcc2695c58ba16fb37d819b0a4dc
```
First we need to know the function selector, which is the first **4 bytes** of the `keccak256` hash of the function signature. In this case, the function signature is `mint(uint8,uint256,address)` and the `keccak256` hash of it is `0x2715eb15703f2d8697ebb56dbe6d9311aee605bf91ddcfea9b2264e30bd4d4a0` so first **4 bytes** of it are `0x2715eb15`.

So the raw calldata will look like this:

```javascript
0x2715eb15
0000000000000000000000000000000000000000000000000000000000000001
0000000000000000000000000000000000000000000000000000000000000017
0000000000000000000000009965507d1a55bcc2695c58ba16fb37d819b0a4dc
```
Remember that this is output in hexadecimal, so the 17 you see there is actually 23 in decimal.

**Ey! but what about the uint8? Doesn't it change the lenght of something somewhere?** Well, the `uint8` type is the same as the `uint256` type in terms of encoding, the only difference is that the `uint8` type is limited to 8 bits, so the value will be truncated to 8 bits.

To prove that we are right, let's again use `foundry` but this time, to further increase our knowledge of `foundry` toolset, we won't be transmiting a tx to the node, instead, use the `calldata` command.

```javascript
cast calldata "mint(uint8,uint256,address)" 1 23 0x9965507d1a55bcc2695c58ba16fb37d819b0a4dc 
```
Returns:

```javascript
0x2715eb15
0000000000000000000000000000000000000000000000000000000000000001
0000000000000000000000000000000000000000000000000000000000000017
0000000000000000000000009965507d1a55bcc2695c58ba16fb37d819b0a4dc
```
---

## Wrapping up

This is barely the tip of the iceberg, there is a lot more to learn about calldata, we just scratched the surface of dynamic calldata and didn't even talk about multicall *([DeGatchi](https://degatchi.com/articles/reading-raw-evm-calldata) does in his blog)*, but I hope this was enough to get you started and interested in the topic plus see how powerful `foundry` can be and how important it is to know the tools you are using.

## Bonus

As you may see, this output is quite horrible to read, so we can use the `foundry` toolset to make it more readable.

```javascript
cast pretty-calldata <calldata>
```
This will return:

```javascript
Possible methods:
- mint(uint8,uint256,address)
------------
[000]: 0000000000000000000000000000000000000000000000000000000000000001
[020]: 0000000000000000000000000000000000000000000000000000000000000017
[040]: 0000000000000000000000009965507d1a55bcc2695c58ba16fb37d819b0a4dc
```



