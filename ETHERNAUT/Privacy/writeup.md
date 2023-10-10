**Description**

This challenge is labeled as challenging, but it's quite easy; it requires a basic understanding of the EVM storage, and I'll give a brief explanation about it.

Basically, the EVM storage is set up in the form of 256-bit slots, each slot can hold a single or multiple variables, depending on the size of the variable's data type. For instance, a variable of type `uint256` holds the whole slot, but two variables of type `uint128` will hold the same slot if declared right next to each other with no other variables that can share slots with them.

Moving on to type casting, since the EVM follows the big endian memory model, it stores the least significant byte into the highest memory address. This means that when casting from a larger data type to a smaller one, it will store the most significant bytes of the initial one.

With this knowledge, the challenge should be a piece of cake, Let's take a look at the code.
- **Privacy.sol**
```solidity=
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Privacy {

  bool public locked = true;
  uint256 public ID = block.timestamp;
  uint8 private flattening = 10;
  uint8 private denomination = 255;
  uint16 private awkwardness = uint16(block.timestamp);
  bytes32[3] private data;

  constructor(bytes32[3] memory _data) {
    data = _data;
  }
  
  function unlock(bytes16 _key) public {
    require(_key == bytes16(data[2]));
    locked = false;
  }

  /*
    A bunch of super advanced solidity algorithms...

      ,*'^`*.,*'^`*.,*'^`*.,*'^`*.,*'^`*.,*'^`
      .,*'^`*.,*'^`*.,*'^`*.,*'^`*.,*'^`*.,*'^`*.,
      *.,*'^`*.,*'^`*.,*'^`*.,*'^`*.,*'^`*.,*'^`*.,*'^         ,---/V\
      `*.,*'^`*.,*'^`*.,*'^`*.,*'^`*.,*'^`*.,*'^`*.,*'^`*.    ~|__(o.o)
      ^`*.,*'^`*.,*'^`*.,*'^`*.,*'^`*.,*'^`*.,*'^`*.,*'^`*.,*'  UU  UU
  */
}
```

Looking at the code I know for sure that I need data in the fifth slot to call the "unlock," because even though the first variable is a boolean and only holds one byte, the second variable requires a full 256-bit slot to be stored. Now that we've reached index 2, we have a bytes32 array, where each element holds a 256-bit slot, which means that to get the element at index 2, we need to add 3 to the index we reached. Now we're at index 5. Let's hop onto the console. We are going to use the following command:

```
web3.eth.getStorageAt(instance, 5)
```
This will return the following value:

```
0x718651faf77b1930e1e59c5beaa941b017d85c58de8c068559869fa5a07069ec
```
This is the data in data[2], but remember it is cast to bytes16, meaning we are only going to take the first 16 bytes of this data, leaving us with:

```
0x718651faf77b1930e1e59c5beaa941b0
```
Now we are just going to use the following command in the console:

```
contract.unlock('0x718651faf77b1930e1e59c5beaa941b0')
```

and we just hacked it.

**Recommendation**
As we can see, even though the array was private, we could still retrieve data from it. Therefore, it is recommended to never store sensitive data on the contract itself.