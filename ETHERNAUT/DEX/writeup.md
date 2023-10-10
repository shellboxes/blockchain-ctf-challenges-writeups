**Description**

In this challenge, we are required to drain the contract from at least one token. The contract is a basic `DEX` where the value of each token is determined by the ratio of the balance of the contract's balance for each token. This basic formula will allow us to drain the contract's funds. This will be done by increasing the amount of tokenA while holding tokenB, which will drop the value of tokenA compared to tokenB causing the value of tokenB to rise, which we already own.Let's take a look at the code.

* **Dex.sol**
```solidity=
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "openzeppelin-contracts-08/token/ERC20/IERC20.sol";
import "openzeppelin-contracts-08/token/ERC20/ERC20.sol";
import 'openzeppelin-contracts-08/access/Ownable.sol';

contract Dex is Ownable {
  address public token1;
  address public token2;
  constructor() {}

  function setTokens(address _token1, address _token2) public onlyOwner {
    token1 = _token1;
    token2 = _token2;
  }

  function addLiquidity(address token_address, uint amount) public onlyOwner {
    IERC20(token_address).transferFrom(msg.sender, address(this), amount);
  }

  function swap(address from, address to, uint amount) public {
    require((from == token1 && to == token2) || (from == token2 && to == token1), "Invalid tokens");
    require(IERC20(from).balanceOf(msg.sender) >= amount, "Not enough to swap");
    uint swapAmount = getSwapPrice(from, to, amount);
    IERC20(from).transferFrom(msg.sender, address(this), amount);
    IERC20(to).approve(address(this), swapAmount);
    IERC20(to).transferFrom(address(this), msg.sender, swapAmount);
  }

  function getSwapPrice(address from, address to, uint amount) public view returns(uint){
    return((amount * IERC20(to).balanceOf(address(this)))/IERC20(from).balanceOf(address(this)));
  }

  function approve(address spender, uint amount) public {
    SwappableToken(token1).approve(msg.sender, spender, amount);
    SwappableToken(token2).approve(msg.sender, spender, amount);
  }

  function balanceOf(address token, address account) public view returns (uint){
    return IERC20(token).balanceOf(account);
  }
}

contract SwappableToken is ERC20 {
  address private _dex;
  constructor(address dexInstance, string memory name, string memory symbol, uint256 initialSupply) ERC20(name, symbol) {
        _mint(msg.sender, initialSupply);
        _dex = dexInstance;
  }

  function approve(address owner, address spender, uint256 amount) public {
    require(owner != _dex, "InvalidApprover");
    super._approve(owner, spender, amount);
  }
}
```

The challenge starts like this, the player owns 10 token1 and 10 token2 , the balance of the contract is 100 token1 and 100 token2. With this knowledge let's see what happens when we execute a simple swap.

Let's say I want to swap all 10 of my tokens for token2, the execution will go like this:

1- calling the swap function
2- The swap amount will be calculated as follows : `(amount * token2.balanceOf(this contract))/token1.balanceOf(this contract)`. In our case, it will be 10 since both the balances are equal.
3- As a result, the user will have 0 tokens 1 and 20 tokens 2, while the contract will have 110 tokens 1 and 90 tokens 2, indicating that the swap ratio for the t1->t2 swap is 0.8 and 1.2 for the t2->t1 swap.
4- The user swaps the extra 10 t2 for t1 and they will get 12 t1 and 10 t2, and for the contract, 98 t1 and 100 t2.

Theese four operations cause an increase in the t1 value of 0.02 % and a decrease of 0.02 % in the t2 value.

The following table shows how the value is changed for each swap.
(CBTn : contract balance of token n)
(Tk->Tn : the swap price of token k for token n)
(UBTn : the user balance of token n)


| CBT1 | CBT2 | T1->T2 | T2->T1 | UBT1 | UBT2 |
| :--: | :--: | :----: | :----: | :--: | :--: |
| 100  | 100  |   1    |   1    |  10  |  10  |
| 110  |  90  |  0.8   |  1.2   |  0   |  20  |
|  98  | 100  |  1.02  |  0.98  |  12  |  10  |
| 108  |  90  |  0.8   |  1.2   |  2   |  20  |
|  96  | 100  |  1.04  |  0.96  |  14  |  10  |
| ...  | ...  |  ...   |  ...   | ...  | ...  |

The margin between the amounts of the two tokens available in this contract will keep growing, and each swap will be more effective with more margin and bigger amounts.

**Recommendatiom**

It is recommended to use the  Constant Product `X * Y = K` formulas to avoid running out of liquidity.