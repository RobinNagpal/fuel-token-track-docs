# Token Track Project

One of the most common uses for smart contracts is creating a token. In this example, we'll make a contract that can 
mint, burn, and track a user's token balance. We'll also create a frontend to interact with the contract.

#### Here is how the UI will look like:

![User Interface](https://raw.githubusercontent.com/RobinNagpal/fuels-token-example/main/assets/images/ui.png)

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

mkdir contracts
mkdir frontend

cd contracts

```

Now let's create the `fuels.config.ts` file in the root directory.

```bash
npx fuels@0.84.0 init --contracts ./contracts/TokenTrack --output ./frontend/src/sway-api 
``` 

This will create a `fuels.config.ts` file with the following content:

```typescript
import { createConfig } from 'fuels';

export default createConfig({
  contracts: ['./contracts/TokenTrack'],
  output: './frontend/src/sway-api',
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
## Workspace

Note: We'll briefly discuss the concept of a workspace here, as it's not used in this example. For more details, you 
can refer to [this](https://docs.fuel.network/docs/forc/workspaces/#workspaces)

A workspace is a group of one or more packages, called workspace members, that are managed together.

Workspace manifests are declared within `Forc.toml` files and appear like this:
```toml
[workspace]
members = ["TokenTrack"]
``` 
We can use `forc` commands like `forc build`, `forc deploy`, etc. in the workspace directory to build and deploy all the
members in the workspace.

## Fuels CLI Commands

Below are the useful commands that can be used with the Fuels CLI:

#### 1. `fuels init`

The `fuels init` command helps you create a sample `fuel.config.ts` file.

#### 2. `fuels build`

The `fuels build` command basically serves two purposes:

- Building all Sway programs under your workspace using forc.
- Generating types for them using fuels-typegen.

```
npx fuels@0.84.0 build --deploy
```

Using the `--deploy` flag will additionally:

- Automatically start a short-lived fuel-core node if needed.
- Run deploy on that node.

### 3. `fuels deploy`

The `fuels deploy` command does two things:

1. Deploys all Sway contracts under the workspace.
2. Saves their deployed IDs to: `{output_in_config_file}/contract-ids.json`

```
{
  "myContract1": "0x..",
  "myContract2": "0x.."
}
```

Note: It is recommended to use the `fuels deploy` command only when deploying contracts to a local node. If you are deploying contracts to a live network like the Testnet, it is recommended to use the `forc deploy` command instead.

### 4. `fuels dev`

```
npx fuels@0.84.0 dev
```

The `fuels dev` command does three things:
1. Automatically starts a short-lived fuel-core node.
2. Runs build and deploy once at the start.
3. If autoStartFuelCore is enabled, it watches your Forc workspace and repeats the previous steps on every change.
