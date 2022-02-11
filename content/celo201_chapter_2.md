# Celo Development 201 - NFT Contract Development with Hardhat

In this learning module, we will learn how to develop smart contracts with Hardhat.

Hardhat has become the most popular development environment for smart contracts. We can use it to compile, test, debug and deploy our smart contracts.

We can interact with Hardhat via its CLI to run tasks. There are many built-in tasks that we can use, e.g., `npx hardhat compile` to compile contracts, but we can also create our own tasks to optimize our workflow.

Hardhat is built with a modular approach, where most of its functionality comes from its plugins. For example, we could test with ethers.js and waffle or with web3.js and truffle or use completely different tools. Here is a list of the official plugins https://hardhat.org/plugins/.

Hardhat also has a local blockchain built-in, the Hardhat Network, with which we can easily test and debug our contracts.

Let's set up our Hardhat environment next.

## 1. Project Setup

For this learning module, we will need a version of node.js that is higher or equal to 12.

Open your terminal and check your node version.

```sh
node -v
```

If you have a version of node.js that is lower than 12, please upgrade node.js.

We create a new directory for this project.

```sh
mkdir celo-hardhat
```

We navigate into the directory.

```sh
cd celo-hardhat
```

We create a new package.json file.

```sh
npm init
```

We install Hardhat.

```sh
npm install hardhat
```

We run hardhat.

```sh
npx hardhat
```

During the configuration select “Create a basic sample project” and use the preselected settings.

Now we need to install the dependencies as suggested in the configuration.

```sh
npm install --save-dev @nomiclabs/hardhat-ethers ethers @nomiclabs/hardhat-waffle ethereum-waffle chai
```

We can get an overview of the built-in Hardhat tasks with:

```sh
npx hardhat help
```

### 1.2 Celo configuration

We want to enable Hardhat to deploy to the Celo network, in this case to the alfajores testnet. To do that we need to add a network entry to our `hardhat.config.js`.

Change your “module.exports” to:

```
module.exports = {
  solidity: "0.8.4",
  networks: {
    alfajores: {
      url: "https://alfajores-forno.celo-testnet.org",
      accounts: {
        mnemonic: process.env.MNEMONIC, // line 25
        path: "m/44'/60'/0'/0" // line 26
      },
      chainId: 44787
    }
  }
};
```

You can see that we are importing a `MNEMONIC` (A list of words to access crypto assets) to get access to a Celo account (line 25). We will store our mnemonic in an environment variable in a second.

One other thing that is important to mention here is that the derivation path (line 26) changes depending on how your mnemonic is created. This configuration is for an account created with the Metamask wallet. For an account created with the Celoextensionwallet the path would be `path: "m/44'/52752'/0'/0"` for example.

We will need `dotenv` to read our mnemonic from our environment variable so let's require it on top of the file.

```
require("@nomiclabs/hardhat-waffle");
require('dotenv').config({path: '.env'});
```

Let’s install dotenv too.
`npm install dotenv`

For the next part, you will need to have Metamask installed with an account that has alfajores testnet tokens.

In the "Intro to NFT Contract Development" learning module, you learn how to create an account via Metamask and how to get testnet tokens for it via the Celo faucet.

In Metamask, you can click on the identicon, go to settings, select "Security & Privacy ", click on "Reveal Secret Recovery Phrase", and copy that phrase.

Create a new file in your main directory called `.env`. Inside the `.env` file, store your mnemonic. It should look like this:

```
MNEMONIC="YOUR_SECRET_RECOVERY_PHRASE`
```

To test if everything worked as intended try to deploy the testnet to alfajores via our terminal.

```sh
npx hardhat run scripts/sample-script.js --network alfajores
```

If everything worked correctly, Hardhat should return the address to which the contract was deployed.

## 2. Write and compile smart contracts

This section looks into the example greeter contract and compiles it.

### 2.1 Greeter Contract

Inside the `contracts/Greeter.sol` file that is part of the sample project, you should see the following code.

```
//SPDX-License-Identifier: Unlicense
pragma solidity ^0.8.0;

import "hardhat/console.sol";

contract Greeter {
    string private greeting; // line 7

    constructor(string memory _greeting) {
        console.log("Deploying a Greeter with greeting:", _greeting);
        greeting = _greeting; // line 11
    }

    function greet() public view returns (string memory) { // line 14
        return greeting;
    }

    function setGreeting(string memory _greeting) public { // line 18
        console.log("Changing greeting from '%s' to '%s'", greeting, _greeting);
        greeting = _greeting;
    }
}
```

At this point, we expect you to be familiar with Solidity contract development; otherwise, look at our Solidity learning resources.

This is a very simple contract that stores a `greeting` string (line 7) set in the constructor when deploying the contract (line 11).
We also have a function `greet` that returns the string (line 14) and a function `setGretting` that allows us to set a new value for greeting (line 18).

To easier debug our Solidity contracts, Hardhat allows us to log messages and variables via `console.log()` (lines 10 and 19). We will see the messages later when we run our tests.

### 2.2 Compiling

Before we compile our contract, we need to remove all artifacts that we created when we deployed our contract in the last section so we are aware of what is happening.

We use Hardhat's built-in clean task for that.

```sh
npx hardhat clean
```

To compile our contracts, we can use the built-in compile task that we have seen earlier in this learning module.

```sh
npx hardhat compile
```

By default, the compiled artifacts will be saved in the `artifacts` directory. You might want to change this later if you want your Dapp to access the ABI, for example.

After compiling your contract, you should be able to find the ABI and the compiled byte code in the `artifacts/contracts/greeter.sol/greeter.json` file.
