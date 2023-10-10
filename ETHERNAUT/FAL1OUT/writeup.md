**Description**
The goal of this challenge is to claim the ownership of th contract.

* Fal1out.sol
```solidity=
// SPDX-License-Identifier: MIT
pragma solidity ^0.6.0;

import 'openzeppelin-contracts-06/math/SafeMath.sol';

contract Fallout {
  
  using SafeMath for uint256;
  mapping (address => uint) allocations;
  address payable public owner;


  /* constructor */
  function Fal1out() public payable {
    owner = msg.sender;
    allocations[owner] = msg.value;
  }

  modifier onlyOwner {
	        require(
	            msg.sender == owner,
	            "caller is not the owner"
	        );
	        _;
	    }

  function allocate() public payable {
    allocations[msg.sender] = allocations[msg.sender].add(msg.value);
  }

  function sendAllocation(address payable allocator) public {
    require(allocations[allocator] > 0);
    allocator.transfer(allocations[allocator]);
  }

  function collectAllocations() public onlyOwner {
    msg.sender.transfer(address(this).balance);
  }

  function allocatorBalance(address allocator) public view returns (uint) {
    return allocations[allocator];
  }
}

```
**Attack Vector**

In line 13, The comment indicates that the below code is expected to be a aonstructor, However, the `constructor` keyword was not used but it was called as a `public` function instead which anybody can call.

**Attack Scenario**
The attacker can simply call the `Fal1out` function and they will take ownership of the contract.


**Recommendation**
It is recommended to use the `constructor` keyword to only allow the function to be executed once by the deployer.