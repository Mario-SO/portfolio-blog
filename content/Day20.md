---
date: 2023-04-27T14:49:25-04:00
draft: false
tags: [audit]
title: "Day20 - Adversarial thinking"
---

Today, the auditing team and I got started with the [Code4rena](https://code4rena.com/) audit for [EigenLayer](https://code4rena.com/contests/2023-04-eigenlayer-contest) and even though it is not live yet (couple of hour remaining), we started to take a look at their public repo to see what we can encounter.

# The team is key

I'm super grateful for the opportunity to be part of this team, I'm learning a lot from them, and I'm sure I will learn a lot more in the future, they are really welcoming for newcomers like me, and don't hesitate to answer any question I might have and share their knowledge with me.

What I wanted to share today is the thought process that a really skillful guy from the team shared and that it seems to have unlocked a new way of thinking for me too.

# Simple but incredible advice

So this guy was looking at a piece of code, to be precise, this piece of code:

```javascript
if (totalShares == 0) {
    newShares = amount;
} else {
    uint256 priorTokenBalance = _tokenBalance() - amount;
    if (priorTokenBalance == 0) {
        newShares = amount;
    } else {
        newShares = (amount * totalShares) / priorTokenBalance;
    }
}
```

What was his thought process? Well, pretty simple if you think about it, but until you dont have that *click* in your head, you won't be able to think like that.

*Almost* literal words from him:

"We have `*` and `/` operators, those are specially vulnarable to `0` values, what can happen if either `totalShares` or `priorTokenBalance` are `0`?, and **HOW CAN WE GET THE SMART CONTRACT TO THAT STATE?**"

This was what clicked for me, see what may break the contract (even if at first sight it seems impossible), and then, how can we get the contract to that state, that's what adversarial thinking is all about.

## Wrapping up

Hope you also got the *click* in your head, and if you didn't, don't worry, it will come to you eventually, just keep pushing.

Thanks for reading!