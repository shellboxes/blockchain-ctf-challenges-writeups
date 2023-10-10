**Description**

This challenge is really interesting because it shows the risk of relying on vulnerable code. Recall from EP.3 that we had a vulnerable security check that used a secret word to accept a transaction. Now this challenge, like EP.4, is just showing you how far you can go. We are going to use the vulnerable `addOracle()` function to gain Oracle privilege access to withdraw funds from a multisig wallet. On top of the CBDC contract, we have `Multisig` contract, which uses the CBDC token.


- **MultiSig.sol**
```solidity=
// SPDX-License-Identifier: GPL-3.0
pragma solidity ^0.8.0;

import "@openzeppelin/contracts/token/ERC20/IERC20.sol";

interface CBDC is IERC20 {
  function addOracle(string calldata _secret) external;
  function isOracle(address _checkAddress) external view returns (bool);
}

contract MultiSig {
    address public cbdc;
    address public centralBank;
    address public usdc = 0x2f3A40A3db8a7e3D09B0adfEfbCe4f6F81927557;
    address[] signaturies;
    mapping(address => bool) public signatures;

    constructor (address _cbdc) {
        cbdc = _cbdc;
        centralBank = msg.sender;
        signaturies.push(msg.sender);
    }

    function upgradeUSDC(address _usdc) public {
        require(msg.sender == centralBank, "Only The Bank Can Change The USDC Token Address");
        usdc = _usdc;
    }

    function signWithdrawal() public {
        signatures[msg.sender] = true;
    }

    function withdrawFunds() public {
        for (uint256 i=0; i<signaturies.length; i++) {
            address signer = signaturies[i];
            require(signatures[signer] == true, "Not Everyone Has Signed Off On This");
        }
        IERC20(cbdc).transfer(msg.sender,100000);
    }

    function buyFundsPublic() public {
        IERC20(usdc).transferFrom(msg.sender,address(this), 1000000000000);
        IERC20(cbdc).transfer(msg.sender,1);
    }

    function updateCentralBank(address _newBank) public {
        bool oracle = CBDC(cbdc).isOracle(_newBank);
        require(oracle == true, "You Are Not An Authorized Oracle");
        centralBank = _newBank;
    }

    function addSignature(address _newSig) public {
        require(msg.sender == centralBank, "Only The Bank Can Add Signatures");
        signaturies.push(_newSig);
    }
}
```
**Attack Vector**
As you can see, we can't withdraw funds from this contract until everyone signs and accepts it. So is there a way to withdraw funds? Let's take a look at the `buyFundsPublic` function. It requires the caller to pay a huge amount of USDC in exchange for a single CBDC, which must be a valuable token. Now take a look at the `upgradeUSDC` function; it updates the USDC token's address, but it's only accessible by the `centralBank`, we need to find a way to gain this access. Take a look at the `updateCentralBank` function; it requires the caller to be a CBDC oracle, but wait, we already gained Oracle privilege in the CBDC contract; let's take another look at the CBDC code.

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

We already know the secret word for the `addOracle` function; we can start from here.

**NOTE**
You have to create your own ERC20 token to solve this challenge; here is mine:

```solidity=
contract fakeUsdc is ERC20 {
    constructor() ERC20("SixieCow","sixie"){}
    function mint() public {
        _mint(msg.sender,type(uint256).max);
    }
}
```

**Exploit Scenario**

1. Deploy your fake `ERC20` token on the testnet.
2. Call the `mint` function to mint enough of the token.
3. Approve the `Multisig` contract enough to buy the CBDC token.
4. Call the `addOracle` function with `_secret` we found out on EP.3. This will make the caller's address an oracle.
5. Call the `updateCentralBank` function with our address.
6. Now that we have the CentralBank's privilege, we will update the USDC token's address so that it becomes our fake USDC token's address.
7. Call the `buyFundsPublic` function, and now we just got our CBDC token and broke the multisig wallet.

**Recommendation**
Check the given privileges really carefully and don't rely on vulnerable code. In our case, beside the vulnerable code, we trusted oracles to change the given privileges, which should not be the case. Privileges should be updated by owners or admins, and we should consider carefully who we trust with our functions.