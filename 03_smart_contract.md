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

`Forc.toml`is a mandatory manifest file for each package, written in TOML format. It includes fields such as
project details, dependencies, network information, build profiles, and other configurations essential for project
setup.

### `src/main.sw`

This the file where all of the contract code will go.

### Sway

The file `src/main.sw` uses the `.sw` suffix, indicating it's written in Sway, a programming language designed for
the FuelVM. Sway is statically typed and compiled, with type inference and traits. It aims to enhance the safety and
performance of smart contract development through strong static analysis and compiler feedback.

Let's examine the contents of `main.sw`.

The first line in the file is:

```
contract
```

In Sway, a program must declare its type, it can be a contract, a predicate, a script, or a library. Contracts,
predicates, and scripts are deployable to the blockchain, while a library is for code reuse and isn't deployed directly.

A Sway file begins by stating its type. A forc project can include many libraries but only one contract, script, or predicate.
Contracts are typically used for protocols or systems that adhere to a set of rules.

The next element in `main.sw` is an ABI.

### ABI

An ABI (Application Binary Interface) outlines an interface without including function bodies. A contract must define
or import an ABI declaration and implement it. It is best practice to define the ABI in a separate library and import
it into the contract. This allows others to import and use the ABI in their scripts to interact with the contract.

Following the ABI in the `main.sw` file is the implementation of the ABI, where the functions defined in the ABI
are actually written.

# Token Track Contract

This section guides us on how to write a contract for Token Track using the Fuel network with Sway. Here, we will
create a fungible token that includes several key functionalities necessary for managing a digital currency:

- **Minting**: Generate and add new tokens to the circulation.
- **Burning**: Remove tokens from circulation permanently.
- **Transferring**: Send tokens between different addresses and contracts on the Fuel network.
- **Balance Tracking**: Record how many tokens each address or contract holds.

By following this guide and using the provided Sway code, we'll gain practical experience in creating our own token contract, ready for deployment on the Fuel network!

Begin by clearing the file, keeping only the `contract` keyword, as we are drafting a contract.

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

Next, we will define the ABI (Application Binary Interface) for our project. We will discuss the details of
each function when we implement them.

```rust
// Define the contract's ABI (Application Binary Interface)
abi MyContract {
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

In Sway, functions are by default pure, meaning they do not access any persistent storage. However, they can be modified to access storage by using the `storage` function attribute. This attribute can take `read` and/or `write` arguments to specify the required type of access:

- The `#[storage(read)]` attribute indicates that a function needs read access to the storage.
- The `#[storage(write)]` attribute indicates that a function needs write access to the storage.

## Contract Code - Implementation of ABI

Now that we understand the ABI and storage, let's examine the `mint_to_address` function in our Sway contract.

### Minting Functionality

This function is designed to create new tokens and add them directly to a specific address on the Fuel network. Here's
the essential code for the `mint_to_address` function:

```rust
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
```

The function takes two parameters:

- **recipient (Address)**: The address that will receive the newly minted tokens.
- **amount (u64)**: The number of tokens to mint.

Steps involved in the `mint_to_address` function:

1. **Update Total Supply**: It reads the current total supply and adds the new tokens.
2. **Mint and Update Balance**: It then updates the balance of the recipient, initializing it to 0 if not previously set, and adds the new tokens.

This method for creating tokens is mirrored by the `mint_to_contract` function, which credits new tokens to a
contract on the Fuel network.

### Burning Tokens from Addresses

The `burn_from_address` function is crucial for managing the lifecycle of our tokens by removing tokens from circulation from a specified address.

#### Understanding Burning

Burning tokens reduces their total supply, making them unusable. This is often used to control inflation or to burn transaction fees.

Here's the core code for the `burn_from_address` function:

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

- **amount (u64)**: The quantity of tokens to burn.

There is only one step involved in the `burn_token` function, i.e., to decrease the total supply by the amount to be burned.

### Transferring Tokens to Addresses

Token transfers are essential operations that allow users to move tokens between wallets or accounts on the Fuel network. This function makes these transfers possible, acting as a bridge between user addresses.

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

- **coins (u64):** The quantity of tokens to be transferred.
- **from (Address):** The sender's address initiating the token transfer.
- **target (Address):** The recipient address that will receive the tokens.

Steps involved in the `transfer_coins_to_address` function:

1. **Balance Checks:** It checks that the sender has enough tokens to cover the transfer.
2. **Insufficient Balance Handling:** If there are not enough tokens, the transaction is stopped to prevent an invalid transfer.
3. **Balance Updates:** If the balance is sufficient, it deducts the specified amount from the sender's balance and adds it to the recipient's balance.
4. **Storage Updates:** Finally, it updates the storage to reflect the new balances of both the sender and the recipient.

The `transfer_coins_to_contract` function builds on this by allowing token transfers to another contract on the Fuel network.

### Checking Token Balances

This function provides a view into how many tokens each address or contract holds.

Here's a snippet of the `get_balance` function to illustrate its core components:

```rust
#[storage(read)]
fn get_balance(addr: Identity) -> u64 {
  return storage.balances.get(addr).try_read().unwrap_or(0);
}
```

The function takes one argument and returns an integer:

- **addr (Identity):** This parameter specifies the address or contract ID whose balance we want to retrieve.
- **u64:** It returns the token balance associated with the specified identity.

The `get_balance` function is read-only and does not modify the contract's storage. Here's how it works:

1. **Storage Access:** It accesses the balances storage map, which keeps track of each token holder's balance.
2. **Balance Lookup:** It looks up the balance using the specified identity.
3. **Balance Return:** If a balance entry exists, it returns the stored value as an integer.

## Building the Contract

The "Build" phase is essential in preparing our smart contract for deployment on the Fuel network. Smart contracts are typically written in high-level languages like Sway, which are easy to read and maintain. However, the Fuel network operates with bytecode, a low-level machine-readable format. The building process translates our Sway code into bytecode that the Fuel network can execute.

Navigate to the contract folder:

```shell
cd token-track
```

Then run the command to build the contract:

```shell
forc build
```

### Output

A successful build will create a new directory named `out` within the contract folder. This directory contains several important files:

- **debug subdirectory:** This folder contains debugging information useful during development.

  - **TokenTrack-contract-abi.json:** This file stores the contract's public Application Binary Interface (ABI) in JSON format, which describes how other applications and contracts can interact with it.
  - **TokenTrack-contract-storage_slots.json:** This file details the storage layout of our contract, showing how data is stored on the blockchain.
  - **TokenTrack-contract.bin:** The most important output, this file is the bytecode version of our Sway code, containing instructions that the Fuel network will execute upon deployment.

## Deploying the Contract (Testnet)

With the contract built, the next step is to deploy it to the testnet. Here are the steps involved, including wallet setup and deployment commands:

1. **Wallet Setup:**

   - You need a Fuel wallet to pay for transaction fees.
   - If you don’t have a wallet, the `forc deploy` command will guide you through creating one, which includes generating a secure seed phrase to keep confidential.
   - Set a strong, unique password for your wallet when prompted.

2. **Securing Funds (for Testnet Deployment):**

   - We are deploying to a testnet, which uses fake currency (testnet FUEL tokens).
   - Obtain testnet coins to cover deployment fees from the [beta-5 faucet](https://faucet-beta-5.fuel.network/).

3. **The Deployment Command:**

   - With your wallet set up and funded, deploy the contract using:
     ```shell
     forc deploy --testnet
     ```

4. **Wallet Interaction and Confirmation:**

   - Enter your wallet password when prompted to unlock it and authorize the deployment.
   - Choose the account for signing the deployment transaction by entering its index number.
   - Confirm the deployment by typing Y (yes) when prompted.

5. **Deployment Success and Information:**
   - Upon successful deployment, you’ll receive:
     - Network Endpoint: The URL of the Fuel network where the contract is deployed.
     - Contract ID: A unique identifier for your contract on the blockchain.
     - Deployed in Block: The blockchain block number that includes your transaction.

## Deploying the Contract (Local Node)

We have now seen how to deploy to the testnet, let us also see how we can deploy the contract to a local node.

There are two types of Fuel networks that can be run:

1. In-memory network (without persistence)
2. Local network with persistence

For this example, we will be using an In-memory network.

An in-memory node does not persist the blockchain state anywhere, it is only stored in memory as long as the node is active and running.

To spin-up a local in-memory Fuel node, run the following command:

```bash
fuel-core run --db-type in-memory --debug
```

Once the node is up and running, make sure we are in the `contracts/TokenTrack`, we can deploy our contract using the following command:

```bash
forc deploy --unsigned --node-url 127.0.0.1:4000/graphql
```

This will deploy the contract to the local node.

**Congratulations on building your first Fuel contract!**

You've successfully completed the writing, building, and deploying phases of your smart contract on the Fuel network.

Ready to explore further?

For the complete code of this contract, visit the project repository [here](https://github.com/RobinNagpal/fuels-token-example-code).
