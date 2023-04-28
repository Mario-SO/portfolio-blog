---
date: 2023-04-26T13:35:47-04:00
draft: false
tags: [audit]
title: "Day19 - 1 wei attack"
---

The learning is starting to flourish XD. As I told you in a previous post, I joined a team of people for auditing and I'm learning a shit ton of stuff, which of course, will also share with everyone reading this posts of mine hehe.

# ELI5 1 wei attack

First of all this is also known as **inflation attack on EIP-4626**.

In simple terms, the inflation attack on EIP-4626 is an exploit where an attacker manipulates the exchange rate between a vault's shares and its underlying assets, causing a victim to lose their deposited assets.

This is made possible due to the lack of slippage protection in the deposit function when a vault has low liquidity.

Here's a step-by-step illustration of the attack:

1. Alice wants to deposit 1 token into the vault.
2. The vault is initially empty, and the default exchange rate is set (usually 1 share per asset).
3. Bob, the attacker, sees Alice's transaction in the mempool (a pool of pending transactions) and decides to sandwich it.
4. Bob first deposits a small amount (1 wei) into the vault and receives an equivalent amount of shares.
5. The exchange rate remains 1 share per asset.
6. Bob then transfers 1 token directly to the vault using an ERC-20 transfer, without receiving any shares.
7. The exchange rate now increases, and Alice's 1 token deposit is worth much less in shares.
8. Alice's deposit is executed, but she receives very few or no shares in return, essentially donating her tokens to the vault.
9. Finally, Bob redeems his small number of shares and receives all the assets in the vault, including his own deposits and Alice's tokens.

# How to prevent it?

The proposed mitigation for this attack is to introduce a "decimal offset" along with "virtual assets and shares" to better define the exchange rate, even when the vault is empty.

This would make it more difficult for an attacker to manipulate the exchange rate and would require a significantly larger donation to inflate it.