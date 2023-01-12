# King 🤴🏻

Ninth Challenge description is as follows:

The contract below represents a very simple game: whoever sends it an amount of ether that is larger than the current prize becomes the new king. On such an event, the overthrown king gets paid the new prize, making a bit of ether in the process! As ponzi as it gets xD

Such a fun game. Your goal is to break it.

When you submit the instance back to the level, the level is going to reclaim kingship. You will beat the level if you can avoid such a self proclamation.


Code:

```
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract King {

  address king;
  uint public prize;
  address public owner;

  constructor() payable {
    owner = msg.sender;  
    king = msg.sender;
    prize = msg.value;
  }

  receive() external payable {
    require(msg.value >= prize || msg.sender == owner);
    payable(king).transfer(msg.value);
    king = msg.sender;
    prize = msg.value;
  }

  function _king() public view returns (address) {
    return king;
  }
}
```

### Resolution
The goal of this level is to set ourselves as the new king by sending some ETH to the contract, but also breaking it so that the next entity who tries to become king by sending ETH cannot do it.

On a first read, everything seems pretty good, and actually took me a long time to figure, as I initially thought that could be something related either with under/over flow or logic condition. But whatever we try, the Ethernaut game is going to try to claim kingship again so that means that we need to do something in order to make it impossible to claim kingship again!

To avoid claiming kingship again, the `receive()` function needs to revert. For that, it's either reverting on the `require` or in the `transfer` function.

Looking at the `require` and assuming the Ethernaut game is the `owner` doesn't matter how much we try to break the `prize` it will always go through for the owner of the contract.

So that leaves us with the possibility of making `payable(king).transfer(msg.value)` fail after we set ourselves as king.

There's a few reasons why `transfer` could fail but one straightforward way is to have a contract without any `payable` nor `fallback` function, so no one can send ETH to the contract (except for self destruct or pre-calculate address but that's out of scope in this situation).

In this scenario, the contract sends ETH to the `King` contract, sets it's self as the new king, and when the level tries to claim kingship again, the `payable(king).transfer(msg.value)` function will always fail, not reaching to the statement where it sets a new king!

Let's try it! First, you can check out from Etherscan what is the current `prize` from the contract creation transaction parameters. (`1000000000000000` wei). So, let's deploy a contract and send `1000000000000001` to the `King` Contract!

```
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract AttackKing__Ethernaut {
    address kingContractAddr = {KING_CONTRACT_ADDRESS};
    
    // Call this function with 1000000000000001 wei
    function attackKingContract() public payable {
         (bool success, ) = kingContractAddr.call{ value: msg.value }("");
         require(success, "");          
    }
}
```

And that's it! Now the level will try to claim kingship but the transfer will fail and the king will be us forever! 💥 

## License

[MIT](https://choosealicense.com/licenses/mit/)