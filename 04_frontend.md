# Initializing the React App

Let's begin by setting up the React application. Create a new React project in the root directory:

```shell
npx create-react-app frontend --template typescript
cd frontend
```

Next, install the fuels SDK:

```shell
npm install fuels@0.82.0 @fuels/react@0.18.0 @fuels/connectors@0.2.2 @tanstack/react-query@5.28.9
```

# Generating Contract Types

We have already added the configuration for fuels in the `fuels.config.ts` file. To generate the contract types, run the following command:

```shell
cd ..
npx fuels@0.82.0 build
```

This command generates the contract types in the path specified in the `fuels.config.ts` file, which is `frontend/src/sway-contracts-api` in our case.

The generated file structure would look like this:

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

## 1. `TokenTrackAbi`

The `TokenTrackAbi` class acts as a typed interface designed to interact with our TokenTrackAbi smart contract. It
provides details about each function in our smart contract, including:

- **Function Names**: These serve as identifiers that correspond to the actual names in our contract, such as `burn_from_address`, `mint_to_address`.
- **Function Arguments**: Each function lists its required arguments with specific types like `AddressInput` or `BigNumberish` to ensure data compatibility.
- **Return Types**: This indicates the type of data a function may return, such as void for no return value or BN for returning a large number.

#### Benefits of Typed Interface

- **Code Clarity:** The typed interface simplifies understanding of what data each function requires and its potential return type.
- **Error Prevention:** The type system helps identify and prevent errors by ensuring data compatibility before runtime, saving time and reducing frustrations.

## 2. `TokenTrackAbi__factory`

The `TokenTrackAbi__factory` class manages interactions with the `TokenTrackAbi` smart contract on the Fuel network. It includes:

#### ABI and Storage Slots:

- `static readonly abi = _abi` : This line holds the ABI of our smart contract, detailing the functions and variables it exposes.
- `static readonly storageSlots = _storageSlots` : This array contains the storage layout of our smart contract.

#### Interface Creation (createInterface):

`static createInterface(): TokenTrackAbiInterface`: This method generates a TypeScript interface that reflects our contract's structure, enhancing type safety and enabling code completion.

#### Contract Connection (connect):

`static connect(id: string | AbstractAddress, accountOrProvider: Account | Provider): TokenTrackAbi:`: This function connects our frontend code to a deployed TokenTrackAbi contract on the Fuel network.

#### Contract Deployment (deployContract):

`static async deployContract(bytecode: BytesLike, wallet: Account, options: DeployContractOptions = {}): Promise<TokenTrackAbi>`: This method deploys a new instance of our contract. It takes the contract's bytecode, ABI, wallet details, and the storageSlots, ensuring correct storage layout. The method returns a promise resolving to a TokenTrackAbi object for the newly deployed instance.

# Connecting Our Frontend to Fuel: Setting Up the Environment

Now that we have a basic React application structure, let's integrate the necessary components to interact with our Fuel smart contract. This section will focus on setting up the environment within our index.tsx file.

1. **Importing Required Modules:**

At the beginning of the frontend/src/index.tsx file, we need to import the following modules:

```typescript
import { FuelProvider } from "@fuels/react";
import {
  FuelWalletConnector,
  FuelWalletDevelopmentConnector,
  FueletWalletConnector,
} from "@fuels/connectors";
import { QueryClient, QueryClientProvider } from "@tanstack/react-query";
```

- `FuelProvider`: This component from @fuels/react wraps our entire application and provides the context for interacting with the Fuel network.
- `Wallet Connectors`: These classes (imported from @fuels/connectors) help connect our application with different wallet types, allowing users to manage their FUEL tokens and interact with the contract.
- `QueryClient` and `QueryClientProvider`: These components from @tanstack/react-query set up a context for managing data fetching and caching within our React application.

2. **Creating a Query Client:**

   Create a new instance of the QueryClient class:

```typescript
const queryClient = new QueryClient();
```

This query client will help manage data fetching related to our smart contract interactions in our React application.

3. **Wrapping the App with Providers:**

   Next, modify the structure of the index.tsx file to wrap the main application component (App) with the FuelProvider and `QueryClientProvider` components. Here's the updated structure:

```typescript jsx
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

- `QueryClientProvider`: This component wraps the entire application and provides the queryClient context for data fetching.
- `FuelProvider`: This component wraps our App component and sets up the context for connecting to the Fuel network. Inside the `FuelProvider`, we define a configuration object (fuelConfig) that outlines how our application will connect to the network and interact with wallets.
  - The `connectors` property is an array containing instances of different wallet connector classes. These connectors enable users to connect their Fuel wallets to the application.

Now we'll create hooks for our wallet. Create a folder for holding all the customs hooks and create new file for each hook in it.

```shell
mkdir src/hooks
```

There are two types of wallets:

1. Burner Wallet: A Burner wallet is embedded inside of the template app and stored in local storage.
2. Browser Wallet: A wallet connected via a browser extension like the Fuel Wallet.

The following code defines a hook `useActiveWallet` which manages the active wallet within your Fuel Dapp:

```typescript
import { useEffect, useState } from "react";
import { useBrowserWallet } from "./useBrowserWallet";
import { useBurnerWallet } from "./useBurnerWallet";
import { AppWallet } from "../lib";

type WalletTypes = "burner" | "browser";

export const useActiveWallet = (): AppWallet => {
  const {
    wallet: burnerWallet,
    walletBalance: burnerWalletBalance,
    refreshWalletBalance: refreshBurnerWalletBalance,
  } = useBurnerWallet();
  const {
    wallet: browserWallet,
    walletBalance: browserWalletBalance,
    refreshWalletBalance: refreshBrowserWalletBalance,
    isConnected: isBrowserWalletConnected,
  } = useBrowserWallet();

  const [activeWallet, setActiveWallet] = useState<WalletTypes>("burner");

  useEffect(() => {
    if (isBrowserWalletConnected) {
      setActiveWallet("browser");
      refreshBrowserWalletBalance?.();
    } else {
      setActiveWallet("burner");
      refreshBurnerWalletBalance?.();
    }
  }, [isBrowserWalletConnected]);

  return {
    wallet: activeWallet === "browser" ? browserWallet : burnerWallet,
    walletBalance:
      activeWallet === "browser" ? browserWalletBalance : burnerWalletBalance,
    refreshWalletBalance:
      activeWallet == "browser"
        ? refreshBrowserWalletBalance
        : refreshBurnerWalletBalance,
  };
};
```

useActiveWallet leverages two sub-hooks, useBurnerWallet and useBrowserWallet, to handle both burner and browser-connected wallets seamlessly. It returns an object containing the following properties:

- wallet: The currently active wallet object (either burnerWallet or browserWallet).
- walletBalance: The balance of the currently active wallet.
- refreshWalletBalance: A function to refresh the balance of the active wallet.

## Functionality:

- The hook initially sets the active wallet to "burner".
- It uses a useEffect hook to monitor the isConnected state from useBrowserWallet.
- If a browser wallet is connected (isConnected is true), the active wallet switches to "browser" and its balance is refreshed.
- If no browser wallet is connected, the active wallet remains "burner" and its balance is refreshed.

Next, let's establish a connection between the React application and our deployed smart contract in the `App.tsx` file. This connection will allow the application to call our contract's functions and retrieve data from the blockchain. The following code snippet demonstrates how we achieve this connection using the `useEffect` hook:

```typescript
useEffect(() => {
  async function connectContract() {
    if (isConnected && wallet) {
      const TokenContract = TokenTrackAbi__factory.connect(CONTRACT_ID, wallet);
      setContract(TokenContract);
    }
  }

  connectContract();
}, [isConnected, wallet]);
```

To call the `mint_to_address` function in our contract, we have the handleMintToAddress function, which serves as the
bridge between our user interface and the `mint_to_address` function defined in our Fuel smart contract.

## Functionality Breakdown

1. **Contract Check**

```typescript
if (!contract) {
  return alert("Contract not loaded");
}
```

- Before starting, the function checks if there is a successful connection to our Fuel smart contract. If the contract object does not exist, it shows an alert message telling the user that they cannot mint tokens yet.

2. **Extracting Mint Address**

```typescript
const address = Address.fromString(mintTo);
const addressInput = { value: address.toB256() };
```

- This step retrieves the mint address from our application's state, stored in a variable named `mintTo`.
- It uses the Address class from fuels to convert the user-provided address string into an Address object.
- The converted address is then placed in an object (`addressInput`) to match the `AddressInput` type.

3. **Calling the Contract Function:**

```typescript
await contract.functions
  .mint_to_address(addressInput, Number(mintAmount))
  .txParams({
    gasPrice: 1,
    gasLimit: 1_000_000,
  })
  .call();
```

- The main action occurs here. It uses the contract object to call the `mint_to_address` function from our smart contract.
- It sets up arguments for the function call:
  - `addressInput`: The address object containing the converted mint address.
  - `Number(mintAmount)`: The mint amount provided by the user, converted to a number.
- It also sets transaction parameters:
  - `gasPrice`: A fixed gas price.
  - `gasLimit`: The maximum gas allowed for this transaction.
- The `.call()` method executes the transaction, sending the mint request to the Fuel network.

4. **Error Handling:**

```typescript
catch (error) {
    console.error(error);
}
```

- The function includes a `try...catch` block to manage any possible errors during the transaction, such as insufficient funds or invalid inputs. It logs errors to the console for debugging.

5. **Resetting Input Values:**

```typescript
setMintTo("");
setMintAmount("");
```

- After the execution (or if an error is caught), the function resets the user input fields for `mintTo` and `mintAmount` to blank, allowing the user to make a new mint request.

Overall, the `handleMintToAddress` function serves as the link between the user interface and our smart contract, similar to how other functions operate.

## User Interface for Token Interactions

This section discusses the user interface (UI) components that enable users to interact with token functionalities. The UI includes a form with fields and buttons for minting, transferring, and burning tokens.

### Form Breakdown:

The form has three sections: Minting, Transferring, and Burning. Each section is designed for specific token interactions:

#### Minting Token:

- Users can mint new tokens to a specified address.
- The `mintTo` input field is where users enter the recipient's address.
- The `mintAmount` field allows users to specify the number of tokens they wish to mint.
- The `mintType` dropdown field allows users to specify if they want to mint to an Address or a Contract.
- The "Mint New Tokens" button activates the `handleMintToAddress` function to start the minting process.

```typescript jsx
<div className="form-field">
  <input
    type="text"
    value={mintTo}
    onChange={(e) => setMintTo(e.target.value)}
    placeholder={`${mintType} to mint`}
  />
  <input
    type="number"
    value={mintAmount}
    onChange={(e) => setMintAmount(e.target.value)}
    placeholder="Tokens to mint"
  />
  <select
    value={mintType}
    onChange={(e) => setMintType(e.target.value)}
    style={{ marginRight: "10px" }}
  >
    <option value="Address">Address</option>
    <option value="Contract">Contract</option>
  </select>
  <button
    onClick={
      mintType === "Address" ? handleMintToAddress : handleMintToContract
    }
  >
    Mint New Tokens
  </button>
</div>
```

Here is how the UI would look like

![Mint UI](https://raw.githubusercontent.com/RobinNagpal/fuel-token-track-docs/main/assets/images/mint_ui.png)

#### Transferring Token:

- Users can transfer tokens from their wallet to another address.
- The `transferTo` field is for entering the recipient's address.
- The `transferAmount` field specifies the number of tokens to transfer.
- The `transferType` dropdown field allows users to specify if they want to transfer to an Address or a Contract.
- The "Transfer Tokens" button initiates the transfer via the `handleTransferToAddress` function.

```typescript jsx
<div className="form-field">
  <input
    type="text"
    value={transferTo}
    onChange={(e) => setTransferTo(e.target.value)}
    placeholder={`${transferType} to transfer to`}
  />
  <input
    type="number"
    value={transferAmount}
    onChange={(e) => setTransferAmount(e.target.value)}
    placeholder="Tokens to transfer"
  />
  <select
    value={transferType}
    onChange={(e) => setTransferType(e.target.value)}
    style={{ marginRight: "10px" }}
  >
    <option value="Address">Address</option>
    <option value="Contract">Contract</option>
  </select>
  <button
    onClick={
      transferType === "Address"
        ? handleTransferToAddress
        : handleTransferToContract
    }
  >
    Transfer Tokens
  </button>
</div>
```

Here is how the Transfer UI would look like

![Transfer UI](https://raw.githubusercontent.com/RobinNagpal/fuel-token-track-docs/main/assets/images/transfer_ui.png)

#### Burning Token:

- Users can burn (destroy) tokens from their wallet.
- The `burnAddress` field might automatically show a "burn" address where tokens are sent to be removed from circulation.
- The `burnAmount` field lets users decide how many tokens to burn.
- The "Burn Tokens" button starts the burning process with the `handleBurnFromAddress` function.

```typescript jsx
<div className="form-field">
  <input
    type="number"
    value={burnAmount}
    onChange={(e) => setBurnAmount(e.target.value)}
    placeholder="Tokens to burn"
  />
  <button onClick={handleBurn}>Burn Tokens</button>
</div>
```

Here is how burn UI would look like

![Burn UI](https://raw.githubusercontent.com/RobinNagpal/fuel-token-track-docs/main/assets/images/burn_ui.png)

### Installing the Fuel Browser Wallet

To interact with the frontend application, a browser wallet is required. Install the Fuel Wallet [here](https://chromewebstore.google.com/detail/fuel-wallet/dldjpboieedgcmpkchcjcbijingjcgok).

### Running the Project

To start the project, navigate to the `/frontend` directory and run:

```
npm start
```

This setup will launch the application and allow you to interact with the smart contract functionalities.
