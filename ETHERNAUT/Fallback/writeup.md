**Description**
The goal of this challenge is to claim the ownership then drain the contract's balance.

* Fallback.sol
```solidity=
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Fallback {

  mapping(address => uint) public contributions;
  address public owner;

  constructor() {
    owner = msg.sender;
    contributions[msg.sender] = 1000 * (1 ether);
  }

  modifier onlyOwner {
        require(
            msg.sender == owner,
            "caller is not the owner"
        );
        _;
    }

  function contribute() public payable {
    require(msg.value < 0.001 ether);
    contributions[msg.sender] += msg.value;
    if(contributions[msg.sender] > contributions[owner]) {
      owner = msg.sender;
    }
  }

  function getContribution() public view returns (uint) {
    return contributions[msg.sender];
  }

  function withdraw() public onlyOwner {
    payable(owner).transfer(address(this).balance);
  }

  receive() external payable {
    require(msg.value > 0 && contributions[msg.sender] > 0);
    owner = msg.sender;
  }
}
```

**Attack Vector**
Unlike the `contribute` function, the `receive` function does not implement any limitations on the amount that can be deposited which makes it possible to be the new owner by contributing a smaller amount than the owner's contribution. 

**Attack Scenario**
The attacker can become the owner of the contract by contributing more than the original owner and surpassing their total contributions. But using the `contribute` function, the attacker can send a transaction with a value greater than 0 and smaller than `0.001 ether`, in order to have a contribution, then the attacker can call the `receive` function to claim the ownership. Bieng the owner, the attacker can easily drain the contract by calling the `withdraw` function.

**Recommendation**
Consider adapting the `receive` function match the logic of the `contribute` function.