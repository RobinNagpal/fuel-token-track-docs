-----
--------- Ignore below this line for now --------
-----

# Contract

## ToDo

- Explain the project structure and how to initialize the project using forc cli.
- Explain the various files and what is their purpose. (like Forc.toml, main.sw)
- A bit of explanation about libraries and how to import them. At the very least, explanation about the std library and why we don’t import it explicitly (cause of standard library prelude)
    - std (this is included by default)
- Start with coding of the contract and explain the concepts like ABI and implementation of the ABI alongside, also a bit about the data types in Sway.
    - contract keyword (program type in sway)
    - ABIs
    - Defining a function in Sway
    - Return types and variables in Sway
    - implicit and explicit return types in Sway
    - Option and Result types in Sway
- Concepts like storage and configurable constants to follow next.
    - variables in storage block
    - Advanced data structures like Hashmap and vector
    - How to create configurable constants
    - Write to storage
    - Read from storage
- Compilation
- After coding of the contract is completed, next move to building the project, necessary commands to build the project, explaining the output of the build command and explaining the purpose of each of the files in the out directory.
    - TokenTrack-abi.json
    - TokenTrack-storage_slots.json
    - TokenTrack-wallet.bin


# New Forc Project

To initialize a new project with Forc, use `forc new`:

```sh
forc new my-fuel-project
```

Here is the project that Forc has initialized:

<!-- This section should show the tree for a new forc project -->
<!-- tree:example:start -->
```console
$ cd my-fuel-project
$ tree .
├── Forc.toml
└── src
    └── main.sw
```
<!-- tree:example:end -->

<!-- This section should explain the `Forc.toml` file -->
<!-- forc_toml:example:start -->
`Forc.toml` is the _manifest file_ (similar to `Cargo.toml` for Cargo or `package.json` for Node), and defines project metadata such as the project name and dependencies.
<!-- forc_toml:example:end -->

For additional information on dependency management, see: [here](../forc/dependencies.md).

```toml
[project]
authors = ["User"]
entry = "main.sw"
license = "Apache-2.0"
name = "my-fuel-project"

[dependencies]
```

Here are the contents of the only Sway file in the project, and the main entry point, `src/main.sw`:

```sway
contract;

abi MyContract {
    fn test_function() -> bool;
}

impl MyContract for Contract {
    fn test_function() -> bool {
        true
    }
}
```

The project is a _contract_, one of four different project types. For additional information on different project
types, see [here](../sway-program-types/index.md).

# Forc wallet

A forc plugin for managing Fuel wallets.

#### Create a wallet

Before creating accounts and signing transactions with them you need to create a wallet. To do so:

```sh
forc-wallet new
```

This will require a password for encrypting the wallet. After the wallet is created you will be shown the mnemonic
phrase.

> Note: You will need your password for signing and account derivation, and you will need your mnemonic phrase if you
> wish to recover your wallet in the future.

#### Import a wallet

To import a wallet from an existing mnemonic phrase, use:

```sh
forc-wallet import
```

> Note: `forc-wallet` adheres to
>
the [Web3 Secret Storage Definition](https://ethereum.org/en/developers/docs/data-structures-and-encoding/web3-secret-storage)
> and accepts paths to wallet files that adhere to this standard.

#### Create an account

To create an account for the wallet, you can run:

```sh
forc-wallet account new
```

This will require your wallet password (the one that you chose during creation). This command will always derive the
next account that has not yet been derived locally.

To list all accounts derived so far, use the following:

```sh
forc-wallet accounts
```




## Contract

#### Syntax of a Smart Contract
https://fuellabs.github.io/sway/v0.19.0/sway-program-types/smart_contracts.html

As with any Sway program, the program starts with a declaration of what [program type](./index.md) it is. A contract must also either define or import an [ABI declaration](#the-abi-declaration) and implement it.


It is considered good practice to define your ABI in a separate library and import it into your contract. This allows callers of your contract to simply import the ABI directly and use it in their scripts to call your contract.

```sway
library wallet_abi;

abi Wallet {
    fn receive_funds();
    fn send_funds(amount_to_send: u64, recipient_address: b256);
}
```
#### Blockchain Types

Sway is fundamentally a blockchain language, and it offers a selection of types tailored for the blockchain use case.

These are provided via the standard library ([`lib-std`](https://github.com/FuelLabs/sway/tree/master/sway-lib-std)) which both add a degree of type-safety, as well as make the intention of the developer more clear.

###### `Address` Type

<!-- This section should explain the `Address` type -->
<!-- address:example:start -->
The `Address` type is a type-safe wrapper around the primitive `b256` type. Unlike the EVM, an address **never** refers to a deployed smart contract (see the `ContractId` type below). An `Address` can be either the hash of a public key (effectively an [externally owned account](https://ethereum.org/en/whitepaper/#ethereum-accounts) if you're coming from the EVM) or the hash of a [predicate](../sway-program-types/predicates.md). Addresses own UTXOs.
<!-- address:example:end -->

An `Address` is implemented as follows.

```sway
pub struct Address {
    value: b256,
}
```

Casting between the `b256` and `Address` types must be done explicitly:

```sway
let my_number: b256 = 0x000000000000000000000000000000000000000000000000000000000000002A;
let my_address: Address = Address::from(my_number);
let forty_two: b256 = my_address.into();
```

###### `ContractId` Type

<!-- This section should explain the `ContractId` type -->
<!-- contract_id:example:start -->
The `ContractId` type is a type-safe wrapper around the primitive `b256` type. A contract's ID is a unique, deterministic identifier analogous to a contract's address in the EVM. Contracts cannot own UTXOs but can own assets.
<!-- contract_id:example:end -->

A `ContractId` is implemented as follows.

```sway
pub struct ContractId {
    value: b256,
}
```

Casting between the `b256` and `ContractId` types must be done explicitly:

```sway
let my_number: b256 = 0x000000000000000000000000000000000000000000000000000000000000002A;
let my_contract_id: ContractId = ContractId::from(my_number);
let forty_two: b256 = my_contract_id.into();
```

###### `Identity` Type

<!-- This section should explain the `Identity` type -->
<!-- identity:example:start -->
The `Identity` type is an enum that allows for the handling of both `Address` and `ContractId` types. This is useful in cases where either type is accepted, e.g., receiving funds from an identified sender, but not caring if the sender is an address or a contract.
<!-- identity:example:end -->

An `Identity` is implemented as follows.

```sway
{{#include ../../../../sway-lib-std/src/identity.sw:docs_identity}}
```

Casting to an `Identity` must be done explicitly:

```sway
{{#include ../../../../examples/identity/src/main.sw:cast_to_identity}}
```

A `match` statement can be used to return to an `Address` or `ContractId` as well as handle cases in which their execution differs.

```sway
{{#include ../../../../examples/identity/src/main.sw:identity_to_contract_id}}
```

```sway
{{#include ../../../../examples/identity/src/main.sw:different_executions}}
```
<!-- This section should explain the use case for the `Identity` type -->
<!-- use_identity:example:start -->
A common use case for `Identity` is for access control. The use of `Identity` uniquely allows both `ContractId` and `Address` to have access control inclusively.
<!-- use_identity:example:end -->

```sway
{{#include ../../../../examples/identity/src/main.sw:access_control_with_identity}}
```

### Deploying the Contract

- Explain deploying the contract to the testnet using forc CLI.
- Instructions for setting up the wallet locally plus how to transfer test coins using the faucet.

### Storage

https://fuellabs.github.io/sway/v0.43.2/book/blockchain-development/storage.html
When developing a smart contract, you will typically need some sort of persistent storage. In this case, persistent
storage, often just called storage in this context, is a place where you can store values that are persisted inside
the contract itself. This is in contrast to a regular value in memory, which disappears after the contract exits.

Put in conventional programming terms, contract storage is like saving data to a hard drive. That data is saved even
after the program which saved it exits. That data is persistent. Using memory is like declaring a variable in a
program: it exists for the duration of the program and is non-persistent.

Some basic use cases of storage include declaring an owner address for a contract and saving balances in a wallet.

https://fuellabs.github.io/sway/v0.43.2/book/reference/attributes.html#storage

In Sway, functions are pure by default but can be opted into impurity via the storage function attribute. The
storage attribute may take read and/or write arguments indicating which type of access the function requires.

The #[storage(read)] attribute indicates that a function requires read access to the storage.

The #[storage(write)] attribute indicates that a function requires write access to the storage.

#### Storage Maps

https://fuellabs.github.io/sway/v0.43.2/book/blockchain-development/storage.html#storage-maps




# Forc
As mentioned before, `Forc` provides a variety of tools and commands for Smart Contract development, such as
scaffolding a new project, formatting, running scripts, deploying contracts, testing contracts, and
more.

Some of the most used commands provided by forc are:
- `forc init` - Create a new Forc project in an existing directory.
- `forc new` -  Create a new Forc project at <path>.
- `forc build` - Compile the current or target project. The output produced will depend on the project's program type.
- `forc-test` - Run the Sway unit tests for the current project.


---- 

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
  - sway-contracts-api folder
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
