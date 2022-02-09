# Celo 201 - Build an NFT Minter with Hardhat and React
In this chapter, you will learn how to build the user interface of your NFT minter, connect it to your smart contract and host it on GitHub.

### Tech Stack
We will use the following tech stack in this learning module:
* Hardhat - A Solidity development environment
* React - A JavaScript library for building user interfaces
* Bootstrap - A CSS framework
* useContractKit - A React hook to interact with the Celo Blockchain
* IPFS - A distributed file storage system

// TODO: Add links to projects

### Prerequisites
You will need the following for this tutorial:

* [Node JS](https://nodejs.org/en/download/) - Please make sure you have Node.js v12 or higher installed.
* [Yarn](https://docs.npmjs.com/downloading-and-installing-node-js-and-npm) - You will need to have Yarn installed on your machine.

### About
This tutorial will walk you through building a NFT minter with Celo.
// TODO: add more explanation about what we are going to build

## 1. Project Setup
Firstly, go ahead and clone or download the Celo react boilerplate code locally.

1. Open your terminal and navigate to the directory where downloaded the boilerplate code.
2. Install the dependencies with the command:
```
npm install
```

3. For this project we will need some other packages from NPM. Go ahead and run this command:
```
npm install axios ipfs-http-client
```

6. Start a local development server:
   ​​`npm start`

Your boilerplate should be running here http://localhost:3000/

## 2. Building the User Interface

### 2.1 Let us do a little cleanup first.



Navigate to the ‘public/index.html’ page and change the title and meta description of your website to your preferred texts. This step is optional.

Next, you can go to the *'index.js'* file and change the name and description values inside the ContractKitProvider wrapper to your project title and description. This set is also optional.

Secondly, Open the src folder and delete the ‘App.test.js’, and ‘setupTests.js’ files. We will not be needing these files for this tutorial.


<!-- Next, we add a cover image. Create a folder called _‘assets'_ under the _'src'_ folder , _‘src/assets’ then proceed by creating a folder inside of the ‘assets’ folder created called ‘img’ and then paste the ‘nft_geo_cover.png’ image file._
 -->

### 2.2 Explaining the boilerplate code

The boilerplate code consists of a couple of components in the *'src'* folder we would briefly discuss.

Firstly, the *'assets'* folder will be where we would save images we would be using in our NFT minter.

Secondly, in the components folder will, we will create reusable components which we can import into other files to handle various individual functions and display pieces of UI we would need.

Thirdly, we have the contracts folder which has two files: *'MyContract.json'* which stores the contracts ABIs and bytecode, and *'MyContractAddress.json'* which as the name implies will store the address of the deployed contract.
We would not manually update this files throughout our development as it will be updated automatically for us during contract deployment with truffle.

Next we have the *'hooks'* folder which contains the hooks used in our DApp. Hooks are a new addition to React in version 16.8 that allows you use state and other React features, like lifecycle methods, without writing a class.

finally we have the *'utils'* folder which will store helper functions and constant values.


### 2.2 Updating the UI

#### 2.1 Cover Page

Let us start by updating the code in the *'app.js'* file. Inside this file,
we are fetching the address of the user wallet that is connected to this dapp with the useContractKit() function.
```
  const {address} = useContractKit()
```

Next we perform a tenary statement to check if the address is not null or undefined and display the Counter component else it will display the Cover page which prompts the user to connect his wallet.


```
...
 return (
        <div className="App">
            <header className="App-header">
                {address ?
                    <Counter/>
                    :
                    <Cover/>
                }

            </header>
        </div>
    );
}
```

Firstly let us go ahead and customize the Cover component a bit. Navigate to *'components/Cover.js'* file.

Replace the imports with the following

```
import React from 'react';
import { Button } from "react-bootstrap";
import PropTypes from 'prop-types';
```


Next, we declare the Cover component and replace the code:


```
const Cover = ({ name, coverImg, connect }) => {
 if (name) {
   return (
     <div
         className="d-flex justify-content-center flex-column text-center "
         style={{ background: "#000", minHeight: "100vh" }}
       >
         <div className="mt-auto text-light mb-5">
           <div
             className=" ratio ratio-1x1 mx-auto mb-2"
             style={{ maxWidth: "320px" }}
           >
             <img src={coverImg} alt="" />
           </div>
           <h1>{name}</h1>
           <p>Please connect your wallet to continue.</p>
           <Button
             onClick={() => connect().catch((e) => console.log(e))}
             variant="outline-light"
             className="rounded-pill px-3 mt-3"
           >
             Connect Wallet
           </Button>
         </div>

         <p className="mt-auto text-secondary">Powered by Celo</p>
       </div>
   );
 }

 return null;
};
```


We are expecting three variables via props (name, coverImge, and connect):

Name: which is the name of the Dapp,

CoverImg : as you guessed, the cover image used in this DApp,

Connect: which is a function that prompts the user to connect his wallet to the DApp and is exported from Celo useContractKit which we will explore later on in this tutorial.

In case you don't know what props are read [here](https://reactjs.org/docs/components-and-props.html). React allows us to pass data down into other components using something we call props which is short for properties.

Above we check if the name prop exists  and if it does, we render the component.

We then create an onClick handler on the connect wallet button to trigger the connect function when clicked.

Finally, let us go ahead and declare our proptypes.


```
Cover.propTypes = {
 // props passed into this component
 name: PropTypes.string,
};

Cover.defaultProps = {
 name: '',
};

export default Cover;
```


In case you don't know what propTypes are, PropTypes are **a mechanism to ensure that components use the correct data type and pass the right data**, that components use the right type of props, and that receiving components receive the right type of props. [Click to know more](https://reactjs.org/docs/typechecking-with-proptypes.html).

Finally let us go back into our *'App.js'* file and replace the Cover code with the following:

```
    ...
   
   <Cover name="GEO Collection" coverImg={coverImg} connect={connect}/>
```
We need to import the coverImg into this file. Create a folder called *'img'* under the *'assets'* folder and save the cover image. You can get this image from the complete code on the Github repository. We then import it.

```
import coverImg from "./assets/img/nft_geo_cover.png";
```

Next we import the destroy function as well from useContractKit hook.
This function basically disconnects a user's wallet from the DApp when called.

 ```
 ...
 
 const App = function AppWrapper() {
   const {address, destroy, connect} = useContractKit()
 ```

### 2.2.2 Notification Library

In order to show users updates on functions happening under the hood in the DApp, we need to display notifications. We can import the Notification component that exist in the boilerplate code into our *'App.js'* file.

```
...
import { Notification } from "./components/ui/Notifications";
```

Next we render the notification component
```
... 

 return (
      <>
        <Notification/>
```


<!-- Next we remove the *'Counter'* component from the *'App.js'* file as we do not need it. We then  -->

### 2.2.3. Wallet


This component will format and display the connected user’s address.
Firstly, we import this component into our *'App.js'* file.

```
...

import Wallet from "./components/wallet";
```
Before we render this component, lets go ahead and remove the *'Counter'* component from our *'App.js'* file. Remove the import and also the code below from the body of this file.
```
...

    {address ?
        <Counter/>
```

We then replace the code above with our Wallet component
```
...

<Container fluid="md">
              <Nav className="justify-content-end pt-3 pb-5">
                <Nav.Item>

                  {/*display user wallet*/}
                  <Wallet
                      address={address}
                      amount={balance.CELO}
                      symbol="CELO"
                      destroy={destroy}
                  />
                </Nav.Item>
              </Nav>
            </Container>

```
In the code above we passed the Celo balance of the connected user wallet into the Wallet component via props. We dont have this data yet.
We get this data by using the useBalance hook from our boilerplate code.
Let us first import it into this file:
```
import React from "react";

...

import { useBalance } from "./hooks";
```
Next we call the hook before our return statement
```
  const {balance, getBalance} = useBalance();

```
This hook exports the balance of the connected wallet as well as a getBalance function to manually fetch the user's balance.


### 2.3. Mint and List NFTs.

In this section we will cover the processes of minting and listing NFTs.

Firstly, let us rename the *'dapp'* folder under the components folder to  *'minter'*, then create another folder called *'nfts'* under this folder.


### 2.4.1 Index.

Firstly, let us create a file called *'index.js'* under the nfts folder called *'index.js'*.

This folder will be responsible for displaying the add NFT and list NFT components.

Let us start by importing the necessary components and libraries.


```
import { useContractKit } from "@celo-tools/use-contractkit";
import React, { useEffect, useState, useCallback } from "react";
import { toast } from "react-toastify";
import PropTypes from "prop-types";
import AddNfts from "./Add";
import Nft from "./Card";
import Loader from "../../ui/Loader";
import { NotificationSuccess, NotificationError } from "../../ui/Notifications";
import {
 getNfts,
 createNft,
 fetchNftContractOwner,
} from "../../../utils/minter";
import { Row } from "react-bootstrap";
```


In this component, we will be using the useContractKit from the celo-tools library which exposes us to a ton of cool features needed for this DApp.

We also import the AddNfts and Nft component which we will create later on in the tutorial. We also imported some helper functions from the *'utils/minter'* which we will also create later on this tutorial.

Let us go ahead to declare the component :

This component expects one props which is the contract address of the NFT smart contract.


```
const NftList = ({minterContract}) => {

};
```


Next we use destructuring to get the performActions function and address from the useContracKit library. The performActions method will take care of displaying a modal to users telling them to confirm any actions on their connected wallet.

We also use some inbuilt react hooks like useState which helps to temporarily store data in the components state and useCallback which will return a memoized version of the callback that only changes if one of the dependencies has changed, in our case, the minterContract.

Finally, we then create a couple of functions we will need in this component.


```
const NftList = ({minterContract, name}) => {

 /* performActions : used to run smart contract interactions in order
 *  address : fetch the address of the connected wallet
 */
 const {performActions, address} = useContractKit();
 const [nfts, setNfts] = useState([]);
 const [loading, setLoading] = useState(false);
 const [nftOwner, setNftOwner] = useState(null);

 const getAssets = useCallback(async () => {
   try {
     setLoading(true);

     // fetch all nfts from the smart contract
     const allNfts = await getNfts(minterContract);
     if (!allNfts) return
     setNfts(allNfts);
   } catch (error) {
     console.log({ error });
   } finally {
     setLoading(false);
   }
 }, [minterContract]);

 const addNft = async (data) => {
   try {
     setLoading(true);

     // create an nft functionality
     await createNft(minterContract, performActions, data);
     toast(<NotificationSuccess text="Updating NFT list...."/>);
     getAssets();
   } catch (error) {
     console.log({ error });
     toast(<NotificationError text="Failed to create an NFT." />);
   } finally {
     setLoading(false);
   }
 };

 const fetchContractOwner = useCallback(async (minterContract) => {

   // get the address that deployed the NFT contract
   const _address = await fetchNftContractOwner(minterContract);
   setNftOwner(_address);
 }, []);

 useEffect(() => {
   try {
     if (address && minterContract) {
       getAssets();
       fetchContractOwner(minterContract);
     }
   } catch (error) {
     console.log({ error });
   }
 }, [minterContract, address, getAssets, fetchContractOwner]);

};
```


Finally, let us go ahead and design the interface of this component. We basically check if the user address is available, which in our case means that the user’s wallet is connected. If that returns true, we list out the NFTs that belong to that account and also, if the owner of the NFT smart contract is the same address that is connected to the DApp, we display the AddNfts component which allows the user mint a new NFT.


```
if (address) {
 return (
   <>
     {!loading ? (
       <>
         <div className="d-flex justify-content-between align-items-center mb-4">
           <h1 className="fs-4 fw-bold mb-0">{name}</h1>

           {/* give the add NFT permission to user who deployed the NFT smart contract */}
           {nftOwner === address ? (
               <AddNfts save={addNft} address={address}/>
           ) : null}

         </div>
         <Row xs={1} sm={2} lg={3} className="g-3  mb-5 g-xl-4 g-xxl-5">

           {/* display all NFTs */}
           {nfts.map((_nft) => (
               <Nft
                   key={_nft.index}
                   nft={{
                     ..._nft,
                   }}
               />
           ))}
         </Row>
       </>
     ) : (
       <Loader />
     )}
   </>
 );
}
return null;
```


Finally, let us add some Proptypes to ensure our component receives the required props and export this component :


```
NftList.propTypes = {

 // props passed into this component
 minterContract: PropTypes.instanceOf(Object)
};

NftList.defaultProps = {
 minterContract: null,
};

export default NftList;
```


#### 2.4.2 Add NFT.

Let us go ahead and create a file under the *'nfts'* folder called *'Add.js'*. This file will contain the code for the interface through which users can mint NFTs to the blockchain.

Let us start by importing the necessary components for this file.


```
import React, { useState } from "react";
import PropTypes from "prop-types";
import { Button, Modal, Form, FloatingLabel } from "react-bootstrap";
import { uploadToIpfs } from "../../../utils/minter";
```


Above, we use a function called ‘uploadToIpfs’ which will be responsible for uploading an NFT metadata to [IPFS](https://ipfs.io/).

Next, we create some hard coded attributes which users can add when creating their NFTs.


```
// basic attributes that can be added to NFT
const COLORS = ["Red", "Green", "Blue", "Cyan", "Yellow", "Purple"];
const SHAPES = ["Circle", "Square", "Triangle"];
```


Next, we define our AddNft component and create some functions which we would use when creating an NFT.


```
const AddNfts = ({ save, address }) => {
 const [name, setName] = useState("");
 const [ipfsImage, setIpfsImage] = useState("");
 const [description, setDescription] = useState("");

 //store attributes of an NFT
 const [attributes, setAttributes] = useState([]);
 const [show, setShow] = useState(false);


 // check if all form data has been filled
 const isFormFilled = () =>
     name && ipfsImage && description && attributes.length > 2;

 // close the popup modal
 const handleClose = () => {
   setShow(false);
   setAttributes([]);
 };

 // display the popup modal
 const handleShow = () => setShow(true);

 // add an attribute to an NFT
 const setAttributesFunc = (e, trait_type) => {
   const {value} = e.target;
   const attributeObject = {
     trait_type,
     value,
   };
   const arr = attributes;

   // check if attribute already exists
   const index = arr.findIndex((el) => el.trait_type === trait_type);

   if (index >= 0) {

     // update the existing attribute
     arr[index] = {
       trait_type,
       value,
     };
     setAttributes(arr);
     return;
   }

   // add a new attribute
   setAttributes((oldArray) => [...oldArray, attributeObject]);
```


This component expects two props:

save : this is a function passed down into this component which is used to add an NFT to the Blockchain.

address : this is the address of the user that is connected to the DApp.


We use proptypes to ensure that this data is passed down to this component correctly else an error will be displayed.

Firstly, we use [useState](https://reactjs.org/docs/hooks-state.html) hook to temporarily store data in our components state when interacting with our DApp.

Next, we use the isFormFilled function above to ensure that the form is filled before allowing a user to mint an NFT.

Next, is the setAttributesFunc which allows us to set the attributes a user wants in an NFT. Keep in mind there are no limits to attributes an NFT  can have. For the sake of this tutorial we have the hardcoded attributes above.

Finally, we go ahead and finish up this component by adding the codes for the interface which users can interact with.


```
return (
   <>
     <Button
       onClick={handleShow}
       variant="dark"
       className="rounded-pill px-0"
       style={{ width: "38px" }}
     >
       <i className="bi bi-plus"></i>
     </Button>

     {/* Modal */}
     <Modal show={show} onHide={handleClose} centered>
       <Modal.Header closeButton>
         <Modal.Title>Create NFT</Modal.Title>
       </Modal.Header>

       <Modal.Body>
         <Form>
           <FloatingLabel
             controlId="inputLocation"
             label="Name"
             className="mb-3"
           >
             <Form.Control
               type="text"
               placeholder="Name of NFT"
               onChange={(e) => {
                 setName(e.target.value);
               }}
             />
           </FloatingLabel>

           <FloatingLabel
             controlId="inputDescription"
             label="Description"
             className="mb-3"
           >
             <Form.Control
               as="textarea"
               placeholder="description"
               style={{ height: "80px" }}
               onChange={(e) => {
                 setDescription(e.target.value);
               }}
             />
           </FloatingLabel>

           <Form.Control
             type="file"
             className={"mb-3"}
             onChange={async (e) => {
               const imageUrl = await uploadToIpfs(e);
               if (!imageUrl) {
                 alert("failed to upload image");
                 return;
               }
               setIpfsImage(imageUrl);
             }}
             placeholder="Product name"
           ></Form.Control>
           <Form.Label>
             <h5>Properties</h5>
           </Form.Label>
           <Form.Control
             as="select"
             className={"mb-3"}
             onChange={async (e) => {
               setAttributesFunc(e, "background");
             }}
             placeholder="Background"
           >
             <option hidden>Background</option>
             {COLORS.map((color) => (
               <option
                 key={`background-${color.toLowerCase()}`}
                 value={color.toLowerCase()}
               >
                 {color}
               </option>
             ))}
           </Form.Control>

           <Form.Control
             as="select"
             className={"mb-3"}
             onChange={async (e) => {
               setAttributesFunc(e, "color");
             }}
             placeholder="NFT Color"
           >
             <option hidden>Color</option>
             {COLORS.map((color) => (
               <option
                 key={`color-${color.toLowerCase()}`}
                 value={color.toLowerCase()}
               >
                 {color}
               </option>
             ))}
           </Form.Control>

           <Form.Control
             as="select"
             className={"mb-3"}
             onChange={async (e) => {
               setAttributesFunc(e, "shape");
             }}
             placeholder="NFT Shape"
           >
             <option hidden>Shape</option>
             {SHAPES.map((shape) => (
               <option
                 key={`shape-${shape.toLowerCase()}`}
                 value={shape.toLowerCase()}
               >
                 {shape}
               </option>
             ))}
           </Form.Control>
         </Form>
       </Modal.Body>

       <Modal.Footer>
         <Button variant="outline-secondary" onClick={handleClose}>
           Close
         </Button>
         <Button
           variant="dark"
           disabled={!isFormFilled()}
           onClick={() => {
             save({
               name,
               ipfsImage,
               description,
               ownerAddress: address,
               attributes,
             });
             handleClose();
           }}
         >
           Create NFT
         </Button>
       </Modal.Footer>
     </Modal>
   </>
 );
};

```


Above we use the onChange property to update the various states with the data the user enters into the Form.

When a user then clicks the Create NFT button, the save function which was passed down as props is called with the data the user entered into the form.

Finally, let us add the required proptypes for this component and export it :


```
AddNfts.propTypes = {

 // props passed into this component
 save: PropTypes.func.isRequired,
 address: PropTypes.string.isRequired,
};

export default AddNfts;
```


#### 2.4.3 List NFTs.

Let us go ahead and create the Nft component which we imported in our *'index.js'*	file. This component will contain the design in which our NFT will be displayed to the users.

Firstly let us create a file under the _nfts_ folder called *‘Card.js’*. Next we go ahead to import the necessary libraries.


```
import React from "react";
import PropTypes from "prop-types";
import { Card, Col, Badge, Stack, Row } from "react-bootstrap";
import { truncateAddress } from "../../../utils";
import Identicon from "../../ui/Identicon";
```


Next we declare our NftCard component which expects an nft as props.


```
const NftCard = ({ nft }) => {
 const { image, description, owner, name, index, attributes } = nft;
```

We use destructuring above to get the metadata of the NFT from the data passed into this component as props.

Let us create the UI and display the NFT data.

```
return (
 <Col key={index}>
   <Card className=" h-100">
     <Card.Header>
       <Stack direction="horizontal" gap={2}>
         <Identicon address={owner} size={28} />
         <span className="font-monospace text-secondary">
           {truncateAddress(owner)}
         </span>
         <Badge bg="secondary" className="ms-auto">
           {index} ID
         </Badge>
       </Stack>
     </Card.Header>

     <div className=" ratio ratio-4x3">
       <img src={image} alt={description} style={{ objectFit: "cover" }} />
     </div>

     <Card.Body className="d-flex  flex-column text-center">
       <Card.Title>{name}</Card.Title>
       <Card.Text className="flex-grow-1">{description}</Card.Text>
       <div>
         <Row className="mt-2">
           {attributes.map((attribute, key) => (
             <Col key={key}>
               <div className="border rounded bg-light">
                 <div className="text-secondary fw-lighter small text-capitalize">
                   {attribute.trait_type}
                 </div>
                 <div className="text-secondary text-capitalize font-monospace">
                   {attribute.value}
                 </div>
               </div>
             </Col>
           ))}
         </Row>
       </div>
     </Card.Body>
   </Card>
 </Col>
);
};
```


Finally, we declare our proptypes to ensure the right data is passed into this component.


```
NftCard.propTypes = {

 // props passed into this component
 nft: PropTypes.instanceOf(Object).isRequired,
};

export default NftCard;
```


## 3. Utils


### 3.1 Minter.

Let us go ahead and rename the *'dapp.js'* file under the *'utils'* folder to *'minter.js'* . Basically, this file will contain all the login we use in creating and fetching our NFTs.

First we import the libraries we need for this file.


```
import {create as ipfsHttpClient} from "ipfs-http-client";
import axios from "axios";
```


The *'ipfs-http-client'* library is basically a wrapper which enables us to interact with the IPFS node. This is necessary as we do not save NFT data like attributes and other generic data on the blockchain directly, but instead save those data to IPFS and then save the hash returned by IPFS on the blockchain.

For those who don't know what IPFS is, IPFS is a decentralized file-sharing system that can be leveraged to more efficiently store and share large files.

Next, we import axios which basically allows us to make calls to APIs in order to fetch or save data.

Let us go ahead and initialize our IPFS client which we will use to interact with IPFS


```
// initialize IPFS
const client = ipfsHttpClient("https://ipfs.infura.io:5001/api/v0");
```


Next, we create a function that enables us to mint an NFT and save its metadata to IPFS.


```
// mint an NFT
export const createNft = async (
   minterContract,
   performActions,
   {name, description, ipfsImage, ownerAddress, attributes}
) => {
   await performActions(async (kit) => {
       if (!name || !description || !ipfsImage) return;
       const {defaultAccount} = kit;

       // convert NFT metadata to JSON format
       const data = JSON.stringify({
           name,
           description,
           image: ipfsImage,
           owner: defaultAccount,
           attributes,
       });

       try {

           // save NFT metadata to IPFS
           const added = await client.add(data);

           // IPFS url for uploaded metadata
           const url = `https://ipfs.infura.io/ipfs/${added.path}`;

           // mint the NFT and save the IPFS url to the blockchain
           let transaction = await minterContract.methods
               .safeMint(ownerAddress, url)
               .send({from: defaultAccount});

           return transaction;
       } catch (error) {
           console.log("Error uploading file: ", error);
       }
   });
};
```


Let us break down this function a bit.

Our function above expects 3 parameters:

minterContract : this is an RPC which enables us to interact with our smart contract using the smart contract ABI and the contract’s address. We would look more into this when discussing hooks.

performActions : as we discussed earlier, is exported from useContractKit and enables us to run a set of transactions in an atomic manner and displays a loader modal to users while they interact with their wallets.

Lastly, the functions expect the NFT data. In this case, we are using destructuring to directly get the metadata of the NFT


```
{name, description, ipfsImage, ownerAddress, attributes}
```


Next we use the performActions to run our transaction.

We then ensure the metadata of our NFT exists else we stop the function.

Also we can get the connected user’s address using destructuring from the kit which performActions function exports to us.


```
await performActions(async (kit) => {
   if (!name || !description || !ipfsImage) return;
   const {defaultAccount} = kit;
```


Next we convert our data to a JSON format before saving it to IPFS.


```
// convert NFT metadata to JSON format
const data = JSON.stringify({
   name,
   description,
   image: ipfsImage,
   owner: defaultAccount,
   attributes,
});
```


Finally, we save this data to IPFS and mint our NFT. We use the client we initialized at the beginning of this file and call the add method on it which saves the data to IPFS and returns the hash of the file. Next we use the default base url for IPFS and concatenate the hash we got. This gives us a link to our file on IPFS. We go ahead and call the safeMint function from our minterContract with the address of the minter and the IPFS url as the url of the NFT.


```
try {

   // save NFT metadata to IPFS
   const added = await client.add(data);

   // IPFS url for uploaded metadata
   const url = `https://ipfs.infura.io/ipfs/${added.path}`;

   // mint the NFT and save the IPFS url to the blockchain
   let transaction = await minterContract.methods
       .safeMint(ownerAddress, url)
       .send({from: defaultAccount});

   return transaction;
} catch (error) {
   console.log("Error uploading file: ", error);
}
```


Next we create another function called uploadToIPFS which enables us to save a file from our local files to IPFS.


```
// function to upload a file to IPFS
export const uploadToIpfs = async (e) => {
   const file = e.target.files[0];
   if (!file) return;
   try {
       const added = await client.add(file, {
           progress: (prog) => console.log(`received: ${prog}`),
       });
       return `https://ipfs.infura.io/ipfs/${added.path}`;
   } catch (error) {
       console.log("Error uploading file: ", error);
   }
};
```


Next, let us go ahead and create another function called *‘fetchNftMeta’* which returns the metadata of an NFT from IPFS.


```
// get the metedata for an NFT from IPFS
export const fetchNftMeta = async (ipfsUrl) => {
   try {
       if (!ipfsUrl) return null;
       const meta = await axios.get(ipfsUrl);
       return meta;
   } catch (e) {
       console.log({e});
   }
};
```


Let us create 2 more functions

*‘FetchNFTOwner’*, as the name implies, this fetches the address of the person that owns a particular NFT.


```
// get the owner address of an NFT
export const fetchNftOwner = async (minterContract, index) => {
   try {
       return await minterContract.methods.ownerOf(index).call();
   } catch (e) {
       console.log({e});
   }
};
```


*'FetchNftContractOwner'* returns the address that deployed the NFT smart contract.


```
// get the address that deployed the NFT contract
export const fetchNftContractOwner = async (minterContract) => {
   try {
       let owner = await minterContract.methods.owner().call();
       return owner;
   } catch (e) {
       console.log({e});
   }
};
```


Finally, the last function is the *'getNfts'* function that fetches all the NFTs on the smart contract.

Firstly, you need to know how many NFTs are stored in the contract, so you can iterate over them. The nftsLength variable will store the number of NFTS and we use totalSupply() function from your contract and assign its value.

Next, declare an empty array for the NFT objects.


```
// fetch all NFTs on the smart contract
export const getNfts = async (minterContract) => {
   try {
       const nfts = [];
       const nftsLength = await minterContract.methods.totalSupply().call();
```


Now, loop over the length of your NFTs. For each NFT, we call the tokenURI  function to get the IPFS hash of the NFT.

Next we use the hash to get the metadata from IPFS using the fetchNFTMeta function and also get the owner of the NFT using the fetchNftOwner function we created earlier.

Resolve the promise with your NFT data and then push your nft object to your nfts array.


```
  for (let i = 0; i < Number(nftsLength); i++) {
           const nft = new Promise(async (resolve) => {
               const res = await minterContract.methods.tokenURI(i).call();
               const meta = await fetchNftMeta(res);
               const owner = await fetchNftOwner(minterContract, i);
               resolve({
                   index: i,
                   owner,
                   name: meta.data.name,
                   image: meta.data.image,
                   description: meta.data.description,
                   attributes: meta.data.attributes,
               });
           });
           nfts.push(nft);
       }
       return Promise.all(nfts);
   } catch (e) {
       console.log({e});
   }
};
```


With that we are done with our helper functions!.

## 4. Hooks


### 4.1 useMinterContract.

Finally, let us create our last hook called useMinterContract. This hook is basically going to call the useContract hook and return the RPC.

Let us rename the *'useDappContract.js'* under the *'hooks'* folder to *'useMinterContract'*.

We will import the NFT smart contract ABI and contract address.


```
import {useContract} from './useContract';
import MyNFTAbi from '../contracts/MyNFT.json';
import MyNFTContractAddress from '../contracts/MyNFT-address.json';


// export interface for NFT contract
export const useMinterContract = () => useContract(MyNFTAbi.abi, MyNFTContractAddress.MyNFT);
```


Finally, lets update the *'index.js'* to export our updated file


```
export * from './useMinterContract';

...
export * from './useContract';
```


## 5. Piecing It All Together.

Finally, we are done with all our hooks, helper functions and UI components, we can piece everything together and display our NFT components.

Let us go back to our *'App.js'* and update the code there.


Next we check if the user wallet is connected to the DApp and display the NFTs and Wallet component

Firstly we import the the useMinterContract hook and Nfts component into our *'App.js'*

```
...

import Wallet from "./components/wallet";
import { useBalance, useMinterContract } from "./hooks";
import Nfts from "./components/minter/nfts";

...

```

Secondly we add the *'Nfts'* component we just imported into our component. Render the component just before the closing Container tag
```
...
       <main>

                {/*list NFTs*/}
                <Nfts
                    name="GEO Collection"
                    updateBalance={getBalance}
                    minterContract={minterContract}
                />
              </main>
            </Container>
            
.....
```


## 6. Deploy your DApp to Netlify

Now you can get a React project up and running with a few commands, and in less than 30 seconds you can have it deployed with Netlify.

Make sure you create a netlify account on https://www.netlify.com/.

You can use this one click solution to deploy your react app by clicking [here](https://app.netlify.com/start/deploy?repository=https://github.com/netlify/create-react-app-lambda)

If you are using the netlify CLI, first install netlify globally via your terminal.


```
npm install netlify-cli -g
```


You can then build your project and run the deploy script


```
npm run build
netlify deploy
```


Next, follow command line prompts and choose yes for new project and ./build as your deploy folder and voila you have a production React app!

If that was too hard to follow, here is a gif of me doing it:


![alt_text](../images/image1.gif "image_tooltip")


You can also link to a GitHub repo for [continuous deployment](https://www.netlify.com/docs/continuous-deployment) for specified branches and will grant you a nifty [Deploy Preview](https://www.netlify.com/blog/2016/07/20/introducing-deploy-previews-in-netlify/) feature.

If you choose to use something for routing (like React Router for example), you will need to set up a [redirect and rewrite rule](https://www.netlify.com/docs/redirects) for the single page app.

That redirect rule would look like this:


```
/*    /index.html   200
```


This redirect rule above will serve the index.html instead of giving a 404 no matter what URL the browser requests.

You can add redirect rules to the `_redirects` file or to your `netlify.toml` config file. For more information on redirects on Netlify [checkout the docs](https://www.netlify.com/docs/redirects).
