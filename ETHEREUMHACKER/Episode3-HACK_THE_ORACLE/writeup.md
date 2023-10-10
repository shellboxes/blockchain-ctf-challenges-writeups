**Description**

This challenge has a contract with the name `CBDC` which stands for `Central Bank Digital Currency`, This contract uses oracles to take actions such as updating the price and printing money (minting in our terms). The challenge here is to try and gain oracle privillege, let's see if we can do that.

- **CBDC.sol**

```solidity=
// SPDX-License-Identifier: GPL-3.0
pragma solidity ^0.8.0;

import "@openzeppelin/contracts/token/ERC20/ERC20.sol";

contract CBDC is ERC20 {
    uint256 public initialSupply = 1000000000000 ether; // 1 trillion & 18 decimals
    uint256 public inflation = 100000000 ether;
    address public centralBank;
    uint256 public usdPrice = 100000;
    mapping(address => bool) public oracles;

    constructor() ERC20("Central Bank Digital Currency", "CBDC") {
      _mint(msg.sender, initialSupply);
      centralBank = msg.sender;
    }

    function addOracle(string calldata _secret) public {
        // https://twitter.com/CentralBankDigi
        bytes32 answer = 0x70b00831d459e9f2e4b22b203fbdfd5d1830d5c16a36579bcbb12f91de2159e9;
        bytes32 yourAnswer = keccak256(abi.encode(_secret));
        require(yourAnswer == answer, "You Will Never Break In");
        oracles[msg.sender] = true;
    }

    function testAnswer(string calldata _secret) public pure returns (bytes32) {
        bytes32 yourAnswer = keccak256(abi.encode(_secret));
        return yourAnswer;
    }

    function isOracle(address _checkAddress) public view returns (bool) {
        return oracles[_checkAddress];
    }

    function printMoney() public {
        require(oracles[msg.sender] == true, "Only The Elite Can Print Money");
        _mint(centralBank, inflation);
    }

    function updatePrice(bytes32 _blockHash, uint256 _usdPrice) public {
        require(oracles[msg.sender] == true, "Only The Elite Can Manipulate Markets");
        uint256 blockNumber = block.number - 1;
        bytes32 blockHash = blockhash(blockNumber);
        require(blockHash == _blockHash, "Only Smart Contracts Can Manipulate Markets");
        usdPrice = _usdPrice;
    }

}
```

**Attack Vector**

The contract has an `addOracle()` function that basically adds an address as an oracle. The function has an argument of type `string` called `_secret`. This is then encoded using `abi.encode256` function to hash, and it compares the hash with their hash, which is the secret word to get Oracle privilege. Bruteforcing is obviously not an option, so we must look elsewhere. 

**Attack Scenario**
In the code at line 19, there is a comment with a link to a Twitter post, and clicking on that link redirects to a post with a riddle that says:> I have branches but no leaves. What am I?
I am terrible at solving riddles, so I just googled the answer, which was `bank`, I checked using the `testAnswer` function, and it was a match. So I just called the `addOracle` with the answer, and voila! I got the Oracle privilege.

**Recommendation**
First and foremost, before deploying any project with publicly accessible code, we should always check to see if there are any sensitive information left in comments or even variable names, and the obvious recommendation would be to remove the comment. The best recommendation would be to avoid using a secret word that is shared by multiple parties; instead, use an access control on the function such that only the owner or an admin can add oracles to the contract. Here is the fixed version of the contract:
- **CBDC.sol**
```solidity=
// SPDX-License-Identifier: GPL-3.0
pragma solidity ^0.8.0;

import "@openzeppelin/contracts/token/ERC20/ERC20.sol";
import "@openzeppelin/contracts/access/Ownable.sol";

contract CBDC is ERC20 , Ownable {
    uint256 public initialSupply = 1000000000000 ether; // 1 trillion & 18 decimals
    uint256 public inflation = 100000000 ether;
    address public centralBank;
    uint256 public usdPrice = 100000;
    mapping(address => bool) public oracles;

    constructor() ERC20("Central Bank Digital Currency", "CBDC") {
      _mint(msg.sender, initialSupply);
      centralBank = msg.sender;
    }

    function addOracle(string calldata _secret) public onlyOwner {
        oracles[msg.sender] = true;
    }

    function isOracle(address _checkAddress) public view returns (bool) {
        return oracles[_checkAddress];
    }

    function printMoney() public {
        require(oracles[msg.sender] == true, "Only The Elite Can Print Money");
        _mint(centralBank, inflation);
    }

    function updatePrice(bytes32 _blockHash, uint256 _usdPrice) public {
        require(oracles[msg.sender] == true, "Only The Elite Can Manipulate Markets");
        uint256 blockNumber = block.number - 1;
        bytes32 blockHash = blockhash(blockNumber);
        require(blockHash == _blockHash, "Only Smart Contracts Can Manipulate Markets");
        usdPrice = _usdPrice;
    }

}
```