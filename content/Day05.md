---
date: 2023-04-12T09:57:24-04:00
draft: false
tags: [audit]
title: "Day05 - 3 attack vectors"
---

Today I was lucky to get in time to an [@opensensepw](https://twitter.com/opensensepw) discord study session where [@mis4nthr0pic](https://twitter.com/mis4nthr0pic) was going through a couple of attack vectors in the [Solidity-Security-Compendium](https://github.com/obheda12/Solidity-Security-Compendium)

# Signature Malleability

This is quite a math heavy thing when you start looking into how it actually works and why, but for the context of an auditor it is enough to know a couple of things.

- In Ethereum, the ecrecover function is used to verify signatures. It is a [pre-compiled contract](https://medium.com/@Beosin_com/security-audit-series-what-is-a-precompiled-contract-vulnerability-5174e20e24e9) that performs public key recovery for elliptic curve cryptography. This means it can recover a public key (address) from a given signature.

- In ECDSA, a digital signature is represented by a pair of values `(r, s)`. These values, along with another value called recovery identifier `(v)`, are crucial to the signature verification process.
    - last one byte — `v`
    - first 32 bytes — `r`
    - second 32 bytes — `s`

`v` is the last byte, while `r` and `s` is simply the signature split in half. The `v` is the recovery ID, which can have two values: `0` or `1`, corresponding to two sides of the y-axis.

By knowing this couple of this (and as far as I understood), you will have problems if the only thing you check to verify the signature is the `r` and `s` values. This is because the `v` value can be manipulated to make the signature valid because of the symetrical nature of the [curve](https://user-images.githubusercontent.com/35583758/229375008-254586ef-177c-4111-9aa6-60d42ff6a251.png).

Nowadays OpenZeppelin has a library that will ensure you are not vulnerable to this attack vector.

# Oracle Price Manipulation

I didn't find this one very interesting because I don't think it is that feasible nowadays, there are multiple solutions to this and the most basic and the one that we have been using for a while is to use a [Chainlink](https://chain.link/) oracle, which solves this by using a decentralized oracle network.

Basically consists of a malicious actor that will manipulate the price of an oracle to perform a favorable trade on certain token pair.

# Controlled DelegateCall

This one was my favourite because I have been tinkering around with calldata lately, and this attack vector exactly shows you that by crafting malicious calldata you can exploit a contracts functionality.

This attack could lead to:
- Unauthorized execution of functions, potentially leading to theft of funds, manipulation of contract state, or other unintended consequences.
- Complete contract takeover, giving the attacker control over the contract's functionality and stored assets.

### Talk is cheap, show me the code

```javascript
pragma solidity ^0.8.0;

contract DelegateCallProxy {
    receive() external payable {}

    function execute(address _target, bytes memory _data) public payable {
        (bool success, ) = _target.delegatecall(_data);
        require(success, "DelegateCallProxy: call failed");
    }
}
```
If you see something like this anywhere, first of all, feel sorry for the dev that will get fired, secondly, create a POC that show how you exploit this and contact the dev team.

Our attack contract will look like this:

```javascript
pragma solidity ^0.8.0;

contract Attack {
    address payable public owner;

    constructor() {
        owner = payable(msg.sender);
    }

    function attack(bytes memory _data) external payable {
        // Attack logic
    }

    function stealFunds() external payable {
        owner.transfer(address(this).balance);
    }
}
```

The steps will be the follwing:

1. The attacker sends 100 ether to the DelegateCallProxy contract.
2. The attacker deploys the Attacker contract to the network.
3. The attacker crafts a malicious payload to call the stealEther function of the Attacker contract using delegate call. This can be done by encoding the function signature.
4. The attacker calls the execute function of the DelegateCallProxy contract, passing the address of the Attacker contract and the malicious payload as arguments.
5. The DelegateCallProxy contract performs a delegate call to the Attacker contract, executing the stealEther function in the context of the DelegateCallProxy contract's storage.
6. The stealEther function transfers the DelegateCallProxy contract's balance (100 ether) to the attacker's address.

If you want to know how to perform step 3, you can check my previous post about [Calldata & Foundry](https://blog.mariodev.xyz/day04/)

## Wrapping up

I hope you've learned something in this post.

If you didn't already, I highly recommend to join the [@opensensepw](https://twitter.com/opensensepw) discord server, they are a great community and they are always willing to help, plus you will have the opotunity to also join this study sessions.