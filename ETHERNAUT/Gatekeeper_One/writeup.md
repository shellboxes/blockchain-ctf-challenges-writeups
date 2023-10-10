**Description**

This challenge is about bypassing some gates, as the name might have stated. The goal of this challenge is to set the entrant address to our own address, but there are three gates hidden in the form of modifiers. We are going to walk through the first and third ones since the second one requires a bit of calculation or brute force, but it's not really that hard. Before getting started, we recommend that you have an understanding of the EVM memory layout and type conversion, read [this](https://betterprogramming.pub/solidity-tutorial-all-about-conversion-661130eb8bec).
Let's take a look at the contract's code:

```solidity=
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract GatekeeperOne {

  address public entrant;

  modifier gateOne() {
    require(msg.sender != tx.origin);
    _;
  }

  modifier gateTwo() {
    require(gasleft() % 8191 == 0);
    _;
  }

  modifier gateThree(bytes8 _gateKey) {
      require(uint32(uint64(_gateKey)) == uint16(uint64(_gateKey)), "GatekeeperOne: invalid gateThree part one");
      require(uint32(uint64(_gateKey)) != uint64(_gateKey), "GatekeeperOne: invalid gateThree part two");
      require(uint32(uint64(_gateKey)) == uint16(uint160(tx.origin)), "GatekeeperOne: invalid gateThree part three");
    _;
  }

  function enter(bytes8 _gateKey) public gateOne gateTwo gateThree(_gateKey) returns (bool) {
    entrant = tx.origin;
    return true;
  }
}
```
**Attack Scenario**
- **gateOne()** as we can see, the `gateOne()` modifier is easy to bypass since it only requires the `msg.sender` to be different than the `tx.origin`. This is easy since you can just call enter from another contract, and it is bypassed.

- **gateTwo()** This one is a little more difficult because it requires you to send an exact amount of gas that is a multiple of 8191; this is difficult because you can calculate the amount of gas required to reach that line of code starting from your contract, but it will still not be the exact amount needed because the gasprice is not stable enough to validate your transactions, which means the only way to solve this is to find a range that holds the approximate value that satisfies that condition and try with that.