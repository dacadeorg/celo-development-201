# Celo 201 - Build an NFT Minter with Hardhat and React

<!-- TODO:
- add more explanation about what we are going to build (Monorepo // Full Stack)

- add tests for the contracts
- should we use the loader somewhere?
- look into hooks and the useCount hook / update counter util
- add correct links to learning modules.
- add gitpod yml later.
-->

In this learning module, we will build the frontend for an NFT minting contract.

This tutorial will walk you through building a NFT minter with Celo.

### Tech Stack

We will use the following tech stack in this learning module:

- [Hardhat](https://hardhat.org/) - A Solidity development environment.
- [React](https://reactjs.org/) - A JavaScript library for building user interfaces.
- [Bootstrap](https://getbootstrap.com/) - A CSS framework.
- [useContractKit](https://github.com/celo-org/use-contractkit) - A React hook to interact with the Celo Blockchain.
- [IPFS](https://ipfs.io/) - A distributed file storage system.

### Prerequisites

You will need the following for this learning module:

- [Node JS](https://nodejs.org/en/download/) - Please make sure you have Node.js v12 or higher installed.
- Understanding of Solidity and the Celo Blockchain.
- Know how to use Hardhat to develop Solidity contracts.

## 1. Project Setup

In the first section of this learning module, we will set up the project and install the necessary dependencies.

1. Clone the repository:

```sh
git clone https://github.com/dacadeorg/celo-react-boilerplate.git
```

2. Navigate to the directory:

```sh
cd celo-react-boilerplate
```

3. Install the dependencies:

```sh
npm install
```

4. Create a envoironment variable for your mnemonic.

- Create a file named `.env` in the root directory of the project.

- Add your mnemonic into the file, like this:

```
MNEMONIC="YOUR_SECRET_RECOVERY_PHRASE`
```

In this case, we are using a mnemonic from an account created on Metmask. You can copy it from your Metamask account settings. An account created on the Celo extension wallet will not work.

You can find more details about the whole process in the [NFT Contract development with Hardhat]() learning module.

### Folder Structure

Your project should now have the following folder structure:

```
├── contracts
├── public
├── scripts
├── src
└── test
```

It is useful to have finished our learning module: "Celo NFT Contract Development with Hardhat". The `contracts`, `scripts` and `test` folders belong to the Hardhat environment and are explained in more detail in that module.

The `contracts` folder contains the smart contracts that Hardhat uses. In the case of the boilerplate, this is an example `Counter` smart contract.

The `scripts` folder contains the scripts that Hardhat uses to deploy the smart contracts. In the case of the boilerplate, this is an example `deploy.js` script.

The `test` folder contains the tests that Hardhat uses to test the smart contracts. In the case of the boilerplate, this is an example `contract-tests.js` script.

The `src` folder contains the React frontend.

The `public` folder contains the compiled static files, that will be served by the web server.

## 2. Boilerplate

In this section, we will look into the boilerplate that we just installed.

Inside the `src` directory, we find the following folders and the index.js and App.js files:

```
├── src
│   ├── components
│   ├── contracts
│   ├── hooks
│   ├── utils
│   ├── index.js
│   └── App.js
```

In the following sub sections, we will go through the files and folders and explain what they are used for. Let's start by going through the `index.js`.

### 2.1 index.js

In the `index.js` file, we import the React library and the `App` component, as well as CSS files for styling and `ContractKitProvider`, `Alfajores` and `NetworkNames` from `use-contractkit`.

In order to connect to a Celo network, we need to wrap the `ContractKitProvider` around the `App` component:

```js
...
ReactDOM.render(
  <React.StrictMode>
    <ContractKitProvider
      networks={[Alfajores]}
      network={{
        name: NetworkNames.Alfajores,
        rpcUrl: "https://alfajores-forno.celo-testnet.org",
        graphQl: "https://alfajores-blockscout.celo-testnet.org/graphiql",
        explorer: "https://alfajores-blockscout.celo-testnet.org",
        chainId: 44787,
      }}
      dapp={{
        name: "Celo React Boilerplate",
        description: "A React Boilerplate for Celo Dapps",
      }}
    >
      <App />
    </ContractKitProvider>
  </React.StrictMode>,
  document.getElementById("root")
);
...
```

The `use-contractkit` provides a `network` variable that contains the network configuration. We can use this to connect to the Celo testnet Alfajores.

### 2.2 App.js

The `App.js` file is the main entry point for the React frontend. It contains the `App` component, which is the container for all other components.

At the top of the `App.js` file, we import the libraries, components and hooks we will use. Our `App` component looks like this:

```js
...
const App = function AppWrapper() {
  const { address, destroy, connect } = useContractKit();
  const { balance } = useBalance();
  const counterContract = useCounterContract();

  return (
    <>
      <Notification />
      {address ? (
        <Container fluid="md">
          <Nav className="justify-content-end pt-3 pb-5">
            <Nav.Item>
              <Wallet
                address={address}
                amount={balance.CELO}
                symbol="CELO"
                destroy={destroy}
              />
            </Nav.Item>
          </Nav>
          <main>
            <Counter counterContract={counterContract} />
          </main>
        </Container>
      ) : (
        <div className="App">
          <header className="App-header">
            <Cover connect={connect} />
          </header>
        </div>
      )}
    </>
  );
};
...
```

We add the `Notification` component at the top of the JSX of our `App` component. This component is used to display notifications to the user. We will explain the components in more detail in the following sub-sections.

If the user is not connected to the Celo network, we display a cover component and pass `useContractKit`'s `connect` method as a prop to the cover component.

If the user is connected, we display our dapp. Which mainly consists of navigation with the `Wallet` component and a main element with the `Counter` component.

We pass `useContractKit`'s `address` and `balance.celo` variables as props to the `Wallet` component, as well as the `destroy` method. The `symobl` prop is used to display the symbol of the token, in this case Celo.

The `Counter` gets the `counterContract` smart contract instance from our `useCounterContract` custom hook.

<!--- TODO: add more explanation about what is in the App component -->

### 2.3 components

Inside the components folder, we find the following files and folders:

```
├── components
│   ├── ui
│   ├── Counter.js
│   ├── Cover.js
│   └── Wallet.js
```

#### Cover.js

The `Cover.js` file contains the `Cover` component. This component contains a landing page used when the user is not connected to the Celo testnet and features a button to connect.

#### Wallet.js

The `Wallet` component is a simple component that displays the address as an identicon (a visual representation of the address) and the Celo balance of the account that the dapp is connected to. If we toggle the `Wallet` component, we will see the truncated (shortened) address and a button to disconnect the account.

The `address`, `amount`, `symbol` and `destroy` props are passed to the `Wallet` component from the `App` component.

#### Counter.js

The `Counter.js` file contains the `Counter` component, which is used to display the current value of a counter, and two buttons to increment and decrement the counter.

We use the `counterContract` prop to access an instance of the `Counter` smart contract. With the instance of the contract and the `performAction` method, we can call the `increaseCount` and `decreaseCount` methods of the smart contract.

#### ui

The `ui` folder contains the following files:

```
├── ui
│   ├── Identicon.js
│   ├── Loader.js
│   └── Notification.js
```

The `Identicon` component is used to display the address as an identicon (a visual representation of the address). The identicon is generated using the Metamask's `Jazzicon` library. We use the identicon in the `Wallet` component.

The `Loader` component is used to display a loading animation when we are waiting for a response from the Celo network.

The `Notification` component is used to display notifications to the user. This component uses the `react-toastify` library to display notifications. We implement `Notification` in the `App` component.

### 2.4 hooks

The `hooks` folder contains the following files:

```
├── hooks
│   ├── index.js
│   ├── useBalance.js
│   ├── useContract.js
│   └── useCounterContract.js
```

#### useBalance.js

The `useBalance.js` file contains the `useBalance` custom hook. This hook is used to get the balance of the account that the dapp is connected to. We use contractkit's `kit` method `getTokenBalance` to query the balances of Celo tokens. The `useBalance` hook is implemented in the `App` component.

#### useContract.js

The `useContract.js` file contains the `useContract` custom hook. This hook is used to get an instance of a smart contract. `useContract` needs an `abi` and a `contractAddress` as props. Then we can use `kit.web3.eth.contract` to create an instance of the smart contract. The `useContract` hook is implemented in the `useCounterContract` custom hook that we are going to look at next.

This hook is very generic and can be used to get an instance of any smart contract.

#### useCounterContract.js

The `useCounterContract.js` file contains the `useCounterContract` custom hook. This hook imports the abi and contract address from json files that are generated by the Hardhat deploy script. With the abi and contract address, we can create and export an instance of the `Counter` smart contract.

The `useCounterContract` hook is implemented in the `App` component.

### 2.5 utils

The `utils` folder contains the following files:

```
├── utils
│   ├── constants.js
│   ├── counter.js
│   └── index.js
```

These files contain constants and untility functions that are used in the dapp.

#### index.js

The `index.js` file contains the functions `truncateAddress` and `formatBigNumber`. These functions are used to shorten an address and convert a very large number to a human readable format.

#### constants.js

The `constants.js` file for now just contains the `CELO_DECIMALS` constant,which stores the number of decimals Celo tokens have. We use this constant in th `formatBigNumber` function.

#### counter.js

In the `counter.js` file, we implement the `increaseCount` and `decreaseCount` functions. These functions get the `counterContract` instance and `useContractKit`'s `performAction` as props, so we can call the `increaseCount` and `decreaseCount` methods of the smart contract.

The `counter.js` file is used in the `Counter` component.

Now that we have gone through our boilerplate dapp, we can start to adapt it to our needs.

## 3. Minter contract

In this section, we will create the smart contract for the minter dapp, create tests for the smart contract and deploy it to the Celo testnet.
We will keep this chapter brief; a more detailed explanation of NFTs and Hardhat development is provided in separate learning modules.

### 3.1 Contract

Inside the `contracts` folder in our root directory, we find the `MyContract.sol` file. Delete that file and create a new one, called `MyNFT.sol`. Add the following code to the file:

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

This contract is a simple NFT contract that has, among other things, the following features:

- It has a `mint` function that mints an NFT to a given address. Each NFT has a unique ID generated by the `_tokenIdCounter` counter.
- It has a `tokenURI` function that returns the url of the NFT metadata.
- It imports the `ER721Enumerable` contract that allows us to keep track of all the NFTs that have been minted.

You can find more details about this contract in the [NFT Contract development with Hardhat]() learning module.

We import contracts from the `@openzeppelin/contracts` library. To use them we need to add them to our project first:

```sh
npm install @openzeppelin/contracts
```

Now we can compile our contract:

```sh
npm install @openzeppelin/contracts
```
