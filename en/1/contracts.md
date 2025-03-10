---
title: "Contracts"
actions: ['checkAnswer', 'hints']
requireLogin: true
material: 
  editor:
    language: sol
    startingCode: |
      pragma solidity //1. Enter solidity version here

      //2. Create contract here
    answer: > 
      pragma solidity ^0.8.21;


      contract ZombieFactory {

      }
---

Starting with the absolute basics:

Solidity's code is encapsulated in **contracts**. A `contract` is the fundamental building block of Ethereum applications — all variables and functions belong to a contract, and this will be the starting point of all your projects.

An empty contract named `HelloWorld` would look like this:

```
contract HelloWorld {

}
```

## Version Pragma

All solidity source code should start with a "version pragma" — a declaration of the version of the Solidity compiler this code should use. This is to prevent issues with future compiler versions potentially introducing changes that would break your code.

For the scope of this tutorial, we'll want to be able to compile our smart contracts with any compiler version that is compatible with 0.8.21. This includes version 0.8.21 and any subsequent 0.8.x versions, but excludes 0.9.0 and later.
It looks like this: `pragma solidity ^0.8.21;`.

Putting it together, here is a bare-bones starting contract — the first thing you'll write every time you start a new project:

```
pragma solidity ^0.8.21;

contract HelloWorld {

}
```

# Put it to the test

To start creating our Zombie army, let's create a base contract called `ZombieFactory`.

1. In the box to the right, make it so our contract uses solidity version `^0.8.21`.

2. Create an empty contract called `ZombieFactory`.

When you're finished, click "check answer" below. If you get stuck, you can click "hint".
