# Celo Development 201 - NFT Contract Development with Hardhat

This learning module consists of a tutorial that will teach you how to develop, test and deploy smart contracts with Hardhat to the Celo testnet Alfajores. We will use an NFT contract as an example.

### Prerequisites

You will need the following for this learning module:

- [Node JS](https://nodejs.org/en/download/) - Please ensure you have Node.js v12 or higher installed.
- Basic understanding of NFT contracts as described in our [Intro to NFT Contract Development](https://hackmd.io/1_56InhcSO-oqrF8WXfhaQ) learning module.
- Be comfortable using a terminal.

### Tech Stack

In this tutorial, we will use the following tools and technologies:

- [Hardhat](https://hardhat.org/) - A Solidity development environment.
- [Node JS](https://nodejs.org/en/download/) - A JavaScript runtime environment.
- [OpenZeppelin Contracts](https://openzeppelin.com/contracts/) - A library of heavily tested smart contracts.

Hardhat has become the most popular development environment for smart contracts. We can use it to compile, test, debug and deploy our smart contracts.

We can interact with Hardhat via its CLI to run tasks. There are many built-in tasks that we can use, e.g., `npx hardhat compile` to compile contracts, but we can also create our own tasks to optimize our workflow.

Hardhat is built with a modular approach, where most of its functionality comes from its plugins. For example, we could test with ethers.js and waffle or with web3.js and truffle or use completely different tools. Here is a list of the official plugins: https://hardhat.org/plugins/.

Hardhat also has a local blockchain built-in, the Hardhat Network, with which we can easily test and debug our contracts.

Let's set up our Hardhat environment next.

## 1. Project Setup

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

During the configuration, select "Create a basic sample project" and use the preselected settings.

Now we need to install the dependencies as suggested in the configuration.

```sh
npm install --save-dev @nomiclabs/hardhat-ethers ethers @nomiclabs/hardhat-waffle ethereum-waffle chai
```

We can get an overview of the built-in Hardhat tasks with:

```sh
npx hardhat help
```

### 1.2 Celo Configuration

We want to enable Hardhat to deploy to the Celo network, in this case to the Alfajores testnet. To do that, we need to add a network entry to our `hardhat.config.js` file located at the project's root directory.

Change your `module.exports` to:

```js
// ...
module.exports = {
  solidity: "0.8.4",
  // declaring networks
  networks: {
    alfajores: {
      url: "https://alfajores-forno.celo-testnet.org", // (line 23) celo alfajores test network url
      accounts: {
        mnemonic: process.env.MNEMONIC, // (line 25) MNEMONIC also know as "recovery phrase" are lst of secret words
        path: "m/44'/60'/0'/0", // (line 26) derivation path 
      },
      chainId: 44787,
    },
  },
};
```

You can see that we are importing a `MNEMONIC` (A list of words to access crypto assets) to get access to a Celo account (line 25). We need to get access to a Celo account to interact with the Celo Alfajores testnet. We will store our mnemonic in an environment variable in a second.

One other thing that is important to mention here is that the derivation path (line 26) changes depending on how your mnemonic is created. This configuration is for an account created with the Metamask wallet. For an account created with the [CeloExtensionWallet](https://chrome.google.com/webstore/detail/celoextensionwallet/kkilomkmpmkbdnfelcpgckmpcaemjcdh?hl=en) the path would be `path: "m/44'/52752'/0'/0"` for example.

We will need `dotenv` to read our mnemonic from our environment variable so let's require it on top of the file.

```js
require("@nomiclabs/hardhat-waffle");
require("dotenv").config({ path: ".env" }); // importing the dotenv to read MNEMONIC
// ...
```

[Here](https://github.com/dacadeorg/celo-nft-minter/blob/master/hardhat.config.js) is the complete code for our `hardhat.config.js` file.

Let's install `dotenv`.

```
npm install dotenv
```

For the next part, you will need to have Metamask installed with an account that has Alfajores testnet tokens.

In the [Intro to NFT Contract Development](https://hackmd.io/1_56InhcSO-oqrF8WXfhaQ) learning module, you learn how to create an account via Metamask and how to get testnet tokens for it via the Celo faucet. You will need to do this for the next part.

In Metamask, you can click on the identicon, go to settings, select "Security & Privacy ", click on "Reveal Secret Recovery Phrase", and copy that phrase.

Create a new file in your main directory called `.env`. Inside the `.env` file, store your mnemonic. It should look like this:

```
MNEMONIC="YOUR_SECRET_RECOVERY_PHRASE"
```

To test if everything worked as intended, try to deploy the token to Alfajores via our terminal.

```sh
npx hardhat run scripts/sample-script.js --network alfajores
```

If everything worked correctly, Hardhat should return the address to which the contract was deployed. You should see something like this:

```
Compiling 2 files with 0.8.4
Solidity compilation finished successfully
Greeter deployed to: 0x454E8E8c37820D84351CCfa34AbCCc03870Fc1fE
```

## 2. Write and Compile Smart Contracts

This section looks into the example greeter contract and compiles it.

### 2.1 Greeter Contract

Inside the `contracts/Greeter.sol` file that is part of the sample project, you should see the following code.

```solidity
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

At this point, we expect you to be familiar with Solidity contract development; otherwise, look at our [Celo Development 101](https://dacade.org/communities/celo/courses/celo-development-101) course.

This is a very simple contract that stores a `greeting` string (line 7) set in the constructor when deploying the contract (line 11).
We also have a function `greet` that returns the string (line 14) and a function `setGretting` that allows us to set a new value for greeting (line 18).

To easier debug our Solidity contracts, Hardhat allows us to log messages and variables via `console.log()` (lines 10 and 19). We will see the messages later when we run our tests.

### 2.2 Compiling

Before we compile our contract, we need to remove all artifacts that we created when we deployed our contract in the last section to see better what is happening.

We use Hardhat's built-in clean task for that:

```sh
npx hardhat clean
```

We can use the built-in compile task that we used earlier to compile our contracts:

```sh
npx hardhat compile
```

By default, the compiled artifacts will be saved in the `artifacts` directory. You might want to change this later if you want to create a special folder for your Dapp to access the ABI, for example.

After compiling your contract, you should be able to find the ABI and the compiled byte code in the `artifacts/contracts/greeter.sol/greeter.json` file.

Let's take a look at how to test our contract.

## 3. Testing Smart Contracts

Creating automated tests is very important when writing smart contracts since they are immutable, and the data stored in them is often very valuable.

In this tutorial, we will use the [Waffle](https://ethereum-waffle.readthedocs.io/en/latest/) library for our tests. Waffle works with ethers.js and claims to be "Sweeter, simpler and faster than Truffle".

Tests in Waffle are written with the popular JavaScript test framework [Mocha](https://mochajs.org/) and the assertion library [Chai](https://chaijs.com/).

### 3.1 Testing File

The tests that we execute can be found in a directory called `test`. We can already find here an example file called `test/sample-test.js`. Let's open it.

In the first lines, we require `expect` from `chai` and `ethers` from `hardhat`.

```js
const { expect } = require("chai");
const { ethers } = require("hardhat");
// ...
```

Waffle makes it easier for us to assert Ethereum values. Have a look at [Waffles documentation](https://ethereum-waffle.readthedocs.io/en/latest/matchers.html) of Chai matchers (asserting functions) that you can use.

```js
// ...
describe("Greeter", function () {
  it("Should return the new greeting once it's changed", async function () {
// ...
```

We use Mocha's testing structure. We use `describe` and `it` functions to test our expectations.
In this case, we want to test our `greeter` function and expect it to return the new greeting after we change it.

Let's start the test by creating a new contract instance:

```js
// ...
const Greeter = await ethers.getContractFactory("Greeter");
// ...
```

We need more information than the Solidity contract code to deploy a contract, e.g., the compiled bytecode. The `ContractFactory` is an ethers.js abstraction that helps us deploy the contract with the needed information.

Now we can deploy the contract:

```js
// ...
const greeter = await Greeter.deploy("Hello, world!");
await greeter.deployed();

// ...
```

We use the `deploy` function of the contract factory to deploy the contract, passing the constructor arguments (`"Hello, world!"`). It returns a `Promise` that resolves to the contract instance `greeter` once it's deployed.

Then we use the `await` keyword to wait for the `deployed` function to resolve once the contract is successfully deployed.

After the contract is deployed, we can call its functions:

```js
// ...
expect(await greeter.greet()).to.equal("Hello, world!");
// ...
```

Here we call the `greet` function of the contract and use Chai's `expect` function to assert that the returned value is equal to the expected value.

Now we can change the greeting and test it again:

```js
// ...
    const setGreetingTx = await greeter.setGreeting("Hola, mundo!");

    // wait until the transaction is mined
    await setGreetingTx.wait();

    expect(await greeter.greet()).to.equal("Hola, mundo!");
  });
});
```

This time we call the `setGreeting` function of the contract and wait for the transaction to be mined. Then we assert that the returned value is equal to the expected value.

Now we can run our tests.

### 3.2 Running tests

We can run our tests with the built-in test task:
```sh
npx hardhat test
```

You should then see something like the following output:
```
  Contract: Greeter
    ✓ Should return the new greeting once it's changed (762ms)

  1 passing (762ms)
```

With the last command, we ran our test on the built-in Hardhat Network, a local Ethereum network that allows us to test and debug our smart contracts.

This is not the Celo blockchain; if you want to run the Celo blockchain locally, you should have a look at [Celo Ganache](https://github.com/celo-org/ganache-core) and how to run a [Loca Development Chain](https://docs.celo.org/developer-guide/development-chain).

For this tutorial, we will test our contracts locally on the Hardhat Network and then on the Celo Alfajores testnet.

#### 3.3 Running Tests on Alfajores 

If you have configured Alfajores, in the `hardhat.config.js` file, you can run your tests on the Alfajores testnet with the following command:

```sh
npx hardhat test --network alfajores
```

Since we are connecting and deploying our contracts on the Alfajores testnet, it will take a little bit longer for the tests to run.

After the tests are finished, you should see something like the following output:
```
  Greeter
    √ Should return the new greeting once it's changed (20547ms)


  1 passing (21s)
```

## 4. Deploying Smart Contracts

To deploy a smart contract to the Celo blockchain, we use a deployment script called. You can find the deployment script in the `sample-script.js` file in the `scripts` directory.

This deployment functionality in this script is almost the same as the one we used in the previous section when we tested our contract.

```js
const hre = require("hardhat");

async function main() {
  const Greeter = await hre.ethers.getContractFactory("Greeter");
  const greeter = await Greeter.deploy("Hello, Hardhat!");

  await greeter.deployed();

  console.log("Greeter deployed to:", greeter.address);
}

main()
  .then(() => process.exit(0))
  .catch((error) => {
    console.error(error);
    process.exit(1);
  });
```

We require the `hre` ([Hardhat Runtime Environment](https://hardhat.org/advanced/hardhat-runtime-environment.html)) and define a function called `main`. We create a new instance of the `ContractFactory` and call the `deploy` function of the contract factory to deploy the contract. When the contract is deployed, we display a message with the contract address.

Now we can run our deployment script.

### 4.1 Deploying to Local Network

We can start a local built-in blockchain with the following command:

```sh
npx hardhat node
```

Upon starting the local node, we get a number of accounts with Ether to use. You should see something like the following output:

```
Started HTTP and WebSocket JSON-RPC server at http://127.0.0.1:8545/

Accounts
========

WARNING: These accounts, and their private keys, are publicly known.
Any funds sent to them on Mainnet or any other live network WILL BE LOST.

Account #0: 0xf39fd6e51aad88f6f4ce6ab8827279cfffb92266 (10000 ETH)
Private Key: 0xac0974bec39a17e36ba4a6b4d238ff944bacb478cbed5efcae784d7bf4f2ff80
```

You should see 19 more accounts with Ether.

Create a new terminal and run the following command to deploy the contract to the local node:

```sh
npx hardhat run --network localhost scripts/sample-script.js
```

You should see something like the following output:
```
Greeter deployed to: 0x5FbDB2315678afecb367f032d93F642f64180aa3
```

Now we can open a Hardhat console and connect to the local node:
```sh
npx hardhat console --network localhost
```

We can use this console to interact with the contract locally. Let's get the `contractFactory` of the `greeter` contract:

```sh
const Greeter = await ethers.getContractFactory('Greeter');
```

You will see an `undefined` value as an output.

Next, we want to get the instance of the contract:

```sh
const greeter = await Greeter.attach('0x5FbDB2315678afecb367f032d93F642f64180aa3')
```

Use the address that your contract was deployed to.
You will again see an `undefined` value as an output.

Now we can call our contract's functions:
```sh
await greeter.greet()
```

You should see the following output:
```sh
'Hello, Hardhat!'
```

We can also call the `setGreeting` function of the contract:
```sh
await greeter.setGreeting('Gm')
```

Now we can call the `greet` function again:
```sh
await greeter.greet()
```

And should see the following output:
```sh
'Gm'
```

### 4.2 Deploying to Alfajores

Deploying to the Alfajores testnet is very similar to deploying to the local network. The commands are the same, but we need to specify the network as `alfajores`.

Run deploy script to deploy the contract to the Alfajores testnet:
```sh
npx hardhat run --network alfajores scripts/sample-script.js
```

Open a Hardhat console and connect to the Alfajores testnet:

```sh
npx hardhat console --network alfajores
```

Get the `contractFactory` of the `greeter` contract:

```sh
const Greeter = await ethers.getContractFactory('Greeter');
```

Get the instance of the contract:

```sh
const greeter = await Greeter.attach('0xa7597BDf14AB9b3E493257A79b4D1724A93A5E3a')
```

Use the address that your contract was deployed to on Alfajores.

Get the greeting:

```sh
await greeter.greet()
```

You can open the Celo Blockchain Explorer to see the contract on the Alfajores testnet: [https://alfajores-blockscout.celo-testnet.org/address/0xa7597BDf14AB9b3E493257A79b4D1724A93A5E3a/read-contract](https://alfajores-blockscout.celo-testnet.org/address/0xa7597BDf14AB9b3E493257A79b4D1724A93A5E3a/read-contract)

Change the address (`0xa7597BDf14AB9b3E493257A79b4D1724A93A5E3a`) to the address of your contract, and you can see the contract and the greeting on the Alfajores testnet.

## 5. Writing an NFT Contract

In this section, we will create an NFT contract based on the ERC721 standard.

Let's clean up before we start:

```sh
npx hardhat clean
```

Delete the example contract `contracts/Greeter.sol`:

```sh
rm contracts/Greeter.sol
```

For this section, we expect you to have gone through the [Intro to NFT Contract Development](https://hackmd.io/1_56InhcSO-oqrF8WXfhaQ) learning module and are familiar with NFT contracts.

We used Openzeppelin's [Contract Wizard](https://docs.openzeppelin.com/contracts/4.x/wizard) to create a secure contract that implements the ERC721 standard. The contract implements some additional features:

- It has a mint function that allows the owner to mint new tokens that automatically increment their token ID because we want to be able to mint tokens after the contract has been deployed and don't want to have to manually increment the token ID.

- Enumerable, allows us to list all the tokens that have been minted.

- URI Storage, allows us to store a URI for the metadata of each token.

Create a new contract file `contracts/MyNFT.sol` and copy the following code into it:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.2;

import "@openzeppelin/contracts/token/ERC721/ERC721.sol";
import "@openzeppelin/contracts/token/ERC721/extensions/ERC721Enumerable.sol";
import "@openzeppelin/contracts/token/ERC721/extensions/ERC721URIStorage.sol";
import "@openzeppelin/contracts/access/Ownable.sol";
import "@openzeppelin/contracts/utils/Counters.sol";

contract MyNFT is ERC721, ERC721Enumerable, ERC721URIStorage, Ownable {
    using Counters for Counters.Counter;

    Counters.Counter private _tokenIdCounter;

    constructor() ERC721("MyNFT", "MNFT") {}

    function safeMint(address to, string memory uri) public onlyOwner {
        uint256 tokenId = _tokenIdCounter.current();
        _tokenIdCounter.increment();
        _safeMint(to, tokenId);
        _setTokenURI(tokenId, uri);
    }

    // The following functions are overrides required by Solidity.

    function _beforeTokenTransfer(address from, address to, uint256 tokenId)
        internal
        override(ERC721, ERC721Enumerable)
    {
        super._beforeTokenTransfer(from, to, tokenId);
    }

    function _burn(uint256 tokenId) internal override(ERC721, ERC721URIStorage) {
        super._burn(tokenId);
    }

    function tokenURI(uint256 tokenId)
        public
        view
        override(ERC721, ERC721URIStorage)
        returns (string memory)
    {
        return super.tokenURI(tokenId);
    }

    function supportsInterface(bytes4 interfaceId)
        public
        view
        override(ERC721, ERC721Enumerable)
        returns (bool)
    {
        return super.supportsInterface(interfaceId);
    }
}
```

Most of the functions should look familiar to you if you went through the [Intro to NFT Contract Development](#deploying-smart-contracts) learning module. There are multiple overrides that we need to implement here, because some of the functions are defined multiple times by the contracts we are inheriting from.

The most important part of the contract is the `safeMint` function:

```solidity
// ...
    function safeMint(address to, string memory uri) public onlyOwner {
        uint256 tokenId = _tokenIdCounter.current();
        _tokenIdCounter.increment();
        _safeMint(to, tokenId);
        _setTokenURI(tokenId, uri);
    }
// ...
```

Only the owner of the contract can mint new tokens. They need to provide the address of the recipient and the URI of the token. The `tokenId` will be automatically assigned by the contract.

Before we can test the contract, we need to install the OpenZeppelin contracts that we are using:

```sh
npm install @openzeppelin/contracts
```

Let's compile the contract to make sure everything works:

```sh
npx hardhat compile
```

Great, now we can write some tests for our contract.

## 6. Testing NFT Contract

Hopefully, you remember how tests work from the previous sections.
Let's first delete the `test/sample-test.js` file:

```sh
rm test/sample-test.js
```

Create a new `test/nft-test.js` file:

```sh
touch scripts/deploy.js
```

Add the following code:

```js
const { expect } = require("chai");
const { ethers } = require("hardhat");

describe("MyNFT", function () {
  this.timeout(50000); // line 5

  let myNFT;
  let owner;
  let acc1;
  let acc2;

  this.beforeEach(async function() { // line 12
      const MyNFT = await ethers.getContractFactory("MyNFT");
      [owner, acc1, acc2] = await ethers.getSigners();

      myNFT = await MyNFT.deploy();
  })
// ...
```

On line 5, we set the timeout to 50 seconds. The default value is 20 seconds, but if we are going to test our contracts on the testnet, this can sometimes take longer.

Before every test (line 12), we deploy our NFT contract and create three accounts, the `owner`, `acc1`, and `acc2`. A Signer in Ethers.js represents an account that can sign transactions.

Next, we test if the contract has been deployed correctly to the owner's address:
```js
// ...
  it("Should set the right owner", async function () {
    expect(await myNFT.owner()).to.equal(owner.address);
  });
// ...
```

Now we check if the owner can mint a new token:

```js
// ...
  it("Should mint one NFT", async function() {
    expect(await myNFT.balanceOf(acc1.address)).to.equal(0);
    
    const tokenURI = "https://example.com/1" // line 28
    const tx = await myNFT.connect(owner).safeMint(acc1.address, tokenURI);
    await tx.wait(); // line 30

    expect(await myNFT.balanceOf(acc1.address)).to.equal(1);
  })
// ...
```

First, we make sure that the first account has none of our tokens. Then we set the `tokenURI` (line 28) and mint the NFT with the owner account to `acc1` with the `tokenURI` as metadata. 

We wait until the transaction goes through (line 30) before checking if the `acc1` now has 1 NFT.

Finally, we check if the right tokenURI has been set:
```js
// ...
  it("Should set the correct tokenURI", async function() {
    const tokenURI_1 = "https://example.com/1"
    const tokenURI_2 = "https://example.com/2"

    const tx1 = await myNFT.connect(owner).safeMint(acc1.address, tokenURI_1);
    await tx1.wait();
    const tx2 = await myNFT.connect(owner).safeMint(acc2.address, tokenURI_2);
    await tx2.wait();

    expect(await myNFT.tokenURI(0)).to.equal(tokenURI_1);
    expect(await myNFT.tokenURI(1)).to.equal(tokenURI_2);
  })
});
```

Here we set the tokenURI for the first token to `tokenURI_1` and the second token to `tokenURI_2`, wait until the transactions go through, and check if the tokenURI for the first token is `tokenURI_1` and the tokenURI for the second token is `tokenURI_2`.

The test is now complete; you can find the final testing file [here](https://github.com/dacadeorg/celo-nft-minter/blob/master/test/nft-test.js).

Additionally, you could test many more critical functions, like `transferFrom` or `enumerable`. But we want to keep this tutorial concise.

Now we can run the tests on the Hardhat network:

```sh
npx hardhat test
```

And we test it on the Alfajores network:

```sh
npx hardhat test --network alfajores
```

## 7. Deploying the NFT Contract

In the last section of this tutorial, we will create a script that will deploy the NFT contract to the owner account.

Delete the `scripts/sample-script.js` file:

```sh
rm scripts/sample-script.js
```

Create a new `scripts/deploy.js` file:

```sh
touch scripts/deploy.js
```

Add the following code:

```js
const hre = require("hardhat");

async function main() {
  const MyNFT = await hre.ethers.getContractFactory("MyNFT");
  const myNFT = await MyNFT.deploy();

  await myNFT.deployed();

  console.log("MyNFT deployed to:", myNFT.address);
}

main()
  .then(() => process.exit(0))
  .catch((error) => {
    console.error(error);
    process.exit(1);
  });
```

This code is the same as the deploy script we created in the previous section, except for a different contract name.

Finally, we can run the script:

```sh
npx hardhat run --network alfajores scripts/deploy.js
```

If everything went well, you should see something like this:
```
MyNFT deployed to: 0xcAa4101e01dC69EebF3C2EbEaA311af03bc3242D
```

As described in the previous section, you can now check your contract in the Celo Blockchain Explorer. You can also call your functions in the console to mint an NFT and transfer it to another account, for example.

Remember that you will need to provide some metadata as described in the [Intro to NFT Contract Development](https://hackmd.io/1_56InhcSO-oqrF8WXfhaQ) learning module.

Great job! You have now completed the NFT tutorial. You can now continue to the next learning module, which will teach you how to [Build an NFT Minter Dapp with React](https://hackmd.io/N4htCobqRKKI3mkjnc59Rg).
