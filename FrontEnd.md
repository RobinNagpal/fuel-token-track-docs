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
