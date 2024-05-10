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
forc new token-track
```

### Generated Project

After running the `forc new` command, the following files and directories will be created in the
`token-track` directory:

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

In Sway, a program must declare its type—it can be a contract, a predicate, a script, or a library. Contracts, 
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

This section guides you on how to write a contract for Token Track using the Fuel network with Sway. Here, we will 
create a fungible token that includes several key functionalities necessary for managing a digital currency:

- **Minting**: Generate and add new tokens to the circulation.
- **Burning**: Remove tokens from circulation permanently.
- **Transferring**: Send tokens between different addresses and contracts on the Fuel network.
- **Balance Tracking**: Record how many tokens each address or contract holds.

By following this guide and using the provided Sway code, you'll gain practical experience in creating your own token contract, ready for deployment on the Fuel network!

Begin by clearing your file, keeping only the `contract` keyword, as we are drafting a contract.

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

    // Function to burn tokens from a specific Address's balance
    #[storage(read, write)]
    fn burn_from_address(target: Address, amount: u64);

    // Function to burn tokens from a specific contract's balance
    #[storage(read, write)]
    fn burn_from_contract(target: ContractId, amount: u64);

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

Now that we've established the concept of ABI and storage, let's delve into the `mint_to_address` function within the 
Sway contract.

#### Minting functionality

This function serves a specific purpose: it allows us to create brand new tokens and credit them directly to a designated address on the Fuel network.
Here's a snippet of the mint_to_address function to illustrate its core components:

```rust
#[storage(read, write)]
fn mint_to_address(recipient: Address, amount: u64) {
    // Read the current total supply from storage
    storage
        .total_supply
        .write(amount + storage.total_supply.read());

    let b256_addr = recipient.into();
    // Access the sender's balance (or initialize it to 0 if it doesn't exist)
    let identity: Identity = Identity::Address(Address::from(b256_addr)); // Casting Address to Identity
    let mut balance = storage.balances.get(identity).try_read().unwrap_or(0);
    balance += amount;
    storage.balances.insert(identity, balance);
}
```

As we can see, the function takes two arguments:

- **recipient (Address):** This specifies the address that will receive the newly minted tokens. Imagine this as the target wallet or account where the fresh tokens will be deposited.
- **amount (u64):** This defines the quantity of tokens to be minted. This value determines how many new tokens will be created and credited to the recipient.

Let's break down the steps involved when we call the mint_to_address function:

1. **Total Supply Update:** The contract first keeps track of the total number of tokens in circulation. The function begins by reading the current total supply stored in the contract.
2. **Minting and Balance Update:** Next, it adds the specified amount of new tokens to the existing total supply. Additionally, it retrieves the balance of the recipient address (initializing it to 0 if the address hasn't received any tokens yet).
3. **Balance Update:** The magic happens here! The function credits the recipient's address with the minted tokens. It essentially adds the amount of newly minted tokens to the recipient's existing balance.
4. **Storage Update:** Finally, the contract doesn't forget to update its records. The function stores the incremented total supply, reflecting the newly minted tokens, along with the recipient's updated balance, which now includes the minted amount.

Building upon the concept of minting with `mint_to_address`, the `mint_to_contract` function allows us to mint new tokens and credit them to another contract residing on the Fuel network.

### Burning Tokens from Addresses

The burn_from_address function serves a critical purpose in our token's lifecycle: it enables us to remove existing tokens from circulation by burning them from a specific address on the Fuel network.

#### Understanding Burning:

Imagine taking tokens out of circulation, effectively reducing their total supply. Burning achieves this by rendering the tokens unusable. This mechanism can be employed for various reasons, such as implementing deflationary models or burning a portion of transaction fees.

Here's a snippet of the burn_from_address function to illustrate its core components:

```rust
#[storage(read, write)]
fn burn_from_address(target: Address, amount: u64) {
    // Read the current total supply from storage
    storage
        .total_supply
        .write(storage.total_supply.read() - amount);

    let b256_addr = target.into();
    let identity: Identity = Identity::Address(Address::from(b256_addr)); // Casting Address to Identity
    // Access the target's balance (or initialize it to 0 if it doesn't exist)
    let mut target_balance = storage.balances.get(identity).try_read().unwrap_or(0);

    // Ensure sufficient balance before burning
    assert(target_balance >= amount);

    // Update the target's balance after burning
    target_balance -= amount;

    // Store the updated balance back in the storage map
    storage.balances.insert(identity, target_balance);
}
```

As we can see, the function takes two arguments:

- **target (Address):** This parameter identifies the address from which tokens will be burned. Think of it as the source account whose token balance will be reduced.
- **amount (u64):** This value specifies the quantity of tokens to be burned. It defines how many tokens will be removed from the total supply and the target address's balance.

Let's break down the steps involved when we call the `burn_from_address` function:

1. **Total Supply Update:** Similar to minting, burning also affects the total supply. The function starts by reading the current total supply stored in the contract and deducting the amount to be burnt.
2. **Balance Check and Update:** Before burning, the function performs a safety check. It ensures that the target address has enough balance to cover the specified amount for burning. If sufficient balance exists, the function proceeds to deduct the amount from the target's balance.
3. **Storage Update:** Finally, the contract diligently updates its records. It stores the updated total supply, reflecting the burned tokens, and the target address's balance, which now has the reduced amount.

Building upon the `burn_from_address` function, the `burn_From_contract` function allows us to burn tokens from a contract residing on the Fuel network.

### Transferring Tokens to Address

Token transfers are fundamental operations that enable users to move our tokens between their wallets or accounts on the Fuel network. This function facilitates these transfers, acting as a bridge between user addresses.

Here's a snippet of the `transfer_coins_to_address` function to illustrate its core components:

```rust
#[storage(read, write)]
fn transfer_coins_to_address(coins: u64, from: Address, target: Address) {
  let b256_addr_from = from.into();
  // Access the sender's balance (or initialize it to 0 if it doesn't exist)
  let identity_from: Identity = Identity::Address(Address::from(b256_addr_from)); // Casting Address to Identity
  let mut from_balance = storage.balances.get(identity_from).try_read().unwrap_or(0);

  // Ensure sufficient balance before transferring
  require(from_balance >= coins, "Not enough tokens");

  // Update the sender's balance after transferring
  from_balance -= coins;

  let b256_addr_to = target.into();
  // Access the recipient's balance (or initialize it to 0 if it doesn't exist)
  let identity_to: Identity = Identity::Address(Address::from(b256_addr_to)); // Casting Address to Identity
  let mut target_balance = storage.balances.get(identity_to).try_read().unwrap_or(0);

  // Update the recipient's balance with the transferred amount
  target_balance += coins;

  // Store the updated balances back in the storage map
  storage.balances.insert(identity_from, from_balance);
  storage.balances.insert(identity_to, target_balance);
}
```

As we can see, the function takes three arguments:

- **coins: u64:** This parameter defines the quantity of tokens to be transferred. It specifies how many tokens will be moved from the contract.
- **from: ContractId:** This identifies the contract acting as the sender, initiating the transfer of tokens to the address.
- **target: Address:** This specifies the recipient address that will receive the transferred tokens. Think of it as the destination wallet or account where the tokens will be deposited.

Let's break down the steps involved when we call the transfer_coins_to_address function:

1. **Balance Checks:** The function likely performs crucial safety checks. It ensures that the sender contract has enough balance to cover the requested coins amount for transfer.
2. **Insufficient Balance Handling:** If the balance check fails (meaning the sender contract doesn't have enough tokens), the function might raise an error or revert the transaction to prevent an invalid transfer.
3. **Balance Updates:** Assuming sufficient balance, the function debits the coins amount from the sender contract's balance. On the other hand, it credits the same coins amount to the recipient's address, effectively increasing their token holdings.
4. **Storage Updates:** Finally, the contract meticulously updates its internal storage to reflect the changes. It stores the updated balance of the sender contract, reflecting the transferred tokens, and potentially updates any relevant storage related to the recipient's address (if it's the first time they're receiving tokens).

Building upon the `transfer_coins_to_address` function, the `transfer_coins_to_contract` function allows us to transfer tokens another contract residing on the Fuel network.

### Checking Token Balances

A balance represents the number of tokens held by a particular address or contract. This function acts as a window into the token distribution, allowing us to query and retrieve this information.

Here's a snippet of the `get_balance` function to illustrate its core components:

```rust
#[storage(read)]
fn get_balance(addr: Identity) -> u64 {
  return storage.balances.get(addr).try_read().unwrap_or(0);
}
```

As we can see, the function takes one argument and returns an integer:

- **addr: Identity:** This parameter acts as the key. It specifies the address or contract ID for which we want to retrieve the balance.
  Function Return Value:

- **u64:** The function returns an unsigned 64-bit integer representing the token balance associated with the provided addr.

The get_balance function is likely a read-only function, meaning it doesn't modify the contract's storage. Here's a simplified breakdown of its functionality:

1. **Storage Access:** The function retrieves the balances storage map, which stores the current balance for each address or contract that has interacted with our token.
2. **Balance Lookup:** Using the provided addr (address or contract ID), the function searches the balances storage map for the corresponding balance entry.
3. **Balance Return:** If a balance entry exists for the provided addr, the function returns the stored value as a u64 integer.

## Building the contract

The "Build" phase is a crucial step in preparing our smart contract for deployment on the Fuel network. Smart contracts are written in high-level languages like Sway for readability and maintainability. The Fuel network, however, operates with bytecode, a low-level machine-readable format. Building bridges the gap by translating our Sway code into bytecode that the Fuel network can understand and execute.

Navigate to the contract folder:

```shell
cd token-track
```

Then run the following command to build the contract:

```shell
forc build
```

### Output

A successful build will generate a new directory named `out` within the contract folder. This directory contains several key files:

- **debug subdirectory:** This folder holds debugging information that can be helpful during development.

  - **counter-contract-abi.json:** This file stores the public Application Binary Interface (ABI) of the contract in JSON format. The ABI defines how other applications and contracts can interact with our functions and variables. Think of it as an instruction manual for external entities to communicate with our contract.
  - **counter-contract-storage_slots.json:** This file contains details about the storage layout of our contract. It defines how data is persisted within the contract on the blockchain.
  - **counter-contract.bin:** This is the most crucial output. It's the actual bytecode representation of the Sway code. This file contains the machine-readable instructions that the Fuel network can execute when we deploy our contract.

## Deploying the contract

Now that we have built the contract, next step is to deploy the contract to the testnet. Here's a detailed breakdown of the steps involved, including wallet setup, deployment commands, and a touch on testnets:

1. **Wallet Setup:**

- Before deploying, we'll need a Fuel wallet to pay for the transaction fees associated with deployment.
- If we don't have a wallet installed, the forc deploy command will guide through the creation process. This usually involves generating a secure seed phrase that we should keep confidential.
- Once created, the terminal will ask us to set a password for the wallet. Choose a strong and unique password to safeguard the wallet.

2. **Securing Funds (for Testnet Deployment):**

- We will be deploying our contract to a testnet. Testnets are special blockchain environments that mimic the functionality of a mainnet but operate with fake currency (testnet FUEL tokens).
- Since we're deploying to a testnet (beta-5 testnet), we'll need testnet coins (FUEL tokens) to cover the deployment fees, we can use the [beta-5 faucet](https://faucet-beta-5.fuel.network/) to get some testnet coins.

3. **The Deployment Command:**

- Once our wallet is set up and funded (with testnet FUEL for deployment fees), we can use the following command to deploy our contract:

```shell
forc deploy --testnet
```

4. **Wallet Interaction and Confirmation:**

- The terminal will prompt us for our wallet's password to unlock it and authorize the deployment transaction. Enter the password securely.
- The terminal will then display a list of accounts associated with the wallet. Choose the account you want to use for signing the deployment transaction by entering its corresponding index number.
- Finally, the terminal will ask for confirmation before proceeding with the deployment. Type Y (yes) when ready to deploy.

5. **Deployment Success and Information:**

- Upon successful deployment, the terminal will display valuable information:

  - Network Endpoint: This is the URL of the Fuel network we deployed to (e.g., https://beta-5.fuel.network).
  - Contract ID: This is a unique identifier for our deployed contract on the blockchain. We'll need this ID to interact with our contract from applications or other contracts.
  - Deployed in Block: This indicates the block number on the blockchain where our deployment transaction was included.

**Congratulations on Building the First Fuel Contract!**

We've successfully navigated the process of writing, building, and deploying our first smart contract on the Fuel network!

Ready to Explore Further?

To delve deeper and see the complete code for this contract, check out the project repository [here](https://github.com/RobinNagpal/fuels-token-example-code)
