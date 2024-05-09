# FrontEnd

## ToDo

- A brief overview of the tools and technologies we will be using for the front-end.
  - Fuel browser wallet
  - fuels SDK
  - Typings
  - React
- Mention the versions of the tools which are being used in the quickstart guide to avoid any incompatibility.
- Instructions to set up the required frontend library
- Explanation about the fuels sdk and why is it required, also explain the modifications in the index.tsx file, the providers and other changes
- Instructions to generate contract types for type safety. There should be an explanation of the output files, what content do they contain.
  - fuels-config.ts
  - sway-api folder
- A step-by-step guide to explaining the frontend code.

### Installing the Browser Wallet

- A guide explaining the installation of wallet extension in the browser.
- Instructions telling about transferring test coins using the faucet.

### UI Functionality

### Walk Through of UI

- Walk the user through different functionalities of the application like minting, burning and transferring of the tokens.

---

Now that we have a contract built and deployed to the Fuel testnet, we now need a UI to interact with that contract. Fuel provide a typescript SDK to interact with the Fuel Blockchain. So we will be creating a frontend with the following tools:

1. React (v18.3.1): A JavaScript library for building user interfaces.
2. fuels (v0.84.0): A TypeScript SDK that simplifies interacting with Fuel contracts from the browser.
3. Fuel Browser

Important: While using these specific versions is recommended for this guide, always double-check compatibility when working on your own projects. Refer to the official documentation for the latest supported versions.

# Initializing the React App

We'll start by initializing the React application. Assuming we are currently at `contracts/TokenTrack`, go back up two directories:

```
cd ../..
```

Now create a new React project

```
npx create-react-app frontend  --template typescript
cd frontend
```

Next up, we'll install the fuels SDK

```
npm install fuels@0.84.0 @fuels/react@0.18.0 @fuels/connectors@0.2.2 @tanstack/react-query@5.28.9
```

# Generating contract types

The fuels init command generates a `fuels.config.ts` file that is used by the SDK to generate contract types. Use the contracts flag to define where the contract folder is located, and the output flag to define where we want the generated files to be created.

```
npx fuels@0.84.0 init --contracts ../contracts/TokenTrack/ --output ./src/sway-contracts-api
```

Now that we have a fuels.config.ts file, we can use the fuels build command to rebuild our contract and generate types. Running this command will interpret the output ABI JSON from our contract and generate the correct TypeScript definitions.

Inside the `/frontend` directory run:

```
npx fuels@0.84.0 build
```

This will create a folder named `sway-contract-api` which the following files and folders:

```
.
├── contracts
│   ├── TokenTrackAbi.d.ts
│   ├── TokenTrackAbi.hex.ts
│   ├── common.d.ts
│   ├── factories
│   │   └── TokenTrackAbi__factory.ts
│   └── index.ts
└── index.ts
```

Among these generated files, there are two important classes:

## 1. TokenTrackAbi

The TokenTrackAbi class serves as a typed interface specifically designed to interact with our TokenTrackAbi smart contract.
The functions property within the class is the heart of this typed interface. It defines each function available in our smart contract, along with the data it expects:

- Function Names: These act as labels, mirroring the actual function names in our contract (e.g., burn_from_address, mint_to_address).
- Function Arguments: Each function has an array listing its arguments. These arguments come with specific types (like AddressInput, BigNumberish). These types ensure we provide data that matches what the contract expects.
- Return Types: Similar to arguments, functions can return values. The return type specifies what kind of data the function call might return (e.g., void for functions that don't return anything, BN for functions returning a big number).

### Benefits of typed interface

- Code Clarity: By using this typed interface, our code becomes much easier to understand. We can see at a glance what data each function requires and what it might return.
- Error Prevention: The type system helps catch errors early on. If we try to use incompatible data, the development tools might warn us before we even run the code. This saves time and frustration!

## 2. TokenTrackAbi\_\_factory

TokenTrackAbi**factory class deals with interacting with our TokenTrackAbi smart contract on the Fuel network. Following we explain some of the key components of the TokenTrackAbi**factory class:

### ABI and Storage Slots:

- **static readonly abi = \_abi:**  
  This line stores the compiled ABI (Application Binary Interface) of our smart contract. The ABI defines the functions and variables exposed by our contract, allowing our frontend code to interact with them.

- **static readonly storageSlots = \_storageSlots:**  
  This line holds an array representing the storage layout of our smart contract. This information is used for interacting with the contract's storage variables.

### Interface Creation (createInterface):

- **static createInterface(): TokenTrackAbiInterface:**  
  This method utilizes the stored ABI to generate a TypeScript interface named TokenTrackAbiInterface. This interface reflects the structure of our contract, providing type safety and code completion for interacting with our contract's functions in our frontend code.

### Contract Connection (connect):

- **static connect(id: string | AbstractAddress, accountOrProvider: Account | Provider): TokenTrackAbi:**  
  This method allows us to connect our frontend code to a deployed instance of our TokenTrackAbi contract on the Fuel network.

### Contract Deployment (deployContract):

- **static async deployContract(bytecode: BytesLike, wallet: Account, options: DeployContractOptions = {}): Promise<TokenTrackAbi>:**  
  This asynchronous method is where the deployment functionality comes in. It allows us to deploy a new instance of our TokenTrackAbi contract:

  - bytecode: This argument holds the raw bytecode of our smart contract, compiled for deployment on the Fuel network.
  - wallet: This refers to the Fuel account or wallet we want to use for deploying the contract (with sufficient funds for gas costs).
  - options: This optional object let us specify additional deployment parameters like gas price or gas limit.

The method uses a ContractFactory object to handle the deployment process. It provides the contract's bytecode, ABI, wallet information, and the storageSlots to ensure proper storage layout for the new contract.

Finally, it deploys the contract and returns a promise that resolves to a TokenTrackAbi object representing the newly deployed instance.

# Connecting Our Frontend to Fuel: Setting Up the Environment

Now that we have a basic React application structure, let's integrate the necessary components to enable interaction with our Fuel smart contract. This section focuses on setting up the environment within our index.tsx file.

1. Importing Required Modules:

At the beginning of the frontend/src/index.tsx file, we'll need to import the following modules:

```
import { FuelProvider } from '@fuels/react';
import {
  FuelWalletConnector,
  FuelWalletDevelopmentConnector,
  FueletWalletConnector,
} from '@fuels/connectors';
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
```

- FuelProvider: This component from @fuels/react wraps our entire application and provides the context for interacting with the Fuel network.
- Wallet Connectors: These classes (imported from @fuels/connectors) facilitate connections between our application and different wallet types, allowing users to manage their FUEL tokens and interact with the contract.
- QueryClient and QueryClientProvider: These components from @tanstack/react-query establish a context for managing data fetching and caching within our React application.

2. Creating a Query Client:
   Create a new instance of the QueryClient class:

```
const queryClient = new QueryClient();
```

This query client will assist with managing data fetching related to our smart contract interactions in our React application.

3. Wrapping the App with Providers:
   Next, modify the structure of the index.tsx file to wrap the main application component (App) with the FuelProvider and QueryClientProvider components. Here's the updated structure:

```
<QueryClientProvider client={queryClient}>
  <FuelProvider
    fuelConfig={{
      connectors: [
        new FuelWalletConnector(),
        new FuelWalletDevelopmentConnector(),
        new FueletWalletConnector(),
      ],
    }}
  >
    <App />
  </FuelProvider>
</QueryClientProvider>
```

- `QueryClientProvider:` This component wraps the entire application and provides the queryClient context for data fetching.

- `FuelProvider:` This component wraps our App component and establishes the Fuel network connection context. Inside the FuelProvider, we define a configuration object (fuelConfig) that specifies how our application will connect to the network and interact with wallets
  - The connectors property is an array containing instances of different wallet connector classes. These connectors allow users to connect their Fuel wallets to the application.

Next up, we'll define the code and functions to interact with the contract.

Let's establish a connection between the React application and our deployed smart contract. This connection will allow the application to call our contract's functions and retrieve data from the blockchain. The following code snippet demonstrates how we achieve this connection using the `useEffect` hook:

```
  useEffect(() => {
    async function connectContract() {
      if (isConnected && wallet) {
        const TokenContract = TokenTrackAbi__factory.connect(
          CONTRACT_ID,
          wallet
        );
        setContract(TokenContract);
      }
    }

    connectContract();
  }, [isConnected, wallet]);
```

Now to call the `mint_to_address` function in our contract, we have handleMintToAddress function, which serves as the bridge between our user interface and the `mint_to_address` function defined in our Fuel smart contract.

## Functionality breakdown

1. Contract Check

```
if (!contract) {
    return alert("Contract not loaded");
}
```

- Before proceeding, the function ensures a successful connection to our Fuel smart contract. It checks if the contract object exists. If not, it displays an alert message informing the user that they cannot mint tokens yet.

2. Extracting Mint Address

```
const address = Address.fromString(mintTo);
const addressInput = { value: address.toB256() };
```

- This section retrieves the mint address value from our application's state, stored in a variable named mintTo.
- It utilizes the Address class from fuels to convert the user-provided address string into the Address object.
- The converted address is then wrapped in an object (addressInput) to form `AddressInput` type.

3. Calling the Contract Function:

```
await contract.functions
    .mint_to_address(addressInput, Number(mintAmount))
    .txParams({
        gasPrice: 1,
        gasLimit: 1_000_000,
    })
    .call();
```

- The core functionality happens here. It utilizes the contract object to access the `mint_to_address` function defined in our smart contract.
- Arguments for the function call are constructed:
  - addressInput: The prepared address object containing the converted mint address.
  - Number(mintAmount): The user-provided mint amount, converted to a number for the contract call.
- Additionally, transaction parameters are set:
  - gasPrice: A fixed gas price.
  - gasLimit: A maximum amount of gas allowed for this transaction.
- Finally, the .call() method executes the transaction, sending the mint request to the Fuel network.

4. Error Handling:

```
catch (error) {
    console.error(error);
}
```

- The function includes a `try...catch` block to handle any potential errors that might occur during the transaction, such as insufficient funds or invalid inputs. Errors are logged to the console for debugging purposes.

5. Resetting Input Values:

```
setMintTo("");
setMintAmount("");
```

- After successful execution (or catching an error), the function resets the user input fields for mintTo and mintAmount to an empty state, allowing the user to initiate a new mint request.

In essence, the `handleMintToAddress` function acts as the intermediary between the user interface and our smart contract. A similar kind of approach is followed by all other functions.

## User Interface for Token Interactions

This section delves into the user interface (UI) components that allow users to interact with our token functionalities. The provided code snippet showcases a form with various fields and buttons to enable minting, transferring, and burning tokens.

### Form Breakdown:

The form is divided into three sections: Minting, Transferring, and Burning. Each section allows users to specify the desired action and provides input fields for relevant data.

#### Minting Section:

- This section allows users to mint new tokens and send them to a specified address.
- Users can enter the recipient's address in the `mintTo` input field.
- The mintAmount input field lets users specify the number of tokens they want to mint.
- A dropdown menu (mintType) allows users to potentially choose between minting to Address and minting to Contract.
- The "Mint New Tokens" button triggers the handleMintToAddress function to initiate the minting process.

#### Transferring Section:

- This section enables users to transfer existing tokens from their wallet to another address.
- Users provide the recipient's address in the transferTo input field.
- The transferAmount input field specifies the number of tokens to transfer.
- The "Transfer Tokens" button triggers the `handleTransferToAddress` function to execute the transfer transaction.

#### Burning Section:

- This section allows users to burn (destroy) a specified amount of tokens from their wallet.
- The burnAddress input field might be pre-filled with a "burn" address where tokens are sent to be effectively removed from circulation.
- The burnAmount input field specifies the number of tokens to burn.
- A dropdown menu allows users to specify where to burn tokens from (Address or Contract).
- The "Burn Tokens" button triggers the `handleBurnFromAddress` function to initiate the burning process.

# Install the Fuel Browser Wallet

Our frontend application will allow users to connect with a wallet, so you'll need to have a browser wallet installed.
Install the Fuel Wallet [here](https://chromewebstore.google.com/detail/fuel-wallet/dldjpboieedgcmpkchcjcbijingjcgok)

# Running the project

Inside the `/frontend` directory run:

```
npm start
```
