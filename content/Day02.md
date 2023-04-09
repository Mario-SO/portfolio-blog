---
date: 2023-04-09T05:49:23-04:00
draft: false
tags: [audit]
title: "Day02 - People don't read standards"
---

Today I went on and after doing some [Ethernaut](https://ethernaut.openzeppelin.com/) challenges, I decided to try and go through open challenges on [Code4rena](https://code4rena.com/).

Prepared my environment, opened [filter4rena](https://filter4rena.vercel.app/), and started looking for some vulns.

# First finding

When starting to look at the contracts I quickly found this [line of code](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/Factory.sol#L152) that caught my attention:

```solidity
ERC20(token).transfer(msg.sender, amount);
```
Even though this was my first ever *real* audit, I have been reading a lot of blog posts and common vulnerabilities, so I knew that this line of code was considered a vulnerability because it doesn't check the return value of the `transfer` function, and some tokens do not `revert` when the transfer fails but instead return `false`.

It is really easy to fix, just use the `safeTransfer` function from openzeppelin lib:

```solidity
ERC20(token).safeTransfer(msg.sender, amount);
```

Same thing happens with the `transferFrom` function.

This is now considered a *Low severity* vulnerability, but not so long ago it was considered a *Medium severity* vulnerability, see [this](https://code4rena.com/reports/2021-09-yaxis/#m-02-erc20-return-values-not-checked) audit report from [@cmichelio](https://twitter.com/cmichelio)

# Familiarize with standards

Did you know that *$USDT* doesn't follow the ERC20 standard? It is a bit weird, but it is true. Therefore, using *$USDT* as the `token` parameter, will result in a `revert`.

## The catch

Hehe, you might be thinking: *"Damn, first day and I already found a vulnerability, this guy knows what he is doing"*

The real thing is that no, I have no Idea what I am doing, I'm just lurking around and reading stuff, but I'm still not able to craft a proper *POC* so my scope is really on *QA* and *Low severity* vulnerabilities.

In fact **(here is the catch)** this is such a simple and easy to spot vulnerability that the automatic *4nalyzer* from *Code4rena* already found it, not only this one but even TWO MEDIUM SEVERITY VULNERABILITIES, and they immediately become common knowledge and are not accountable for reward.

Pretty crappy that this happened while I was just about to submit my first *low severity*.

Some people even think that the *4nalyzer* could be next top auditor 😅
