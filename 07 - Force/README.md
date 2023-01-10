# Force 💪🏻

Seventh Challenge description is as follows:

Some contracts will simply not take your money ¯\_(ツ)_/¯

The goal of this level is to make the balance of the contract greater than zero.

Things that might help:

- Fallback methods
- Sometimes the best way to attack a contract is with another contract.
- See the Help page above, section "Beyond the console"

Code:

```
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Force {/*

                   MEOW ?
         /\_/\   /
    ____/ o o \
  /~____  =ø= /
 (______)__m_m)

*/}
```

### Resolution

The goal of this challenge is to make the balance of this contract greater than zero. Ok, so first thing to come to mind is to just send some ETH to the contract right?

Well, yes, but as we can see from the code above, there's no either `receive` or `fallback` payable functions here that can be called upon receiving ETH. So apparently this should be "enough" to assume this contract cannot receive ETH, although there's are two more ways that we can send ETH to a contract!

One is very straightforward which is by self destroying another contract with a balance, and the other one more complicated is by pre-calculating the address of a contract and send ETH to that address before the bytecode is deployed, this way is more complex and involve very specific situations.

For the resolution, we're just gonna deploy a contract that self destructs and sends its' balance to the `Force` contract.

Let's try it!
```
// SPDX-License-Identifier: MIT
pragma solidity ^0.5.0;

contract AttackForce {
    
    address payable forceAddr = {FORCE_CONTRACT_ADDRESS};

    function forceDepositOnContract() public payable {
        selfdestruct(forceAddr);
    }
}

```

Once deployed, call the `forceDepositOnContract` with some ETH (as it is marked `payable`) and the contract will self destruct (delete bytecode) and send the ETH to the Force contract!

And that's it! If we check the ownership on the Delegate contract should have changed! 💥 

### Ethernaut's message after submiting:
In solidity, for a contract to be able to receive ether, the fallback function must be marked `payable`.

However, there is no way to stop an attacker from sending ether to a contract by self destroying. Hence, it is important not to count on the invariant `address(this).balance == 0 for any contract logic`.



## License

[MIT](https://choosealicense.com/licenses/mit/)