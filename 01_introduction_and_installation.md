# Blockchain DApps and Fuel

Decentralized applications (DApps) have a different architecture compared to traditional web applications. The table
below shows the components of a typical DApp and how they interact with each other.

| Component            | Interactions                                                                                                                                                                                | Fuel's Components                                                                                                                                                                                                                                                        |
|----------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **User (Browser)**   | User loads the DApp through a web browser (like Chrome).                                                                                                                                    | N/A                                                                                                                                                                                                                                                                      |
| **Wallet**           | A digital wallet (like MetaMask) holds your cryptocurrencies and manages your blockchain identity. The wallet is necessary for signing transactions and interacting securely with the DApp. | Fuel provides a [Browser Extension Wallet](https://wallet.fuel.network/docs/install/) and a command line utility - [forc-wallet](https://github.com/FuelLabs/forc-wallet) for this purpose.                                                                              |
| **Frontend/UI**      | The part of the DApp that you see and interact with. It is built using typical web technologies like HTML, CSS, and JavaScript.                                                             | You can use the [Fuel TS SDK](https://github.com/FuelLabs/fuels-ts) that accelerates development by providing libraries to simplify pulling data and interacting with the Fuel Chain.                                                                                    |
| **Backend**          | Unlike traditional apps, most DApps do not have a conventional backend (like servers or databases). Instead, the backend logic is mainly handled by smart contracts on the blockchain.      | If you have a backend that needs to interact with smart contracts, you can use the [Fuel TS SDK](https://github.com/FuelLabs/fuels-ts) or [Fuel Rust SDK](https://github.com/FuelLabs/fuels-rust).                                                                       |
| **JsonRPC Provider** | This component allows the DApp's frontend to communicate with the Fuel blockchain. It uses the JSON-RPC protocol to send and receive messages to the blockchain.                            |                                                                                                                                                                                                                                                                          |
| **Smart Contracts**  | Programs stored on the blockchain that run when predetermined conditions are met. They handle the business logic of the DApp, such as processing transactions or managing data.             | These are written in [Sway (Fuel's DSL)](https://github.com/FuelLabs/sway), which makes it very easy to write bug-free code. Fuel also provides [Sway Libs](https://github.com/FuelLabs/sway-libs) which contains a lot of code that can be reused in your applications. |
| **Fuel Chain**       | This is the blockchain network where the smart contracts are deployed. Fuel provides the environment that allows the smart contracts to run securely and transparently.                     | [fuel-core](https://github.com/FuelLabs/fuel-core) is the Fuel client implementation.                                                                                                                                                                                    |
| **Indexer**          | A tool that reads data from the blockchain and organizes it in a way that makes it easier and faster for the DApp to query and retrieve information.                                        | [fuel-indexer](https://github.com/FuelLabs/fuel-indexer/) is a standalone service that can be used to index various components of the blockchain.                                                                                                                        |

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

Here are some tools developers might use to build a DApp on Fuel, along with comparisons to Ethereum:

| Tool                        | Description                                                          | Fuel's Equivalent                     | Ethereum's Equivalent |
|-----------------------------|----------------------------------------------------------------------|---------------------------------------|-----------------------|
| **Smart Contract Language** | Language used to write smart contracts.                              | Sway                                  | Solidity              |
| **Deployment Tool**         | Compiles, runs tests, and deploys smart contracts to the blockchain. | forc, fuels-toolchain                 | Hardhat, Foundry      |
| **Wallet**                  | Manages blockchain identity and transactions.                        | forc-wallet, Wallet Browser Extension | MetaMask              |
| **Node**                    | Full node implementation of Fuel.                                    | fuel-core                             | Geth                  |
| **Indexer**                 | Indexes data from the blockchain.                                    | fuel-indexer                          | The Graph             |

# Installation Instructions

`fuelup` is the official package manager for Fuel, which installs most components of the Fuel ecosystem. Using 
`fuel up`, the Fuel Toolchain installs components like `forc`, `forc-client`, `forc-fmt`, `forc-crypto`, `forc-debug`, 
`forc-lsp`, `forc-wallet`, and `fuel-core` from the official release channels.

Installation is done through the `fuelup-init` script. The script downloads the core Fuel binaries:

```sh
curl -fsSL https://install.fuel.network/ | sh
```

During the installation, the script will ask for permission to add `~/.fuelup/bin` to your `PATH`.

To verify that the installation was successful, run the following command:

```
fuelup --version
```

This command should display the version of the `fuelup` tool, confirming its installation.

For detailed installation instructions, refer to the [Fuel Installation Guide](https://install.fuel.network/master/).

# Forc

Forc, or Fuel Orchestrator, offers a variety of tools and commands for developers working within the Fuel ecosystem. 
It provides functionality for scaffolding new projects, formatting code, running scripts, deploying contracts, testing 
contracts, and more.

We will use Forc in the Contracts section of this tutorial.

# Fuels CLI
Fuels CLI is a command-line tool that assists in building, deploying, and interacting with smart contracts. It's 
built on top of the `forc` command-line tool. We will learn more about the Fuels CLI in the next section. 
