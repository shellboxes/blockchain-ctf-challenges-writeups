**Description**

GalaNFT is a smart contract deployed on the `Goreli` testnet, which, as the name might suggest, is an `ERC721` smart contract or a contract that manages NFTs. The contract is very simple and has basic functionalities like minting, and each token has a `_tokenId`, increasing the maximum supply of available tokens. The contract in the `constructor` mints 100 tokens for the deployer; the `mint()` function has a `require` statement that requires the `_tokenId` to be less than the total supply; the challenge here is to bypass this condition.

* `GalaNFT.sol`

```solidity=
pragma solidity ^0.8.0;

import "@openzeppelin/contracts/token/ERC721/ERC721.sol";

contract GalaNFT is ERC721 {
    uint256 public _tokenId = 0;
    uint256 public _maxSupply = 100;

    constructor() ERC721("GalaNFT", "GALANFT") {
        do {
            mint(msg.sender);
        }
        while (_tokenId < _maxSupply); 
    }

    function mint(address _to) public {
        require (_tokenId <= _maxSupply,"You need to increase the maximum supply");
        _mint(_to, _tokenId);
        _tokenId += 1;
    }

    function increaseMaxSupply() public {
        _maxSupply += 1;
    }
}
```

**Attack Vector**
The first thing I noticed is that the `increaseMaxSupply()` and `mint()` functions are accessible by anyone, which means I can call `increaseMaxSupply()` and `mint()` to my address and get unlimited tokens, which is not what we want.

**PoC**

Trying to call `mint()` directly will always revert the transaction, the reason being that they require the `_tokenId` variable (which increments each time a token is minted) to be less than the `_maxSupply` so what we can do is :
1.  call `increaseMaxSupply` which increases the `_maxSupply` by 1.
2. execute `mint()` with your address as an argument.
3. Congratulations! You've just received your own NFT. 

**Recommendation**

As we can see, this contract lacks access controls, making it vulnerable because there are some functions that we don't want the public to control. We can fix this by adding access controls to the `increaseMaxSupply()` and `mint()` functions, so that only the owner or an admin can call them.