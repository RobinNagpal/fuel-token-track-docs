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

# A Forc Project

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

The project is a _contract_, one of four different project types. For additional information on different project types, see [here](../sway-program-types/index.md).

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

## Testing the Contract

- Next, there should be a section for testing.
- Since the testing is purely based on Rust, this section should have some intro for testing in Rust, and should have instructions for setting up the Rust environment.
    - Cargo
    - Files generated by running the command `cargo generate` (Cargo.toml and test/harness.ts)
    - Explanation of the tests in harness.ts
- Once testing is done, next we can move to the deployment section.

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

