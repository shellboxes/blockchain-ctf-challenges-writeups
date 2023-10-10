**Description**
We are given 20 tokens to start with and the goal of the challenge is to somehow get our hands on any additional tokens.

* Token.sol
```solidity=
// SPDX-License-Identifier: MIT
pragma solidity ^0.6.0;

contract Token {

  mapping(address => uint) balances;
  uint public totalSupply;

  constructor(uint _initialSupply) public {
    balances[msg.sender] = totalSupply = _initialSupply;
  }

  function transfer(address _to, uint _value) public returns (bool) {
    require(balances[msg.sender] - _value >= 0);
    balances[msg.sender] -= _value;
    balances[_to] += _value;
    return true;
  }

  function balanceOf(address _owner) public view returns (uint balance) {
    return balances[_owner];
  }
}
```

**Attack Vector**
Integer underflow in Solidity is a type of attack that occurs when an integer value is subtracted below 0, resulting in unexpected and potentially malicious behavior. This kind of attack can be used to execute malicious functions, drain funds, or even create infinite values out of thin air. Such attacks can be prevented by adding additional checks to the code when subtracting integers.

**Attack Scenario**
The attacker can simply transfer 21 tokens (which is more than 20) to bypass the require statement and underflow the value stored in the `balances` mapping.

* Let's check our balance:

`await contract.balanceOf(player)`
it should return 20.

* Now we can initiate a transfer to an address as the target and '21' as value. we can simply transfer the value to the `address(0)`.

* This will cause the underflow and now we can check out balance again. 

`await contract.balanceOf(player)`

* Now we have a huge number of tokens.


**Recommendation**
In order to prevent integer underflow in Solidity, it is recommended to perform additional checks when dealing with integers, such as ensuring that subtractions do not decrease a value to below 0. It is also advisable to add functions that check for overflows as well as underflows. Finally, where possible, use the `SafeMath` library provided by OpenZeppelin to ensure that integer operations do not result in unexpected behavior.