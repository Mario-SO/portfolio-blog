---
date: 2023-04-19T12:06:41-04:00
draft: false
tags: [audit]
title: "Day12 - Weponizing Gas"
---

Today, reading through some past reports on [Solodit](https://solodit.xyz/) I found out about this interesting exploit that is known under the name of *1/64 rule*.

The *EVM* limits the total gas forwarded on to 63/64ths of the total `gasleft()` (and silently reduces it to this value if we try to send more)

# Example

This is the report from [Sherlock](https://app.sherlock.xyz/audits/contests/38) that I was reading.

I still don't fully understand how this works though, but I'll try to explain it as best as I can.

### The exploit

Something interesting I didn't know about **Optimism**:

---

"In `OptimismPortal` there is no replaying of transactions. If a transaction fails, it will simply fail, and all ETH associated with it will remain in the `OptimismPortal` contract.

Users have been warned of this and understand the risks, so **Optimism** takes no responsibility for user error.

In order to ensure that failed transactions can only happen at the fault of the user, the contract implements a check to ensure that the `gasLimit` is sufficient"

```javascript
require(
    gasleft() >= _tx.gasLimit + FINALIZE_GAS_BUFFER,
    "OptimismPortal: insufficient gas to finalize withdrawal"
);
```
---

The problem here is that the *EVM* has a specification that limits the amount of gas that can be sent to an external function call. This limit is set to **63/64ths** of the total `gas_left()` at that point in the contract execution.

However, when the gas limit is very large, the remaining **1/64th** of gas can be greater than a pre-defined value called the *FINALIZE_GAS_BUFFER*. When this happens, the amount of gas forwarded to the external function call would be less than what was actually specified by the contract.

As a side note, this is one really good use case for *fuzz-testing*.

## Wrapping up

If you want to see the *POC* of this exploit, you can check it out in the [Sherlock](https://app.sherlock.xyz/audits/contests/38) report. It is quite long but really interesting (and the report is using foundry ;D )