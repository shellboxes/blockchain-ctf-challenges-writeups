**Description**

This challenge is really interesting since we're technically reverse engineering a contract to find out a password hash. The point here is to analyze the transactions and figure out the password hash. The key here is to learn not to store any sensitive data in a smart contract since no data is private, even though we declared the password hashes as private, we had the ability to retreive this sensitive data.

**Attack Vector**

First, we didn't have any idea about the source code, only the declared state variables, and we had access to a smart contract that was not verified in the explorer, so trying to figure out the contract's logic was going to be time-consuming, so I went through the transactions to figure out the state changes that happened to the contract, thus figuring out the password hash. This challenge had many rabbit holes, which is a bit exhausting.
Before we start our writeup, it is required to have a good understanding of the EVM memory model and the slot system. I would recommend reading [this](https://steveng.medium.com/ethereum-virtual-machine-storage-layout-beb9a72a07e9) before getting started.
In the challenge, we got the following code snippet:

```solidity=
pragma solidity ^0.8.0;
    contract sherlock{
    uint256 public var256_1 = 1337;
    bool public bool_1 = false;
    bool public bool_2 = false;
    bool public bool_3 = true;
    uint16 public var16_1 = 32;
    uint16 private var16_2 = 64;
    address public contractAdd = address(this);
    uint256 private var256_2 = 3445;
    uint256 private var256_3 = 6677;
    bytes32 private iGotThePassword;
    bytes32 private actuallPass;
    bytes32 private definitelyThePass;
    uint256 public var256_4 = 7788;
    uint16 public var16_3 = 69;
    uint16 private var16_4 = 7;
    bool private _Pass = true;
    bool private _The = true;
    bool private _Password = false;
    address private owner;
    uint16 private counter;
    bytes32 public constant thePassword = ...................[REDACTED]...................
    bytes32 private constant ohNoNoNoNoNo = .................[REDACTED]...................
    bytes32[4] private passHashes;
    struct Passwords {
        bytes32 name;
        uint256 secretKey;
        bytes32 password;
    }
    Passwords[] private passwords;
    mapping (uint256 => Passwords) private destiny;
    ...................[REDACTED]................... 
    }
```

So from the first overview, I see that both variables `thePassword` and `ohNoNoNoNoNo` have hidden values, so they might be what we're looking for, let's take another look. As we can see, the first variable will be stored in its own slot, and the next six variables will be stored in a single slot. Knowing this information, we can jump directly to the memory slot we want. Let's hop on `etherscan`.

![](https://i.imgur.com/AWYK2y2.png)

Looking at the first state changes, I'm sure that the first slot contains the first `uint256` which has the value of `1337`. Let's see if I'm right.

![](https://i.imgur.com/NWYJZG4.png)

As we can see, this is the actual value; now, as I said, the following 6 variables share the same slot, Let's take a look.

![](https://i.imgur.com/TMad44l.png)

the first 2 bytes from the right have the value of `false` then the third has the value of `true` after we can see the following: 2 bytes have the value of `0x0020` which is 32 as declared in the state variables visible to us, then we have `0x0040` which is 64 as shown in the contract, and the remaining 20 bytes have the address of the owner.
Now that we have an idea of how this is going, let's jump directly to the variables that caught our interest.

![](https://i.imgur.com/DonUs8d.png)

This is the `thePassword` and `ohNoNoNoNoNo` variables, but remember, we're looking for a hash right now, so this is just a rabbit hole.
There is also another rabbit hole, which is the `passHashes` array, so we're going to skip it, Looking down at the Passwords struct, we have 3 attributes: `name`, `secretKey` and `password`. This indicates that we're looking for a specific password hash, but which one? The challenge creator's name is `razzor` and the challenge has a title that states, `can you find my password hash?` So now that we know what we're looking for, let's take another look at the state variables. We can see that we have an array of `Passwords` structs; we can just loop through this to find the password hash we're looking for.

![](https://i.imgur.com/EWNgHD0.png)

This is the first element of the array. Let's dig deeper.

![](https://i.imgur.com/8EFtjDe.png)
This is what we're looking for.

**Recommendation**
Even though the data was declared as private, we were able to retrieve its value, so it is highly recommended to not store private data inside a contract. If it is mandatory to store this data on the contract, it is recommended to store the hash and not the actual value.