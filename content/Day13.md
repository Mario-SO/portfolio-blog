---
date: 2023-04-20T08:51:40-04:00
draft: false
tags: [huff, key 💡]
title: "Day13 - Huff, learning & bonuses"
---

Yooooo, first of all I want to thank anyone reading this, it keeps me motivated knowing that there is someone out there reading my posts, so thank you!

Shoutout to [@0xMackenzieM](https://twitter.com/0xMackenzieM) who talked to me via DM giving me some feedback on my approach with this blog. If I ever get nearly half on the knowledge he has, don't doubt a single second that I will pay it forward.

Now to the [huff](https://huff.sh/) stuff.

# Huff rocks

Finally I found out a repo with huff challenges [here](https://github.com/RareSkills/huff-puzzles)

I will skip the first *"challenge"* because it is not even a challenge, it is just a tutorial on how to test the rest of the challenges, so I will start with the **CallValue** challenge.

# The challenge

The task is to write within the `MAIN` macro, huff code to get and return the amount of ether sent as part of that call.

Basically, return the `msg.value` we have in solidity but using huff.

# The solution

When first looking at huff, I thought their docs where going to be my very best friend, but I was wrong, the real docs that you want to suck yourslef into are the *EVM opcodes* docs. (I mean, their docs are helpful, but you must first dig into some opcode stuff to feel a bit more confortable with huff)

### Talk is really cheap, show me the code

```javascript
#define macro MAIN() = takes(0) returns(0) {
    // store CALLVALUE in memory at offset 0
    callvalue   // [CALLVALUE]
    0x00        // [0x00, CALLVALUE]
    mstore      // []

    // return 32 bytes of memory starting at offset 0
    0x32        // [0x32]
    0x00        // [0x00, 0x32]
    return      // []
}
```

Not really that difficult (if you know how the *EVM* works) even comments explain the code really clearly, so I will be doing the third one as a bonus.

---

## Bonus[0]

Won't be doing the second one becuase is basically the same as the first one, so will jump to the third one because it is the one that I made the first mistake, a silly one, but really important and fundamental.

The task is to write within the `MAIN` macro below, huff code that retrieves the ether balance of the address that sent the transaction, also known as msg.sender in solidity.

```javascript
#define macro MAIN() = takes(0) returns(0) {
    // Store the balance of the sender in memory at offset 0
    caller          // [CALLER]
    balance         // [BALANCE, CALLER] 
    0x00            // [0x00, BALANCE, CALLER]
    mstore          // []

    // return 64 bytes of memory starting at offset 0
    0x40        // [0x40]
    0x00        // [0x00, 0x40]
    return      // []  
}
```

This is the correct solution, but it is more interesting to see the mistake I made.

```javascript
#define macro MAIN() = takes(0) returns(0) {
    // Store the sender in memory at offset 0
    caller          // [CALLER]
    0x00            // [0x00, CALLER]
    mstore          // []

    // Store the balance in memory at offset 1
    balance         // [BALANCE]
    0x01            // [0x01, BALANCE]
    mstore          // []

    // return 64 bytes of memory starting at offset 0
    0x40        // [0x40]
    0x00        // [0x00, 0x40]
    return      // []  
}
```

This was my first approach, but the thing I was not getting here is the `offset`. I was seeing it as array indexes and they, first of all, are `hex` values and, second of all, they are `bytes` not `indexes`, each variable has its own size, so the `offset` is the number of bytes that you want to skip, not the index of some imaginary stupid array.

I was really mixing concepts here, why the `0x40` made sense in my head, but the `0x00` and `0x01` didn't? No idea to be honest. Learning consists on this.

1. You think you know something
2. Then you don't
3. Then you learn it
4. Back to step 1

I was suppose to already know this if you read some of my past posts, but as you can see, until you don't actually get your hands dirty, you don't really learn.

It is also really useful to get familiar with counting in hex, because you will be doing it a lot, and will save you some headhaches.

## Bonus[1]

Today with the help of ChatGPT I made [this](https://twitter.com/mariodev__/status/1649001656762220546?s=20)

And as promised, here is the [code](https://gist.github.com/Mario-SO/409ca4b38692741420ba1793423c7435)

## Bonus[2]

This bonus is useless, but I realized once GPT already spit out the last prompt result so yeah... I can conclude from this that AI is so good and quick, that I couldn't even take the time to think the usefulness of this script hehe.

Here is the tweet where I'm happy but not happy -> [tweet](https://twitter.com/mariodev__/status/1649062765984874549?s=20)

And the [code](https://gist.github.com/Mario-SO/c988b26e4c4f6e24228bac6d5591d940)

## Wrapping up

Well what a day, I did some really useless stuff, learned some huff, and reinforced some knowledge too.

As you could see today, even things that you are supose to know, will take time until you actually internalize them, so don't be afraid to make mistakes, because you will learn from them.