# Initializing the React App

Let's begin by setting up the React application. First, navigate back to the main directory from `contracts/TokenTrack`:

```shell
cd ../..
```

Now, create a new React project:

```shell
npx create-react-app frontend --template typescript
cd frontend
```

Next, install the fuels SDK:

```shell
npm install fuels@0.84.0 @fuels/react@0.18.0 @fuels/connectors@0.2.2 @tanstack/react-query@5.28.9
```

# Generating Contract Types
We have already added the configuration for fuels in the `fuels.config.ts` file. To generate the contract types, run the following command:

```shell
npx fuels@0.84.0 build
```

This command generates the contract types in the path specified in the `fuels.config.ts` file, which for us will be `frontend/src/sway-api`.

The generated files include:
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

- Function Names: These serve as identifiers that correspond to the actual names in our contract, such as `burn_from_address`, `mint_to_address`.
- Function Arguments: Each function lists its required arguments with specific types like `AddressInput` or `BigNumberish` to ensure data compatibility.
- Return Types: This indicates the type of data a function may return, such as void for no return value or BN for returning a large number.

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

Now that we have a basic React application structure, let's integrate the necessary components to enable interaction with our Fuel smart contract. This section focuses on setting up the environment within our index.tsx file.

1. Importing Required Modules:

At the beginning of the frontend/src/index.tsx file, we'll need to import the following modules:

```typescript
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

```typescript
const queryClient = new QueryClient();
```

This query client will assist with managing data fetching related to our smart contract interactions in our React application.

3. Wrapping the App with Providers:
   Next, modify the structure of the index.tsx file to wrap the main application component (App) with the FuelProvider and QueryClientProvider components. Here's the updated structure:

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

- `QueryClientProvider:` This component wraps the entire application and provides the queryClient context for data fetching.

- `FuelProvider:` This component wraps our App component and establishes the Fuel network connection context. Inside the FuelProvider, we define a configuration object (fuelConfig) that specifies how our application will connect to the network and interact with wallets
  - The connectors property is an array containing instances of different wallet connector classes. These connectors allow users to connect their Fuel wallets to the application.

Next up, we'll define the code and functions to interact with the contract.

Let's establish a connection between the React application and our deployed smart contract. This connection will allow the application to call our contract's functions and retrieve data from the blockchain. The following code snippet demonstrates how we achieve this connection using the `useEffect` hook:

```typescript
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

```typescript
if (!contract) {
    return alert("Contract not loaded");
}
```

- Before proceeding, the function ensures a successful connection to our Fuel smart contract. It checks if the contract object exists. If not, it displays an alert message informing the user that they cannot mint tokens yet.

2. Extracting Mint Address

```typescript
const address = Address.fromString(mintTo);
const addressInput = { value: address.toB256() };
```

- This section retrieves the mint address value from our application's state, stored in a variable named mintTo.
- It utilizes the Address class from fuels to convert the user-provided address string into the Address object.
- The converted address is then wrapped in an object (addressInput) to form `AddressInput` type.

3. Calling the Contract Function:

```typescript
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

```typescript
catch (error) {
    console.error(error);
}
```

- The function includes a `try...catch` block to handle any potential errors that might occur during the transaction, such as insufficient funds or invalid inputs. Errors are logged to the console for debugging purposes.

5. Resetting Input Values:

```typescript
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
