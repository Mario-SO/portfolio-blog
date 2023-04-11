---
date: 2023-04-10T04:55:09-04:00
draft: false
tags: [tips]
title: "Day03 - Smart Contract Auditing Heuristics"
---

I found [this](https://github.com/OpenCoreCH/smart-contract-auditing-heuristics) cool repo on Github that has a list of heuristics for auditing smart contracts. I thought it would be a good idea to go through the ones I found interesting.

# Code asymmetries

---
In many projects, there should be some symmetries for different functions.

For instance, a `withdraw` function should (usually) undo all the state changes of a `deposit` function and a `delete` function should undo all the state changes of the corresponding `add` function.

Asymmetries in these function pairs (e.g., forgetting to unset a field or to subtract from a value) can often lead to undesired behavior.

---

This one might seem obvious, and it is, but the idea of this repo doesn't seem to be to give you the key to the audit kingdom, but to give you a list of things to look for on every project, because they are things that might be **overlooked** and are actually quite **common**.

# Off-by-one errors

---
Off-by-one errors are very common (not only in smart contracts), so it always makes sense to think about the boundaries. Is `<=` correct in this context or should `<` be used? Should a variable be set to the length of a list or the `length - 1`? Should an iteration start at `1` or `0`?

---

I liked this one in particular for almost the same reason as the first one, it is something so easily overlooked, and a mistake that even the most experienced programmers make, so even if you don't find any particular vulnerability, you can still help the developer by pointing out these things.

# Not following EIPs / Standards

---
*Ethereum Improvement Proposals* and in general standards (like RFCs) often have very detailed instructions on how different functions/implementations should behave in different scenarios. Not following these is very bad for **composability** (after all, the reason for these standards is that you can rely on a certain behavior) and it can also result in vulnerabilities. In my experience, there are often slight details (e.g., not reverting when one should revert or not following the specified rounding behavior) that are ignored by implementers.

---

Well, this one may be familiar if you read my previous [post](https://blog.mariodev.xyz/day02/) ;)

# Behavior when src == dst

---
In a lot of smart contracts, there are functions where you have to specify a source (or sender) and a destination (or recipient). Sometimes, the programmer did not think about what happens when these (user-specified) parameters are equal, which can result in undesired behavior. For instance, the balance of the source and destination may be cached in the beginning, in which case you can inflate your balance by specifying yourself as the source and the destination.

---

I just finished watching an interview [100proof - Receiving a 150k Bug Bounty, Web3 Bounty Hunting and Smart Contract Auditing](https://youtu.be/NEmwfl-zLuw) and while writing this post and listening to it in the background 100proof mentioned this exact same thing.

This just assures me that this is a very good thing to test for, maybe not specifically for finding a vulnerability, but to make sure that the code is behaving as expected.

## Wrapping up

I think this is a very good list of things to look for when auditing a smart contract, and I will definitely be using it in the future. I hope you found it useful as well.

I intended to write some code today, but I had some work to do on my *job*, so I didn't get to it. I will try to write some code asap and show some POCs, this will also serve as a good way to improve my POC skills.