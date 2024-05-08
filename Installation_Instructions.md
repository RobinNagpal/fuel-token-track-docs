### Todo

- [ ] Explain in couple of lines that fuel is a blockchain, but also provides tooling making it seamless for
  develops to not only write contracts but also develop front end applications that interacts with the smart contracts.
- [ ] Explain that there are three main components of the application, the node/blockchain, the smart contracts and the
  front end application.
- [ ] Then explain the components present under each of these three main components.

### Questions

- Should we create a diagram?
- What is Sway Toolchain? https://fuellabs.github.io/sway/v0.43.2/book/introduction/installation.html

# Blockchain DApps and Fuel

A decentralized application (DApp) is a type of software application that runs on a blockchain network instead of being
hosted on centralized servers. Here’s how the various components interact in a typical blockchain based DApp:

| Component            | Interactions                                                                                                                                                                                     | Fuel's Components                                                                                                                                                                                                                                                     |
|----------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **User(Browser)**    | User loads the DApp through a web browser(like Chrome).                                                                                                                                          | N/A                                                                                                                                                                                                                                                                   |
| **Wallet**           | A digital wallet (like MetaMask) that holds your cryptocurrencies and manages your blockchain identity. The wallet is necessary for signing transactions and interacting securely with the DApp. | Fuel provides a [Browser Extension Wallet](https://wallet.fuel.network/docs/install/) and a command line utility - [forc-wallet](!https://github.com/FuelLabs/forc-wallet) for this.                                                                                  |               
| **Frontend/UI**      | Part of the DApp that you see and interact with. It's built using typical web technologies like HTML, CSS, and JavaScript.                                                                       | You can use [Fuel TS SDK](https://github.com/FuelLabs/fuels-ts) that accelerates the development by providing libraries that simplify pulling data and interacting with Fuel Chain.                                                                                   |
| **Backend**          | Unlike traditional apps, most DApps don’t have a conventional backend (like servers or databases). Instead, the backend logic is mainly handled by smart contracts on the blockchain.            | In cases you have backend which needs to interact with Smart Contracts, you can use [Fuel TS SDK](https://github.com/FuelLabs/fuels-ts) or [Fuel Rust SDK](https://github.com/FuelLabs/fuels-rust).                                                                   |
| **JsonRPC Provider** | This component allows the DApp's frontend to communicate with the Fuel blockchain. It uses a protocol called JSON-RPC to send and receive messages to the blockchain.                            |                                                                                                                                                                                                                                                                       |
| **Smart Contracts**  | These are programs stored on the blockchain that run when predetermined conditions are met. They handle the business logic of the DApp, such as processing transactions or managing data         | These are written in [Sway(Fuel's DSL)](https://github.com/FuelLabs/sway) which makes it very easy to write bug free code. Fuel also provides [Sway Libs](https://github.com/FuelLabs/sway-libs) which contains a lot of code that can be reused in your applications | 
| **Fuel Chain**       | This is the blockchain network where the smart contracts are deployed. Fuel provides the environment that allows the smart contracts to run securely and transparently                           | [fuel-core](https://github.com/FuelLabs/fuel-core) is Fuel client implementation.                                                                                                                                                                                     | 
| **Indexer**          | This is a tool that reads data from the blockchain and organizes it in a way that makes it easier and faster for the DApp to query and retrieve information.                                     | [fuel-indexer](https://github.com/FuelLabs/fuel-indexer/) is a standalone service that can be used to index various components of the blockchain                                                                                                                      |

![Fuel Dapp](https://raw.githubusercontent.com/RobinNagpal/fuels-token-example/main/assets/images/fuel_dapp.png)

#### Flow of a typical transaction in this DApp might look like this:

In a typical DApp transaction, you start by using your browser to interact with the DApp's interface. When you initiate
a transaction, such as sending tokens or activating a smart contract function, the interface sends this transaction
request to your digital wallet. Your wallet then prompts you to sign the transaction, which verifies your identity and
approves the transaction. Once you sign, your wallet sends the transaction over to the Ethereum blockchain through a
JsonRPC Provider. On the blockchain, the smart contract related to your transaction is activated and executes the
transaction. After the transaction is completed, the results are recorded on the blockchain and organized by an indexer,
making the information easily accessible for later use. Finally, the DApp's interface updates to show the new status,
reflecting the changes made by the transaction.

# Development Tools

Here are some of the tools a developer might use to build a DApp on Fuel and also how it compares to ethereum:

| Tool                         | Description                                  | Fuel's Equivalent     | Ethereum's Equivalent    |
|------------------------------|----------------------------------------------|-----------------------|--------------------------|
| **Smart Contract Languages** | Language for writing smart contracts on Fuel | Sway                  | Solidity                 |
| **Deployment Tool**          | Deploys smart contracts to the blockchain    | forc, fuels-toolchain | Hardhat, Foundry         |
| **Wallet**                   | Manages blockchain identity and transactions | forc-wallet           | MetaMask                 |
| **Node**                     | Full node implementation of Fuel             | fuel-core             | Geth                     |
| **Indexer**                  | Indexes data from the blockchain             | fuel-indexer          | The Graph                |
| **Frontend SDK**             | Libraries for interacting with Fuel Chain    | fuel-ts               | Web3.js, viem, ethers.js |
| **Backend SDK**              | Libraries for interacting with Fuel Chain    | fuel-ts / fuel-rs     | Web3.js, viem, ethers.js |
| **Testing Framework**        | Framework for testing smart contracts        |                       |                          |


# Installation Instructions

`fuelup` is the official package manager for Fuel that installs most of the Fuel's components. `fuel up` installs these 
components are as part the Fuel Toolchain from the official release channels. 

Using the fuelup, Fuel toolchain will install `forc`, `forc-client`, `forc-fmt`, `forc-crypto`, `forc-debug`, 
`forc-lsp`, `forc-wallet` as well as `fuel-core`.

Installation is done through the fuelup-init script.sh, which downloads the core Fuel binaries.
```sh
curl -fsSL https://install.fuel.network/ | sh
```
The script will ask for permission to add `~/.fuelup/bin` to your `PATH`.

To verify that the installation was successful, run the following command:

```
fuelup --version
```

This should display the version of the fuelup tool, confirming its installation.

For detailed installation instructions, refer to the [Fuel Installation Guide](https://install.fuel.network/master/).

# Forc
Forc stands for Fuel Orchestrator. Forc provides a variety of tools and commands for developers working with the Fuel
ecosystem, such as scaffolding a new project, formatting, running scripts, deploying contracts, testing contracts, and
more.

We will be using forc in the Contracts section of this tutorial.

Some of the important commands provided by forc are:
- `forc init` - Create a new Forc project in an existing directory.
- `forc new` -  Create a new Forc project at <path>.

#### forc build

Compile the current or target project. The output produced will depend on the project's program type.

#### forc-test

Run the Sway unit tests for the current project.

### Forc wallet

https://github.com/FuelLabs/forc-wallet

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

### fuel-core

Full node implementation of the Fuel v2 protocol, written in Rust.

## Sway

#### Sway Language Server (`forc-lsp`)

The Sway Language Server `forc-lsp` is provided to expose features to
IDEs. [Installation instructions](../lsp/installation.md).

#### Sway Formatter (`forc-fmt`)

A canonical formatter is provided with `forc-fmt`. [Installation instructions](./getting_started.md). It can be run
manually with

```sh
forc fmt
```

The [Visual Studio Code plugin](https://marketplace.visualstudio.com/items?itemName=FuelLabs.sway-vscode-plugin) will
automatically format Sway files with `forc-fmt` on save, though you might have to explicitly set the Sway plugin as the
default formatter, like this:

```json
"[sway]": {
"editor.defaultFormatter": "FuelLabs.sway-vscode-plugin"
}
```

## Other Components

https://install.fuel.network/master/concepts/components.html


