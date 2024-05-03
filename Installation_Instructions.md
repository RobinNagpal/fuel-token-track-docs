### Todo

- [ ] Explain in couple of lines that fuel is a blockchain, but also provides tooling making it seamless for
  develops to not only write contracts but also develop front end applications that interacts with the smart contracts.
- [ ] Explain that there are three main components of the application, the node/blockchain, the smart contracts and the
  front end application.
- [ ] Then explain the components present under each of these three main components.

### Questions

- Should we create a diagram?
- What is Sway Toolchain? https://fuellabs.github.io/sway/v0.43.2/book/introduction/installation.html

### Notes

This installation guide is quite good and we should borrow information from
here https://fuellabs.github.io/sway/v0.43.2/book/introduction/installation.html

# Installation Instructions

To install the Fuel toolchain, which provides essential tools for developing and interacting with the Fuel blockchain
network, you can use the `fuelup-init` script.

```
curl https://install.fuel.network | sh
```

This script will install the following tools in the `~/.fuelup/bin` directory:

- **forc:** A command-line tool specifically designed for working with Fuel smart contracts written in Sway. It offers
  functionalities like setting up a new Fuel project directory with a basic structure, compiling Sway code into the Fuel
  VM bytecode, deploying contracts to the network, and more.
- **forc-fmt:** A tool for formatting Sway code according to Fuel's style guidelines, improving readability and
  maintainability.
- **forc-lsp:** Do we need to explain it here?. A Language Server Protocol (LSP) implementation for Sway, providing
  features like code completion, syntax highlighting, and type checking within code editors.
- **forc-wallet:** A tool for managing Fuel wallets, including generating keypairs, signing transactions, and
  interacting with the network.
- **fuel-core:** The official Fuel client implementation written in Rust. It's responsible for running Fuel nodes,
  validating transactions. You can think of it as the execution layer of the Fuel blockchain.
- **fuel-ts**: TODO
-

To verify that the installation was successful, run the following command:

```
fuelup --version
```

Many fuelup commands deal with toolchains, a single installation of the Fuel toolchain. fuelup supports two types of 
toolchains.

This should display the version of the fuelup tool, confirming its installation.

* Sway Toolchain - The Sway toolchain is sufficient to compile Sway smart contracts. Otherwise, note that if you want to run Sway smart contracts (e.g. for testing), a Fuel Core full node is required, which is packaged together with the Sway toolchain together as the Fuel toolchain
* Fuelup - `fuelup` installs the Fuel toolchain from our official release channels, enabling you to easily keep the toolchain updated.   



## Install Fuelup
https://github.com/FuelLabs/fuelup

https://install.fuel.network/master/index.html



Currently, this script supports Linux/macOS systems only. For other systems, please [read the Installation chapter](https://fuellabs.github.io/fuelup/master/installation/other.html).

Installation is simple: all you need is `fuelup-init.sh`, which downloads the core Fuel binaries needed to get you started on development.

```sh
curl -fsSL https://install.fuel.network/ | sh
```
<!-- install:example:end -->

This will install `forc`, `forc-client`, `forc-fmt`, `forc-crypto`, `forc-debug`, `forc-lsp`, `forc-wallet` as well as `fuel-core` in `~/.fuelup/bin`. The script will ask for permission to add `~/.fuelup/bin` to your `PATH`.


## Forc wallet
https://github.com/FuelLabs/forc-wallet

A forc plugin for managing Fuel wallets.

#### Create a wallet

Before creating accounts and signing transactions with them you need to create a wallet. To do so:

```sh
forc-wallet new
```

This will require a password for encrypting the wallet. After the wallet is created you will be shown the mnemonic phrase.

> Note: You will need your password for signing and account derivation, and you will need your mnemonic phrase if you wish to recover your wallet in the future.

#### Import a wallet

To import a wallet from an existing mnemonic phrase, use:

```sh
forc-wallet import
```

> Note: `forc-wallet` adheres to the [Web3 Secret Storage Definition](https://ethereum.org/en/developers/docs/data-structures-and-encoding/web3-secret-storage) and accepts paths to wallet files that adhere to this standard.

#### Create an account

To create an account for the wallet, you can run:

```sh
forc-wallet account new
```

This will require your wallet password (the one that you chose during creation). This command will always derive the next account that has not yet been derived locally.

To list all accounts derived so far, use the following:

```sh
forc-wallet accounts
```

## Sway 
#### Sway Language Server (`forc-lsp`)

The Sway Language Server `forc-lsp` is provided to expose features to IDEs. [Installation instructions](../lsp/installation.md).

#### Sway Formatter (`forc-fmt`)

A canonical formatter is provided with `forc-fmt`. [Installation instructions](./getting_started.md). It can be run manually with

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


