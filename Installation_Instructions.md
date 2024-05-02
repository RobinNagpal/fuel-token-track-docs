### Todo
- [ ] Explain in couple of lines that fuel is a blockchain, but also provides tooling making it seamless for
develops to not only write contracts but also develop front end applications that interacts with the smart contracts.
- [ ] Explain that there are three main components of the application, the node/blockchain, the smart contracts and the front end application.
- [ ] Then explain the components present under each of these three main components.

# Installation Instructions

To install the Fuel toolchain, which provides essential tools for developing and interacting with the Fuel blockchain network, you can use the `fuelup-init` script.

```
curl https://install.fuel.network | sh
```

This script will install the following tools in the `~/.fuelup/bin` directory:

- **forc:** A command-line tool specifically designed for working with Fuel smart contracts written in Sway. It offers functionalities like setting up a new Fuel project directory with a basic structure, compiling Sway code into the Fuel VM bytecode, deploying contracts to the network, and more.
- **forc-fmt:** A tool for formatting Sway code according to Fuel's style guidelines, improving readability and maintainability.
- **forc-lsp:** A Language Server Protocol (LSP) implementation for Sway, providing features like code completion, syntax highlighting, and type checking within code editors.
- **forc-wallet:** A tool for managing Fuel wallets, including generating keypairs, signing transactions, and interacting with the network.
- **fuel-core:** The official Fuel client implementation written in Rust. It's responsible for running Fuel nodes, validating transactions. You can think of it as the execution layer of the Fuel blockchain.

To verify that the installation was successful, run the following command:

```
fuelup --version
```

This should display the version of the fuelup tool, confirming its installation.
