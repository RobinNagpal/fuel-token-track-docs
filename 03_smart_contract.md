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

Let's examine the contents of `main.sw`.

The first line in the file is:

```
contract
```

In Sway, a program must declare its type. It can be a contract, a predicate, a script, or a library. Contracts,
predicates, and scripts are deployable to the blockchain, while a library is for code reuse and isn't deployed directly.

A Sway file begins by stating its type. A forc project can include many libraries but only one contract, script, or predicate.
Contracts are typically used for protocols or systems that adhere to a set of rules.

The next element in `main.sw` is an ABI.

### ABI

An ABI (Application Binary Interface) outlines an interface for the contract without including the actual function implementation. A contract must define
or import an ABI declaration and implement it. Best practice is to define the ABI in a separate library and import
it into the contract. This allows others to import and use the ABI in their scripts to interact with the contract.

After importing or declaring the ABI in the `main.sw` file, the contract's functions, as defined in the ABI, are implemented.

# Token Track Contract

This section guides us on how to write a contract for Token Track Project using the Fuel network with Sway. Here, we will
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

In Sway, functions are 'pure' by default, meaning they do not access any persistent storage. However, they can be modified to access storage by using the `storage` function attribute. This attribute can take `read` or `write` arguments to specify the required type of access:

- The `#[storage(read)]` attribute indicates that a function needs read access to the storage.
- The `#[storage(write)]` attribute indicates that a function needs write access to the storage.

## Contract Code - Implementation of ABI

Now that we understand the ABI and storage, let's examine the `mint_to_address` function in our Sway contract.

### Minting Functionality

This function is designed to create new tokens and add them directly to a specific address on the Fuel network. Here's
the basic code for the `mint_to_address` function:

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

## Building the Contract

The "Build" phase is essential in preparing our smart contract for deployment on the Fuel network. Smart contracts are typically written in high-level languages like Sway, which are easy to read and maintain. However, the Fuel network operates on bytecode, a low-level machine-readable format. Build process translates our Sway code into bytecode that the Fuel network can execute.

Navigate to the contract folder:

```shell
cd TokenTrack
```

Then run the command to build the contract:

```shell
forc build
```

### Output

A successful build will create a new directory named `out` within the contract folder. This directory contains several important files:

- **debug:** This folder contains debugging information which will be useful during deployment.

  - **TokenTrack-contract-abi.json:** This file stores the contract's public Application Binary Interface (ABI) in JSON format, which describes how other applications and contracts can interact with it.
  - **TokenTrack-contract-storage_slots.json:** This file contains the storage layout of our contract, showing how data is stored on the blockchain.
  - **TokenTrack-contract.bin:** This file is the bytecode version of our Sway code, containing instructions that the Fuel network will execute upon deployment.

## Deploying the Contract (Testnet)

With the contract built, the next step is to deploy it to the testnet. Here are the steps involved, including wallet setup and deployment commands:

1. **Wallet Setup:**

   - You need a Fuel wallet to pay for the transaction fee.
   - If you don’t have a wallet, the `forc deploy [Options]` command will guide you through creating one, which includes generating a secure seed phrase to keep confidential.
   - Set a strong, unique password for your wallet when prompted.

2. **Securing Funds (for Testnet Deployment):**

   - We are deploying to a testnet, which uses fake currency (testnet FUEL tokens).
   - Obtain testnet coins to cover deployment fee from the [beta-5 faucet](https://faucet-beta-5.fuel.network/).

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

We have now seen how to deploy our contract to the testnet, let us also see how we can deploy the contract to a local node.

There are two types of Fuel networks that can be run:

1. In-memory network (without persistence)
2. Local network with persistence

For this example, we will be using an In-memory network.

An in-memory node does not persist the blockchain state anywhere, it is only stored in memory as long as the node is active and running.

To spin-up a local in-memory Fuel node, run the following command:

```bash
fuel-core run --db-type in-memory --debug
```

But since we need some initial state of the chain, we must configure a `chainConfig.json` file. Here is an example of what that looks like using version 0.22.1 of fuel-core:

<details>
  <summary>chainConfig.json</summary>

```js
{
"chain_name": "Testnet",
"block_gas_limit": 1000000000,
"initial_state": {
  "coins": [
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
    }
  ],
  "contracts": [
    {
      "contract_id": "0x7777777777777777777777777777777777777777777777777777777777777777",
      "code": "0x9000000994318e6e453f30e85bf6088f7161d44e57b86a6af0c955d22b353f91b2465f5e6140000a504d00205d4d30001a4860004945048076440001240400005050c0043d51345024040000",
      "salt": "0x1bfd51cb31b8d0bc7d93d38f97ab771267d8786ab87073e0c2b8f9ddc44b274e"
    }
  ]
},
"consensus_parameters": {
  "tx_params": {
    "max_inputs": 255,
    "max_outputs": 255,
    "max_witnesses": 255,
    "max_gas_per_tx": 1000000000,
    "max_size": 17825792
  },
  "predicate_params": {
    "max_predicate_length": 1048576,
    "max_predicate_data_length": 1048576,
    "max_message_data_length": 1048576,
    "max_gas_per_predicate": 1000000000
  },
  "script_params": {
    "max_script_length": 1048576,
    "max_script_data_length": 1048576
  },
  "contract_params": {
    "contract_max_size": 16777216,
    "max_storage_slots": 131072
  },
  "fee_params": {
    "gas_price_factor": 92,
    "gas_per_byte": 63
  },
  "chain_id": 0,
  "gas_costs": {
    "add": 2,
    "addi": 2,
    "aloc": 1,
    "and": 2,
    "andi": 2,
    "bal": 366,
    "bhei": 2,
    "bhsh": 2,
    "burn": 33949,
    "cb": 2,
    "cfei": 2,
    "cfsi": 2,
    "croo": 40,
    "div": 2,
    "divi": 2,
    "eck1": 3347,
    "ecr1": 46165,
    "ed19": 4210,
    "eq": 2,
    "exp": 2,
    "expi": 2,
    "flag": 1,
    "gm": 2,
    "gt": 2,
    "gtf": 16,
    "ji": 2,
    "jmp": 2,
    "jne": 2,
    "jnei": 2,
    "jnzi": 2,
    "jmpf": 2,
    "jmpb": 2,
    "jnzf": 2,
    "jnzb": 2,
    "jnef": 2,
    "jneb": 2,
    "lb": 2,
    "log": 754,
    "lt": 2,
    "lw": 2,
    "mint": 35718,
    "mlog": 2,
    "mod": 2,
    "modi": 2,
    "move": 2,
    "movi": 2,
    "mroo": 5,
    "mul": 2,
    "muli": 2,
    "mldv": 4,
    "noop": 1,
    "not": 2,
    "or": 2,
    "ori": 2,
    "poph": 3,
    "popl": 3,
    "pshh": 4,
    "pshl": 4,
    "ret_contract": 733,
    "rvrt_contract": 722,
    "sb": 2,
    "sll": 2,
    "slli": 2,
    "srl": 2,
    "srli": 2,
    "srw": 253,
    "sub": 2,
    "subi": 2,
    "sw": 2,
    "sww": 29053,
    "time": 79,
    "tr": 46242,
    "tro": 33251,
    "wdcm": 3,
    "wqcm": 3,
    "wdop": 3,
    "wqop": 3,
    "wdml": 3,
    "wqml": 4,
    "wddv": 5,
    "wqdv": 7,
    "wdmd": 11,
    "wqmd": 18,
    "wdam": 9,
    "wqam": 12,
    "wdmm": 11,
    "wqmm": 11,
    "xor": 2,
    "xori": 2,
    "call": {
      "LightOperation": {
        "base": 21687,
        "units_per_gas": 4
      }
    },
    "ccp": {
      "LightOperation": {
        "base": 59,
        "units_per_gas": 20
      }
    },
    "csiz": {
      "LightOperation": {
        "base": 59,
        "units_per_gas": 195
      }
    },
    "k256": {
      "LightOperation": {
        "base": 282,
        "units_per_gas": 3
      }
    },
    "ldc": {
      "LightOperation": {
        "base": 45,
        "units_per_gas": 65
      }
    },
    "logd": {
      "LightOperation": {
        "base": 1134,
        "units_per_gas": 2
      }
    },
    "mcl": {
      "LightOperation": {
        "base": 3,
        "units_per_gas": 523
      }
    },
    "mcli": {
      "LightOperation": {
        "base": 3,
        "units_per_gas": 526
      }
    },
    "mcp": {
      "LightOperation": {
        "base": 3,
        "units_per_gas": 448
      }
    },
    "mcpi": {
      "LightOperation": {
        "base": 7,
        "units_per_gas": 585
      }
    },
    "meq": {
      "LightOperation": {
        "base": 11,
        "units_per_gas": 1097
      }
    },
    "retd_contract": {
      "LightOperation": {
        "base": 1086,
        "units_per_gas": 2
      }
    },
    "s256": {
      "LightOperation": {
        "base": 45,
        "units_per_gas": 3
      }
    },
    "scwq": {
      "HeavyOperation": {
        "base": 30375,
        "gas_per_unit": 28628
      }
    },
    "smo": {
      "LightOperation": {
        "base": 64196,
        "units_per_gas": 1
      }
    },
    "srwq": {
      "HeavyOperation": {
        "base": 262,
        "gas_per_unit": 249
      }
    },
    "swwq": {
      "HeavyOperation": {
        "base": 28484,
        "gas_per_unit": 26613
      }
    },
    "contract_root": {
      "LightOperation": {
        "base": 45,
        "units_per_gas": 1
      }
    },
    "state_root": {
      "HeavyOperation": {
        "base": 350,
        "gas_per_unit": 176
      }
    },
    "new_storage_per_byte": 63,
    "vm_initialization": {
      "LightOperation": {
        "base": 1645,
        "units_per_gas": 14
      }
    }
  },
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

To get coins in your wallet, add the following entry to the coins array with your wallet address in B256 type address (begins with 0x):

```js
{
      "owner": "your wallet address",
      "amount": "0x1000000000000000",
      "asset_id": "0x0000000000000000000000000000000000000000000000000000000000000000"
    },

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
