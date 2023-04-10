---
date: 2023-04-10T04:55:09-04:00
draft: false
tags: [tips]
title: "Day03 - Smart Contract Auditing Heuristics"
---

I found [this](https://github.com/OpenCoreCH/smart-contract-auditing-heuristics) cool repo on Github that has a list of heuristics for auditing smart contracts. I thought it would be a good idea to go through them and see if I can find more interesting.

# Code asymmetries

---
In many projects, there should be some symmetries for different functions.

For instance, a `withdraw` function should (usually) undo all the state changes of a `deposit` function and a `delete` function should undo all the state changes of the corresponding `add` function.

Asymmetries in these function pairs (e.g., forgetting to unset a field or to subtract from a value) can often lead to undesired behavior.

---

This one might seem obvious, and it is, but the idea of this repo doesn't seem to be to give you the key to the audit kingdom, but to give you a list of things to look for on every project, because they are things that might be overlooked and are actually quite common.

# Off-by-one errors

---
Off-by-one errors are very common (not only in smart contracts), so it always makes sense to think about the boundaries. Is <= correct in this context or should < be used? Should a variable be set to the length of a list or the length - 1? Should an iteration start at 1 or 0?

---

I liked this one in particular because almost the same reason as the first one, it is something so easily overlooked, and a mistake that even the most experienced programmers make.

# Not following EIPs / Standards

---
Ethereum Improvement Proposals and in general standards (like RFCs) often have very detailed instructions on how different functions/implementations should behave in different scenarios. Not following these is very bad for composability (after all, the reason for these standards is that you can rely on a certain behavior) and it can also result in vulnerabilities. In my experience, there are often slight details (e.g., not reverting when one should revert or not following the specified rounding behavior) that are ignored by implementers.

---

Well, this one may be familiar if you read my previous [post](https://blog.mariodev.xyz/day02/) ;)