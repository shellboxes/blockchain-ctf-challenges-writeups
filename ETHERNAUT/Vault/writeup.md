**Description**
The goal of this challenge is to unlock the vault, which looks like it is locked using an unknown password.

* Vault.sol
```
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Vault {
  bool public locked;
  bytes32 private password;

  constructor(bytes32 _password) {
    locked = true;
    password = _password;
  }

  function unlock(bytes32 _password) public {
    if (password == _password) {
      locked = false;
    }
  }
}
```

**Attack Vector**
The issue in this contract is that the constructor assigns the instance's "password" state variable. This means that anyone can access the variable storage or the transaction history can also access the password.

**Attack Scenario**
The password can be accessed easily in the first transaction of the contract that calls the `constructor` or by extracting the value from the storage.

- First, we need the contract's address: `contract.address`
- We can retrieve the password from the contract storage: `web3.utils.getStorage('[contract's address]', 1)`
  `1` is the index of the slot where the password is stored. for more in depth details about Storage in solidity, check out this resource.
  https://mixbytes.io/blog/collisions-solidity-storage-layouts#:~:text=Solidity%20does%20not%20possess%20a,and%20unpacked%20on%20the%20fly.

- Now, we should have a Hash which we can pass to the unlock function: `contract.unlock('[hash]')`

- Submit the instance and now you passed the challenge !

**Recommendation**
Never store sensitive data like passwords in storage as blockchain is public and transparent, which means everybody will be able to see that data.