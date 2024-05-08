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
2. fuels (v0.82.0): A TypeScript SDK that simplifies interacting with Fuel contracts from the browser. 
3. Fuel Browser

Important: While using these specific versions is recommended for this guide, always double-check compatibility when working on your own projects. Refer to the official documentation for the latest supported versions.

# Initializing the React App
We'll start by initializing the React application. Assuming you are currently at `fuel-project/token-track`, go back up one directory:

```
cd ..
```

Now create a new React project

```
npx create-react-app frontend  --template typescript
cd frontend
```
Next up, we'll install the fuels SDK

```
npm install fuels@0.82.0 @fuels/react@0.18.0 @fuels/connectors@0.2.2 @tanstack/react-query@5.28.9
```

# Generating contract types
(same as the docs)

# Connecting Your Frontend to Fuel: Setting Up the Environment
Now that you have a basic React application structure, let's integrate the necessary components to enable interaction with your Fuel smart contract. This section focuses on setting up the environment within your index.tsx file.

1. Importing Required Modules:

At the beginning of your frontend/src/index.tsx file, you'll need to import the following modules:

```
import { FuelProvider } from '@fuels/react';
import {
  FuelWalletConnector,
  FuelWalletDevelopmentConnector,
  FueletWalletConnector,
} from '@fuels/connectors';
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
```

- FuelProvider: This component from @fuels/react wraps your entire application and provides the context for interacting with the Fuel network.
- Wallet Connectors: These classes (imported from @fuels/connectors) facilitate connections between your application and different wallet types, allowing users to manage their FUEL tokens and interact with the contract.
- QueryClient and QueryClientProvider: These components from @tanstack/react-query establish a context for managing data fetching and caching within your React application.

2. Creating a Query Client:
Create a new instance of the QueryClient class:

```
const queryClient = new QueryClient();
```

This query client will assist with managing data fetching related to your smart contract interactions in your React application.

3. Wrapping Your App with Providers:
Next, modify the structure of your index.tsx file to wrap your main application component (App) with the FuelProvider and QueryClientProvider components. Here's the updated structure:

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

- `FuelProvider:` This component wraps your App component and establishes the Fuel network connection context. Inside the FuelProvider, you define a configuration object (fuelConfig) that specifies how your application will connect to the network and interact with wallets
  - The connectors property is an array containing instances of different wallet connector classes. These connectors allow users to connect their Fuel wallets to your application.

Next up, we'll define the code and functions to interact with the contract.

Let's establish a connection between your React application and your deployed smart contract. This connection will allow your application to call your contract's functions and retrieve data from the blockchain. The following code snippet demonstrates how we achieve this connection using the `useEffect` hook:

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


Now to call your `mint_to_address` function in your contract, we have handleMintToAddress function, which serves as the bridge between your user interface and the mint_to_address function defined in your Fuel smart contract.

## Functionality breakdown
1. Contract Check
```
if (!contract) {
    return alert("Contract not loaded");
}
```

- Before proceeding, the function ensures a successful connection to your Fuel smart contract. It checks if the contract object exists. If not, it displays an alert message informing the user that they cannot mint tokens yet.

2. Extracting Mint Address
```
const address = Address.fromString(mintTo);
const addressInput = { value: address.toB256() };
```

- This section retrieves the mint address value from your application's state, stored in a variable named mintTo.
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

- The core functionality happens here. It utilizes the contract object to access the mint_to_address function defined in your smart contract.
- Arguments for the function call are constructed:
  - addressInput: The prepared address object containing the converted mint address.
  - Number(mintAmount): The user-provided mint amount, converted to a number for the contract call.
- Additionally, transaction parameters are set:
  - gasPrice: A fixed gas price (you might want to consider making this dynamic based on network conditions).
  - gasLimit: A maximum amount of gas allowed for this transaction (adjust this based on your contract's requirements).
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

In essence, the `handleMintToAddress` function acts as the intermediary between your user interface and your smart contract. A similar kind of approach is followed by all other functions.

## User Interface for Token Interactions
This section delves into the user interface (UI) components that allow users to interact with your token functionalities. The provided code snippet showcases a form with various fields and buttons to enable minting, transferring, and burning tokens.

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
- A dropdown menu allows you to specify where to burn tokens from (Address or Contract).
- The "Burn Tokens" button triggers the `handleBurnFromAddress` function to initiate the burning process.