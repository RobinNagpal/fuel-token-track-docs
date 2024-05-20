# Token Track Project

One of the most common uses for smart contracts is creating a token. In this example, we'll make a contract that can
mint, burn, and track a user's token balance. We'll also create a frontend to interact with the contract.

#### Here is how the UI will look like:

![User Interface](https://raw.githubusercontent.com/RobinNagpal/fuel-token-track-docs/main/assets/images/ui.png)

The UI allows to:

- Mint tokens
- Transfer tokens
- Burn tokens
- Check the balance of a user or a contract

# Project Structure

```
.
├── README.md
├── contracts
│   └── TokenTrack
│       ├── Forc.lock
│       ├── Forc.toml
│       ├── out
│       │   └── debug
│       │       ├── TokenTrack-abi.json
│       │       ├── TokenTrack-storage_slots.json
│       │       └── TokenTrack.bin
│       └── src
│           └── main.sw
├── frontend
│   ├── README.md
│   ├── package-lock.json
│   ├── package.json
│   ├── public
│   ├── src
│   │   ├── App.css
│   │   ├── App.tsx
│   │   ├── index.css
│   │   ├── index.tsx
│   │   ├── logo.svg
│   │   ├── react-app-env.d.ts
|   |   ├── hooks
|   |   │   ├── useActiveWallet.tsx
|   |   │   ├── useBrowserWallet.tsx
|   |   │   ├── useBurnerWallet.tsx
|   |   │   └── useFaucet.tsx
│   │   └── sway-api
│   │       ├── contracts
│   │       │   ├── TokenTrackAbi.d.ts
│   │       │   ├── TokenTrackAbi.hex.ts
│   │       │   ├── common.d.ts
│   │       │   ├── factories
│   │       │   │   └── TokenTrackAbi__factory.ts
│   │       │   └── index.ts
│   │       └── index.ts
│   └── tsconfig.json
├── fuels.config.ts
└── package.json
```

**Main Folders and Files:**

- `contracts`: This holds the smart contract code. We'll discuss the details in the 'Smart Contract' section.
- `frontend`: Here we'll find the frontend code. We'll dive into the frontend in the 'Frontend' section.
- `fuels.config.ts`: This file contains the configuration for the Fuels CLI.

# Creating a New Project

Lets create the directory structure for the project.

```bash
mkdir fuel-project
cd fuel-project

mkdir -p contracts/TokenTrack
mkdir frontend

```

Now let's create the `fuels.config.ts` file in the root directory.

```bash
npx fuels@0.84.0 init --contracts ./contracts/TokenTrack/ --output ./frontend/src/sway-contracts-api
npm install fuels@0.84.0
```

This will create a `fuels.config.ts` file with the following content:

```typescript
import { createConfig } from "fuels";

export default createConfig({
  contracts: ["./contracts/TokenTrack"],
  output: "./frontend/src/sway-api",
});
```

`Fuels CLI` has many commands that can be used to build, deploy, and interact with smart contracts. We'll discuss these

# Fuels CLI

In this project, we'll use the Fuels CLI to construct and deploy our contracts. The Fuels CLI is a command-line tool
that assists in building, deploying, and interacting with smart contracts. It's built on top of the `forc` command-line
tool. Configuration for the Fuels CLI is done through a `fuels.config.ts` file.

#### fuels.config.ts

In our case, we've specified the `contracts` and `output` fields:

- The `contracts` field specifies the path to the contract directories.
- The `output` field specifies where the generated files will be saved.

The `fuels.config.ts` configuration consists of following fields:

```
workspace?: string;
contracts?: string[];
scripts?: string[];
predicates?: string[];
output: string;
useBuiltinForc?: boolean;
useBuiltinFuelCore?: boolean;
autoStartFuelCore?: boolean;
```

The workspace field specifies the path to the workspace directory. In our case, we have are not using the workspace
and have specified the contracts field.

For other properties in this file you can visit [here](https://docs.fuel.network/docs/nightly/fuels-ts/fuels-cli/config-file/)


# Build and Deploy
We will be using `fuels dev` command to build and deploy the contracts.

```
npx fuels@0.84.0 dev
```

The `fuels dev` command does three things:

1. Automatically starts a short-lived fuel-core node.
2. Runs build and deploy once at the start.
3. If autoStartFuelCore is enabled, it watches your Forc workspace and repeats the previous steps on every change.
4. Saves their deployed IDs of contracts to: {output_in_config_file}/contract-ids.json

# Important Notes
Some of the important points to note are:
* In the `fuels.config.ts` file, we have specified the contracts and output fields, but we could have also specified the workspace field instead of the contracts field.
For more information on the workspace field, you can visit [here](https://docs.fuel.network/docs/forc/workspaces/#workspaces).
* Here we will be using Fuels CLI which wraps forc. `forc` commands like `forc build`, `forc deploy`, etc. can also be used directly for building and deploying contracts.
* Some other important Fuels CLI commands are 
  * `fuels build`: This command builds all Sway programs under your contracts using forc.
  * `fuels deploy`: This command deploys all Sway contracts and saves their deployed IDs to: `{output_in_config_file}/contract-ids.json`. It is recommended to use the `fuels deploy` command only when deploying contracts to a local node. If you are deploying contracts to a live network like the Testnet, it is recommended to use the `forc deploy` command instead.

Here is the [link](https://docs.fuel.network/docs/nightly/fuels-ts/fuels-cli/commands/) to the official documentation for the Fuels CLI commands.

# Output
TODO - Explain a bit about the output generated by the Fuels CLI here. The details are covered in the frontend section.
