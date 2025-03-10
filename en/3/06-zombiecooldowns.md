---
title: Zombie Cooldowns
actions: ['checkAnswer', 'hints']
requireLogin: true
material:
  editor:
    language: sol
    startingCode:
      "zombiefeeding.sol": |
        pragma solidity ^0.8.21;

        import "./zombiefactory.sol";

        interface KittyInterface {
          function getKitty(uint256 _id) external view returns (
            bool isGestating,
            bool isReady,
            uint256 cooldownIndex,
            uint256 nextActionAt,
            uint256 siringWithId,
            uint256 birthTime,
            uint256 matronId,
            uint256 sireId,
            uint256 generation,
            uint256 genes
          );
        }

        contract ZombieFeeding is ZombieFactory {

          KittyInterface kittyContract;

          function setKittyContractAddress(address _address) external onlyOwner {
            kittyContract = KittyInterface(_address);
          }

          // 1. Define `_triggerCooldown` function here

          // 2. Define `_isReady` function here

          function feedAndMultiply(uint _zombieId, uint _targetDna, string memory _species) public {
            require(msg.sender == zombieToOwner[_zombieId]);
            Zombie storage myZombie = zombies[_zombieId];
            _targetDna = _targetDna % dnaModulus;
            uint newDna = (myZombie.dna + _targetDna) / 2;
            if (keccak256(abi.encodePacked(_species)) == keccak256(abi.encodePacked("kitty"))) {
              newDna = newDna - newDna % 100 + 99;
            }
            _createZombie("NoName", newDna);
          }

          function feedOnKitty(uint _zombieId, uint _kittyId) public {
            uint kittyDna;
            (,,,,,,,,,kittyDna) = kittyContract.getKitty(_kittyId);
            feedAndMultiply(_zombieId, kittyDna, "kitty");
          }

        }
      "zombiefactory.sol": |
        pragma solidity ^0.8.21;

        import "./ownable.sol";

        contract ZombieFactory is Ownable {

            event NewZombie(uint zombieId, string name, uint dna);

            uint dnaDigits = 16;
            uint dnaModulus = 10 ** dnaDigits;
            uint cooldownTime = 1 days;

            struct Zombie {
              string name;
              uint dna;
              uint32 level;
              uint32 readyTime;
            }

            Zombie[] public zombies;

            mapping (uint => address) public zombieToOwner;
            mapping (address => uint) ownerZombieCount;

            constructor() Ownable(msg.sender) { }

            function _createZombie(string memory _name, uint _dna) internal {
                zombies.push(Zombie(_name, _dna, 1, uint32(block.timestamp + cooldownTime)));
                uint id = zombies.length - 1;
                zombieToOwner[id] = msg.sender;
                ownerZombieCount[msg.sender]++;
                emit NewZombie(id, _name, _dna);
            }

            function _generateRandomDna(string memory _str) private view returns (uint) {
                uint rand = uint(keccak256(abi.encodePacked(_str)));
                return rand % dnaModulus;
            }

            function createRandomZombie(string memory _name) public {
                require(ownerZombieCount[msg.sender] == 0);
                uint randDna = _generateRandomDna(_name);
                randDna = randDna - randDna % 100;
                _createZombie(_name, randDna);
            }

        }
      "ownable.sol": |
        pragma solidity ^0.8.20;

        import {Context} from "../utils/Context.sol";

        /**
        * @title Ownable
        * @dev The Ownable contract has an owner address, and provides basic authorization control
        * functions, this simplifies the implementation of "user permissions".
        */
        abstract contract Ownable is Context {
          address private _owner;

          /**
          * @dev The caller account is not authorized to perform an operation.
          */
          error OwnableUnauthorizedAccount(address account);

          /**
          * @dev The owner is not a valid owner account. (eg. `address(0)`)
          */
          error OwnableInvalidOwner(address owner);

          event OwnershipTransferred(
            address indexed previousOwner,
            address indexed newOwner
          );

          /**
          * @dev The Ownable constructor sets the original `owner` of the contract to the sender
          * account.
          */
          constructor(address initialOwner) {
            if (initialOwner == address(0)) {
              revert OwnableInvalidOwner(address(0));
            }
            _transferOwnership(initialOwner);
          }

          /**
          * @return the address of the owner.
          */
          function owner() public view virtual returns(address) {
            return _owner;
          }

          /**
          * @dev Throws if called by any account other than the owner.
          */
          modifier onlyOwner() {
            _checkOwner();
            _;
          }

          /**
          * @dev Throws if the sender is not the owner.
          */
          function _checkOwner() internal view virtual {
            if (owner() != _msgSender()) {
              revert OwnableUnauthorizedAccount(_msgSender());
            }
          }

          /**
          * @dev Allows the current owner to relinquish control of the contract.
          * @notice Renouncing to ownership will leave the contract without an owner.
          * It will not be possible to call the functions with the `onlyOwner`
          * modifier anymore.
          */
          function renounceOwnership() public virtual onlyOwner {
            _transferOwnership(address(0));
          }

          /**
          * @dev Allows the current owner to transfer control of the contract to a newOwner.
          * @param newOwner The address to transfer ownership to.
          */
          function transferOwnership(address newOwner) public virtual onlyOwner {
            if (newOwner == address(0)) {
              revert OwnableInvalidOwner(address(0));
            }
            _transferOwnership(newOwner);
          }

          /**
          * @dev Transfers control of the contract to a newOwner.
          * @param newOwner The address to transfer ownership to.
          */
          function _transferOwnership(address newOwner) internal virtual {
            address oldOwner = _owner;
            _owner = newOwner;
            emit OwnershipTransferred(oldOwner, newOwner);
          }
        }
    answer: >
      pragma solidity ^0.8.21;

      import "./zombiefactory.sol";

      interface KittyInterface {
        function getKitty(uint256 _id) external view returns (
          bool isGestating,
          bool isReady,
          uint256 cooldownIndex,
          uint256 nextActionAt,
          uint256 siringWithId,
          uint256 birthTime,
          uint256 matronId,
          uint256 sireId,
          uint256 generation,
          uint256 genes
        );
      }

      contract ZombieFeeding is ZombieFactory {

        KittyInterface kittyContract;

        function setKittyContractAddress(address _address) external onlyOwner {
          kittyContract = KittyInterface(_address);
        }

        function _triggerCooldown(Zombie storage _zombie) internal {
          _zombie.readyTime = uint32(block.timestamp + cooldownTime);
        }

        function _isReady(Zombie storage _zombie) internal view returns (bool) {
            return (_zombie.readyTime <= block.timestamp);
        }

        function feedAndMultiply(uint _zombieId, uint _targetDna, string memory _species) public {
          require(msg.sender == zombieToOwner[_zombieId]);
          Zombie storage myZombie = zombies[_zombieId];
          _targetDna = _targetDna % dnaModulus;
          uint newDna = (myZombie.dna + _targetDna) / 2;
          if (keccak256(abi.encodePacked(_species)) == keccak256(abi.encodePacked("kitty"))) {
            newDna = newDna - newDna % 100 + 99;
          }
          _createZombie("NoName", newDna);
        }

        function feedOnKitty(uint _zombieId, uint _kittyId) public {
          uint kittyDna;
          (,,,,,,,,,kittyDna) = kittyContract.getKitty(_kittyId);
          feedAndMultiply(_zombieId, kittyDna, "kitty");
        }

      }
---

Now that we have a `readyTime` property on our `Zombie` struct, let's jump to `zombiefeeding.sol` and implement a cooldown timer.

We're going to modify our `feedAndMultiply` such that:

1. Feeding triggers a zombie's cooldown, and

2. Zombies can't feed on kitties until their cooldown period has passed

This will make it so zombies can't just feed on unlimited kitties and multiply all day. In the future when we add battle functionality, we'll make it so attacking other zombies also relies on the cooldown.

First, we're going to define some helper functions that let us set and check a zombie's `readyTime`.

## Passing structs as arguments

You can pass a storage pointer to a struct as an argument to a `private` or `internal` function. This is useful, for example, for passing around our `Zombie` structs between functions.

The syntax looks like this:

```
function _doStuff(Zombie storage _zombie) internal {
  // do stuff with _zombie
}
```

This way we can pass a reference to our zombie into a function instead of passing in a zombie ID and looking it up.

## Put it to the test 

1. Start by defining a `_triggerCooldown` function. It will take 1 argument, `_zombie`, a `Zombie storage` pointer. The function should be `internal`.

2. The function body should set `_zombie.readyTime` to `uint32(block.timestamp + cooldownTime)`.

3. Next, create a function called `_isReady`. This function will also take a `Zombie storage` argument named `_zombie`. It will be an `internal view` function, and return a `bool`.

4. The function body should return `(_zombie.readyTime <= block.timestamp)`, which will evaluate to either `true` or `false`. This function will tell us if enough time has passed since the last time the zombie fed.
