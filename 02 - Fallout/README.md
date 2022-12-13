# Fal1Out

Second Challenge description is as follows:

Claim ownership of the contract below to complete this level.

Things that might help

- Solidity Remix IDE

Code:
```
// SPDX-License-Identifier: MIT
pragma solidity ^0.6.0;

import 'openzeppelin-contracts-06/math/SafeMath.sol';

contract Fallout {
  
  using SafeMath for uint256;
  mapping (address => uint) allocations;
  address payable public owner;


  /* constructor */
  function Fal1out() public payable {
    owner = msg.sender;
    allocations[owner] = msg.value;
  }

  modifier onlyOwner {
	        require(
	            msg.sender == owner,
	            "caller is not the owner"
	        );
	        _;
	    }

  function allocate() public payable {
    allocations[msg.sender] = allocations[msg.sender].add(msg.value);
  }

  function sendAllocation(address payable allocator) public {
    require(allocations[allocator] > 0);
    allocator.transfer(allocations[allocator]);
  }

  function collectAllocations() public onlyOwner {
    msg.sender.transfer(address(this).balance);
  }

  function allocatorBalance(address allocator) public view returns (uint) {
    return allocations[allocator];
  }
}
```

### Resolution
The goal of this challenge is also to take ownership of this contract. Everything looks good enough on a first read, although when we look closer, there's something fishy going on. Take a look at the name of the contract `Fallout` and the "constructor" as commented in the code: `Fal1out`. There's a typo in the function name, which actually makes it a public function, and we can directly call it to set ourselves as owners:
```
await contract.Fal1out({ value: toWei('0.001') })

// check ownership now (it should be your address):
await contract.owner()
```


That's it! These steps should be enough to claim ownership of the contract! 💥 
## License

[MIT](https://choosealicense.com/licenses/mit/)