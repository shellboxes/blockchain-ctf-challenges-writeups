**Description**

The goal of this challenge is to learn how to deploy a contract on a testnet. The challenge is about trying to find the address of a user who stole a token with the ID 45.

**Attack Vector**
The `ERC721` has a function called `ownerOf()` that takes in the tokenID and returns the address of the owner; basically, we're just going to call this function in our contract. It is recommended to try and code it yourself before looking at the code.

**PoC**

Now that you know how to deploy the contract after following the steps outlined in the challenge, The code is as follows:

```solidity=
// SPDX-License-Identifier: GPL-3.0
pragma solidity ^0.8.0;

import "@openzeppelin/contracts/token/ERC721/IERC721.sol";

    function checkOwnership(uint256 _tokenId) public view returns (address) {
        return IERC721(0x484Ec30Feff505b545Ed7b905bc25a6a40589181).ownerOf(_tokenId);
    }
}
```
As explained before, the function returns the address of the owner, All that's left to do is deploy it using `RemixIDE` and call the function with the requested `_tokenId`, and this is how you get the address.