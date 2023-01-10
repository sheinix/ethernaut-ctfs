# Vault 🔐

Eight Challenge description is as follows:

Unlock the vault to pass the level!

Code:

```
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Vault {
  bool public locked;
  bytes32 private password;

  constructor(bytes32 _password) {
    locked = true;
    password = _password;
  }

  function unlock(bytes32 _password) public {
    if (password == _password) {
      locked = false;
    }
  }
}
```

### Resolution

The goal of this challenge is actually to know the `_password` to call the function `unlock(bytes32 _password) in order to set the flag `locked` to `false`

If we have a look at the `Vault` contract we can see that `password` variable is defined as `private` but this only means that other contracts or addresses cannot read the value, but every data that is published "on chain" is public. 

So in order to discover the password, we can grab the `Vault` contract address by calling `contract.address` in the console and then paste it in Etherscan. Then look for the contract creation transaction.

Once in the transaction details, if we look at the State transition of this transaction we can see two storage state changes. One is the change from `0` to `1` which means the set of the `locked` flag to `true`. The other state transition is from `0x0` to another hex value, that is translated to a string to: `A very strong secret password :)`  So there we have it! That's the setup of the password.

Now,  for calling the `unlock` function we need to pass the encoded `bytes32` so, you can just copy the hex value from etherscan: `0x412076657279207374726f6e67207365637265742070617373776f7264203a29` or another option to double check it is use a string to bytes32 online converter [like this one](
https://www.devoven.com/string-to-bytes32).

Do: `await contract.unlock('0x412076657279207374726f6e67207365637265742070617373776f7264203a29')` in the console and should be enough to set the `locked` flag to `false`!

And that's it! Vault contract should be unlocked you can check it out checking the `locked` variable! 💥 

## License

[MIT](https://choosealicense.com/licenses/mit/)