# Token

Fifth Challenge description is as follows:

The goal of this level is for you to hack the basic token contract below.

You are given 20 tokens to start with and you will beat the level if you somehow manage to get your hands on any additional tokens. Preferably a very large amount of tokens.

- Things that might help:

What is an odometer?

Code:
```
// SPDX-License-Identifier: MIT
pragma solidity ^0.6.0;

contract Token {

  mapping(address => uint) balances;
  uint public totalSupply;

  constructor(uint _initialSupply) public {
    balances[msg.sender] = totalSupply = _initialSupply;
  }

  function transfer(address _to, uint _value) public returns (bool) {
    require(balances[msg.sender] - _value >= 0);
    balances[msg.sender] -= _value;
    balances[_to] += _value;
    return true;
  }

  function balanceOf(address _owner) public view returns (uint balance) {
    return balances[_owner];
  }
}
```

### Resolution

The goal of this challenge is to get more tokens than your actual balance. The clue that they give us is: "what is an odomoter?". I did a quick search on google and an odomoter is a device for measuring the distance traveled by a vehicle. The trick here is the odometers reset to zero when they reach the maximum number they can hold. This is the clue. If we look into the `balances` mapping we can see the amount defined by `uint` and in the `transfer` method we are validating the parameters via a substraction.

This could lead to an underflow issue, on which we substract an amount greater than the one in our balance (20) provoking the "reseting of the odometer" in this case the `uint` type. Because `uint` is an unsigned integer, it cannot be negative, it will just go to the upper limit after 0.

With that information, we could trigger the underflow by simply calling the `transfer` function with more than our balance (20) and transfer to another address. This will set our balance with a huge number, let's see:

```
// Check your balance first:
fromWei(await contract.balanceOf("{YOUR_WALLET_ADDRESS}"))

// Call the transfer with more than your balance - in this case 21 is ok:
await contract.transfer('{ANOTHER_ADDRESS}', '21')

// Check your balance again, your balance should have been underlow with a big number
fromWei(await contract.balanceOf("{YOUR_WALLET_ADDRESS}"))
```

And that's it! You learned about underflow and overflow and how unsafe are arithmetic operations in Solidity. :P You can submit your instance! 💥 

## License

[MIT](https://choosealicense.com/licenses/mit/)