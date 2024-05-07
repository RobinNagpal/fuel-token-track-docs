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

The Fuel toolchain is your one-stop shop for building amazing things with Sway, a powerful language for creating smart contracts. Think of it as a toolbox packed with everything you need to develop, test, and deploy your Sway creations.
The Fuel toolchain installation is designed to be quick and easy. All you need is a single script, fuelup-init.sh, which downloads the core Fuel binaries to get you started. Currently, this script supports Linux/macOS systems only. For other systems, please [read the Installation chapter](https://fuellabs.github.io/fuelup/master/installation/other.html).

```sh
curl -fsSL https://install.fuel.network/ | sh
```
<!-- install:example:end -->

This will install `forc`, `forc-client`, `forc-fmt`, `forc-crypto`, `forc-debug`, `forc-lsp`, `forc-wallet` as well as `fuel-core` in `~/.fuelup/bin`. The script will ask for permission to add `~/.fuelup/bin` to your `PATH`.

To verify that the installation was successful, run the following command:

```
fuelup --version
```
This should display the version of the fuelup tool, confirming its installation.


Many fuelup commands deal with toolchains, a single installation of the Fuel toolchain. fuelup supports two types of toolchains.

1. Distributable toolchains which track the official release channels (e.g., latest, nightly);
2. Custom toolchains and install individual components in a modular manner.  


## Install Fuelup
https://github.com/FuelLabs/fuelup

https://install.fuel.network/master/index.html

## Components
Each toolchain has several "components", which are tools used to develop on Fuel.

The `fuelup component` command is used to manage the installed components.

Components can be added to an already-installed toolchain with the fuelup component command:

```
fuelup component add forc
```

Next we go through some components installable through fuelup:

### Forc
Forc stands for Fuel Orchestrator. Forc provides a variety of tools and commands for developers working with the Fuel ecosystem, such as scaffolding a new project, formatting, running scripts, deploying contracts, testing contracts, and more.

#### forc init
Create a new Forc project in an existing directory.

#### forc new
Create a new Forc project at <path>.

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

### fuel-core
Full node implementation of the Fuel v2 protocol, written in Rust.

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


