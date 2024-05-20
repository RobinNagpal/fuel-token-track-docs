# Create Smart Contract Project

This section will guide us through creating a smart contract for the Token Track project, where we can mint, burn, and
track a user's token balance.

Assuming you followed the steps mentioned in the previous sections, and have `contracts` and `frontend` directories in your
project, let's start by navigating to the contracts directory.

```shell
cd contracts
```

### Initialize a New Smart Contract

We will be using `forc` to create a new smart contract.

Open the terminal and run the following command:

```shell
forc new TokenTrack
```

### Generated Project

After running the `forc new` command, the following files and directories will be created in the
`TokenTrack` directory:

```
token-track
├── Forc.toml
└── src
    └── main.sw
```

### `Forc.toml`

`Forc.toml`is a mandatory manifest file for each package. It includes fields such as
project details, dependencies, network information, build profiles, and other configurations essential for project
setup.

### `src/main.sw`

This is the file where all of the contract code will go.

### Sway

The file `src/main.sw` uses the `.sw` suffix, indicating it's written in Sway, a programming language designed for
the FuelVM. Sway is statically typed and compiled, with type inference and traits. It aims to enhance the safety and
performance of smart contract development through strong static analysis and compiler feedback.

#### Contract Structure

Sway contracts are structured in a specific way. They begin with a contract declaration, then imports and any struct to be used within the whole contract file, followed by an ABI (Application
Binary Interface) definition, and then the contract and it's functions.

```rust
contract;   // This is the type of sway program

use std::{hash::Hash,}; // This imports from ...
...  // Add any other imports here

// This is a struct accessible in the whole file
storage {

}
...   // Add any other structs

// This is the ABI to be implemented by the Contract
abi ContractAbi{

}

// This is the Contract implementing the ABI
impl ContractAbi for Contract {

}
```

Let's examine the contents of `main.sw`.

The first line in the file is:

```rust
contract;
```

In Sway, a program must declare its type. It can be a contract, a predicate, a script, or a library. Contracts,
predicates, and scripts are deployable to the blockchain, while a library is for code reuse and isn't deployed directly.

A Sway file begins by stating its type. A forc project can include many libraries but only one contract, script, or predicate.
Contracts are typically used for protocols or systems that adhere to a set of rules.

The next line in the file after contract declaration is imports:

```rust
use std::{hash::Hash,};
```

The next element in `main.sw` is an ABI.

### ABI

An ABI (Application Binary Interface) outlines an interface for the contract without including the actual function implementation. A contract must define
or import an ABI declaration and implement it. Best practice is to define the ABI in a separate library and import
it into the contract. This allows others to import and use the ABI in their scripts to interact with the contract. It follows this structure:

```rust
abi <ContractABIName> {
    // Function declarations
}
```

After importing or declaring the ABI in the `main.sw` file, the contract's functions, as defined in the ABI, are implemented.

# Token Track Contract

This section guides us on how to write a contract for Token Track Project using the Fuel network with Sway. It follows this structure:

```rust
impl <ContractABIName> for <ContractName> {
  // Function Implementations
}
```

For our example, we will
create a fungible token that includes several key functionalities necessary for managing a digital currency:

- **Minting**: Generate and add new tokens to the circulation.
- **Burning**: Remove tokens from circulation permanently.
- **Transferring**: Send tokens between different addresses and contracts on the Fuel network.
- **Balance Tracking**: Record how many tokens each address or contract holds.

By following this guide and using the provided Sway code, we'll gain practical experience in creating our own token contract, ready for deployment on the Fuel network!

## Storage

Since we are developing a token, we need storage to track the total circulation and maintain a map of account balances that hold the token.

Define a storage block with the following lines of code:

```rust
// Define storage variables for the contract
storage {
    // `balances` is a StorageMap mapping identities to their u64 balances
    balances: StorageMap<Identity, u64> = StorageMap::<Identity, u64> {},
    // `total_supply` stores the total number of tokens in circulation
    total_supply: u64 = 0,
}
```

In smart contract development, persistent storage is crucial. It stores values within the contract itself, unlike
temporary memory values which disappear after the contract finishes executing. Here, we define two storage variables:

1. `balances`: A Storage Map that links identities to their balances.
2. `total_supply`: An integer that records the total number of tokens in circulation.

Let's clarify some terms:

- **Identity**: An enum type that handles both `Address` and `ContractId` types, useful for identifying senders without
  specifying if they are addresses or contracts. Here is how it's implemented:

```rust
pub enum Identity {
    Address: Address,
    ContractId: ContractId,
}
```

- **Storage Maps**: These are generic storage maps (StorageMap<K, V>) used for storing key-value pairs. They allow
  functions like `insert()` and `get()` to manage values based on keys.

- **u64**: A primitive data type in Sway, representing a 64-bit unsigned integer.

## ABI

Next, we will define the ABI for our project. We will discuss the details of
each function when we implement them.

```rust
// Define the contract's ABI
abi TokenTrackAbi {
    // Function to mint new tokens for a specific address recipient with an amount
    #[storage(read, write)]
    fn mint_to_address(recipient: Address, amount: u64);

    // Function to mint new tokens for a specific contract recipient with an amount
    #[storage(read, write)]
    fn mint_to_contract(recipient: ContractId, amount: u64);

    // Function to burn tokens
    #[storage(read, write)]
    fn burn_token(amount: u64);

    // Function to transfer tokens between two addresses
    #[storage(read, write)]
    fn transfer_coins_to_address(coins: u64, from: Address, target: Address);

    // Function to transfer tokens to a contract (not implemented yet)
    #[storage(read, write)]
    fn transfer_coins_to_contract(coins: u64, from: ContractId, target: ContractId);

    // Function to get the balance of an identity
    #[storage(read)]
    fn get_balance(addr: Identity) -> u64;
}
```

In Sway, functions are 'pure' by default, meaning they do not access any persistent storage. However, they can be modified to access storage by using the `storage` function attribute. This attribute can take `read` or `write` arguments to specify the required type of access:

- The `#[storage(read)]` attribute indicates that a function needs read access to the storage.
- The `#[storage(write)]` attribute indicates that a function needs write access to the storage.

## Contract Code - Implementation of ABI

Now that we understand the ABI and storage, let's examine the `mint_to_address` function in our Sway contract.

### Minting Functionality

This function is designed to create new tokens and add them directly to a specific address on the Fuel network. Here's
the basic code for the `mint_to_address` function:

```rust
impl TokenTrackAbi for TokenTrack {

  #[storage(read, write)]
  fn mint_to_address(recipient: Address, amount: u64) {
      // Read the current total supply from storage
      storage
          .total_supply
          .write(storage.total_supply.read() + amount);

      let b256_addr = recipient.into();
      // Access the sender's balance or initialize it to 0 if it doesn't exist
      let identity: Identity = Identity::Address(Address::from(b256_addr));
      let mut balance = storage.balances.get(identity).try_read().unwrap_or(0);
      balance += amount;
      storage.balances.insert(identity, balance);
  }
}
```

The `mint_to_address` function takes two parameters:

- **recipient (Address)**: The address that will receive the newly minted tokens.
- **amount (u64)**: The number of tokens to mint.

Steps involved in the `mint_to_address` function:

1. **Update Total Supply**: It reads the current total supply and adds the new tokens.
2. **Mint and Update Balance**: It then updates the balance of the recipient, initializing it to 0 if not previously set, and adds the new tokens.

This method for creating tokens is mirrored by the `mint_to_contract` function, which credits new tokens to a specific
contract on the Fuel network.

### Burning Tokens from Addresses

The `burn_from_address` function is crucial for managing the lifecycle of our tokens by removing tokens from circulation from a specified address.

#### Understanding Burning

Burning tokens reduces their total supply, making them unusable. This is often used to control inflation or to burn transaction fees.

Here's the basic code for the `burn_from_address` function:

```rust
// Implementation of the `burn_token` function
#[storage(read, write)]
fn burn_token(amount: u64) {
  // Read the current total supply from storage
  storage
      .total_supply
      .write(storage.total_supply.read() - amount);
}
```

The function takes a single parameter:

- **amount (u64)**: The number of tokens to burn.

There is only one extra step involved in the `burn_token` function, i.e., to decrease the total supply by the amount to be burned.

### Transferring Tokens to Addresses

Token transfer is an essential operation that allow users to move tokens between wallets or accounts on the Fuel network. This function makes these transfers possible, acting as a bridge between user addresses.

Here's a snippet of the `transfer_coins_to_address` function to illustrate its core components:

```rust
#[storage(read, write)]
fn transfer_coins_to_address(coins: u64, from: Address, target: Address) {
  let b256_addr_from = from.into();
  // Access the sender's balance or initialize it to 0 if it doesn't exist
  let identity_from: Identity = Identity::Address(Address::from(b256_addr_from));
  let mut from_balance = storage.balances.get(identity_from).try_read().unwrap_or(0);

  // Ensure sufficient balance before transferring
  require(from_balance >= coins, "Not enough tokens");

  // Update the sender's balance after transferring
  from_balance -= coins;

  let b256_addr_to = target.into();
  // Access the recipient's balance or initialize it to 0 if it doesn't exist
  let identity_to: Identity = Identity::Address(Address::from(b256_addr_to));
  let mut target_balance = storage.balances.get(identity_to).try_read().unwrap_or(0);

  // Update the recipient's balance with the transferred amount
  target_balance += coins;

  // Store the updated balances back in the storage map
  storage.balances.insert(identity_from, from_balance);
  storage.balances.insert(identity_to, target_balance);
}
```

The function takes three arguments:

- **coins (u64):** The number of tokens to be transferred.
- **from (Address):** The sender's address initiating the token transfer.
- **target (Address):** The recipient address that will receive the tokens.

Steps involved in the `transfer_coins_to_address` function:

1. **Balance Check:** It checks whether the sender has enough tokens to cover the transfer.
2. **Insufficient Balance Handling:** If there are not enough tokens, the transaction is stopped to prevent an invalid transfer.
3. **Balance Update:** If the balance is sufficient, it deducts the specified amount from the sender's balance and adds it to the recipient's balance.
4. **Storage Update:** Finally, it updates the storage to reflect the new balances of both the sender and the recipient.

The `transfer_coins_to_contract` function builds on this by allowing transfer of the tokens to another contract on the Fuel network.

### Checking Token Balances

This function shows the token balance of the specified address or contract.

Here's a snippet of the `get_balance` function to illustrate its core components:

```rust
#[storage(read)]
fn get_balance(addr: Identity) -> u64 {
  return storage.balances.get(addr).try_read().unwrap_or(0);
}
```

The function takes one argument and returns an integer:

- **addr (Identity):** This parameter specifies the address or contract ID whose balance we want to retrieve.
- **u64:** Returned token balance of the specified identity (Address/ContractID).

The `get_balance` function is read-only and does not modify the contract's storage. Here's how it works:

1. **Storage Access:** It accesses the balances storage map, which keeps track of each token holder's balance.
2. **Balance Lookup:** It looks up the balance using the specified identity.
3. **Balance Return:** If a balance entry exists, it returns the stored value as an integer.

## Building & Deploying the Contracts Locally

The "Build" phase is essential in preparing our smart contract for deployment on the Fuel network. Smart contracts are typically written in high-level languages like Sway, which are easy to read and maintain. However, the Fuel network operates on bytecode, a low-level machine-readable format. Build process translates our Sway code into bytecode that the Fuel network can execute.

For building and deploying all the contracts under the `contracts` folder, we are going to use the `dev` command from fuel which will build and deploy the contracts on the local node.

But since we need some initial state of the chain, we must configure a `chainConfig.json` file. Here is an example of what that looks like using version 0.22.1 of fuel-core:

<details>
  <summary>chainConfig.json</summary>

```json
{
  "chain_name": "Testnet",
  "block_gas_limit": 1000000000,
  "initial_state": {
    "coins": [
      {
        "owner": "0x387c9580ba76a18c62d38edd400c0473429baa257b2570ee508c5061be25d035",
        "amount": "0x1000000000000000",
        "asset_id": "0x0000000000000000000000000000000000000000000000000000000000000000"
      },
      {
        "owner": "0x6b63804cfbf9856e68e5b6e7aef238dc8311ec55bec04df774003a2c96e0418e",
        "amount": "0x1000000000000000",
        "asset_id": "0x0000000000000000000000000000000000000000000000000000000000000000"
      },
      {
        "owner": "0x54944e5b8189827e470e5a8bacfc6c3667397dc4e1eef7ef3519d16d6d6c6610",
        "amount": "0x1000000000000000",
        "asset_id": "0x0000000000000000000000000000000000000000000000000000000000000000"
      },
      {
        "owner": "0xe10f526b192593793b7a1559a391445faba82a1d669e3eb2dcd17f9c121b24b1",
        "amount": "0x1000000000000000",
        "asset_id": "0x0000000000000000000000000000000000000000000000000000000000000000"
      },
      {
        "owner": "0x577e424ee53a16e6a85291feabc8443862495f74ac39a706d2dd0b9fc16955eb",
        "amount": "0x1000000000000000",
        "asset_id": "0x0000000000000000000000000000000000000000000000000000000000000000"
      },
      {
        "owner": "0xc36be0e14d3eaf5d8d233e0f4a40b3b4e48427d25f84c460d2b03b242a38479e",
        "amount": "0x1000000000000000",
        "asset_id": "0x0000000000000000000000000000000000000000000000000000000000000000"
      },
      {
        "owner": "0xa1184d77d0d08a064e03b2bd9f50863e88faddea4693a05ca1ee9b1732ea99b7",
        "amount": "0x1000000000000000",
        "asset_id": "0x0000000000000000000000000000000000000000000000000000000000000000"
      },
      {
        "owner": "0xb5566df884bee4e458151c2fe4082c8af38095cc442c61e0dc83a371d70d88fd",
        "amount": "0x1000000000000000",
        "asset_id": "0x0000000000000000000000000000000000000000000000000000000000000000"
      },
      {
        "owner": "0x9da7247e1d63d30d69f136f0f8654ee8340362c785b50f0a60513c7edbf5bb7c",
        "amount": "0x1000000000000000",
        "asset_id": "0x0000000000000000000000000000000000000000000000000000000000000000"
      },
      {
        "owner": "0x4b2ca966aad1a9d66994731db5138933cf61679107c3cde2a10d9594e47c084e",
        "amount": "0x1000000000000000",
        "asset_id": "0x0000000000000000000000000000000000000000000000000000000000000000"
      },
      {
        "owner": "0x26183fbe7375045250865947695dfc12500dcc43efb9102b4e8c4d3c20009dcb",
        "amount": "0x1000000000000000",
        "asset_id": "0x0000000000000000000000000000000000000000000000000000000000000000"
      },
      {
        "owner": "0x81f3a10b61828580d06cc4c7b0ed8f59b9fb618be856c55d33decd95489a1e23",
        "amount": "0x1000000000000000",
        "asset_id": "0x0000000000000000000000000000000000000000000000000000000000000000"
      },
      {
        "owner": "0x587aa0482482efea0234752d1ad9a9c438d1f34d2859b8bef2d56a432cb68e33",
        "amount": "0x1000000000000000",
        "asset_id": "0x0000000000000000000000000000000000000000000000000000000000000000"
      },
      {
        "owner": "0x53a9c6a74bee79c5e04115a007984f4bddaafed75f512f68766c6ed59d0aedec",
        "amount": "0x0004000000000000",
        "asset_id": "0x0000000000000000000000000000000000000000000000000000000000000000"
      },
      {
        "owner": "0x459b0d89874f2eff10486fd971d717cc887a775426d12b106ad4a84f0df7de00",
        "amount": "0xFFF4000000000000",
        "asset_id": "0x0000000000000000000000000000000000000000000000000000000000000000"
      },
      {
        "owner": "0x39587e805197c116d9cfbe4505fe6934029d56281aa32d8bc78cd6b93abc28d1",
        "amount": "0xFFF4000000000000",
        "asset_id": "0x0000000000000000000000000000000000000000000000000000000000000000"
      },
      {
        "owner": "0x5f57c3834ad939522d0d927e69f217c3edb0a716df60cdb246cb14efb38723d3",
        "amount": "0xFFF4000000000000",
        "asset_id": "0x0000000000000000000000000000000000000000000000000000000000000000"
      },
      {
        "owner": "0x94ffcc53b892684acefaebc8a3d4a595e528a8cf664eeb3ef36f1020b0809d0d",
        "amount": "0xFFF4000000000000",
        "asset_id": "0x0000000000000000000000000000000000000000000000000000000000000000"
      }
    ]
  },
  "consensus_parameters": {
    "base_asset_id": "0000000000000000000000000000000000000000000000000000000000000000"
  },
  "consensus": {
    "PoA": {
      "signing_key": "f65d6448a273b531ee942c133bb91a6f904c7d7f3104cdaf6b9f7f50d3518871"
    }
  }
}
```

</details>

To get coins in your wallet, add an entry to the coins array with your wallet address in B256 type address (begins with 0x):

```json
{
  "owner": "<YOUR_WALLET_ADDRESS>",
  "amount": "0x1000000000000000",
  "asset_id": "0x0000000000000000000000000000000000000000000000000000000000000000"
}
```

The coins array specifies the initial state of the local Fuel node which we are deploying. Each of its value contains three properties:

- owner: The address of the wallet.
- amount: The number of tokens, in hexadecimal format.
- asset_id: The identifier for the asset type, typically zero for the base asset (e.g., Fuel)

#### Check wallet address

You can check your wallet address by running the following command:

```shell
forc wallet accounts
```

Finally build and deploy the contracts using the `dev` command in the root directory:

```shell
npx fuels@0.82.0 dev
```

### Output

A successful build will create a new directory named `out` within the contract folder. This directory contains several important files:

- **debug:** This folder contains debugging information which will be useful during deployment.

  - **TokenTrack-contract-abi.json:** This file stores the contract's public Application Binary Interface (ABI) in JSON format, which describes how other applications and contracts can interact with it.
  - **TokenTrack-contract-storage_slots.json:** This file contains the storage layout of our contract, showing how data is stored on the blockchain.
  - **TokenTrack-contract.bin:** This file is the bytecode version of our Sway code, containing instructions that the Fuel network will execute upon deployment.

**Congratulations on building your first Fuel contract!**

You've successfully completed the writing, building, and deploying phases of your smart contract on the Fuel network.

Ready to explore further?

For the complete code of this contract, visit the project repository [here](https://github.com/RobinNagpal/fuels-token-example-code).
