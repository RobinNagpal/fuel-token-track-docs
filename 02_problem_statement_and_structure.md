# Token Track Project

One of the most common use cases for smart contracts is to create a token. In this example, we will create a contract
that can mint, burn, and track the balance of a token for a user. We will also create a frontend that will allow us
to interact with the contract.

#### Here is how the UI will look like:

![User Interface](https://raw.githubusercontent.com/RobinNagpal/fuels-token-example/main/assets/images/ui.png)

The UI allows to:

- Mint tokens
- Burn tokens
- Transfer tokens
- Check the balance of a user or a wallet

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

- `contracts`: Contains the smart contract code. Details will be discussed in the 'Smart Contract' section.
- `frontend`: Contains the frontend code. Details on the frontend code will be discussed in the 'Frontend' section.
- `fuels.config.ts`: Contains the configuration for the Fuels Toolchain.

# Fuels CLI
For this project, we will be using the Fuels CLI to build and deploy our contracts. The Fuels CLI is a command-line 
tool that helps you to build, deploy, and interact with your smart contracts. It is a wrapper around the `forc` 
command-line tool. The Fuels CLI can be configured using a `fuels.config.ts` file.

### fuels.config.ts
```typescript
import { createConfig } from 'fuels';

export default createConfig({
  contracts: ['contracts/TokenTrack'],
  output: './frontend/src/sway-api',
});
```

In our case, we have specified the `contracts` and `output` fields. 
- `contracts` field specifies the path to the  contracts directories
- `output` field specifies the path where the generated files will be saved.

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

# Workspace
A workspace is a collection of one or more packages, namely workspace members, that are managed together.

Workspace manifests are declared within `Forc.toml` files and it looks like this:

```toml
[workspace]
members = ["TokenTrack"]
``` 
We can use `forc` commands like `forc build`, `forc deploy`, etc. in the workspace directory to build and deploy all the
members in the workspace.

# Fuels CLI Commands
consists of a couple commands, we'll discuss four of the most used ones:

### 1. `fuels init`

The `fuels init` command helps you to create a sample `fuel.config.ts` file.

Assuming, we are at `./frontend`, run this command to generate `fuel.config.ts` file:

```
npx fuels@0.84.0 init --contracts ../contracts/TokenTrack/ --output ./src/sway-contracts-api
```

This will give you a minimal configuration:

```
import { createConfig } from 'fuels';

export default createConfig({
  contracts: [
        '../contracts/TokenTrack',
  ],
  output: './src/sway-contracts-api',
});
```

This config file contains two key elements:

1. `contracts`: List of relative directory paths to Sway contracts.
2. `output`: Relative directory path to use when generating Typescript definitions.

For other properties in this file you can visit [here](https://docs.fuel.network/docs/nightly/fuels-ts/fuels-cli/config-file/)

### 2. `fuels build`

The `fuels build` command basically has two purposes:

- Build all Sway programs under your workspace using forc.
- Generate types for them using fuels-typegen

```
npx fuels@0.84.0 build --deploy
```

Using the --deploy flag will additionally:

- Auto-start a short-lived fuel-core node if needed
- Run deploy on that node

### 3. `fuels deploy`

    The fuels deploy command does two things:

        1. Deploy all Sway contracts under workspace.
        2. Saves their deployed IDs to:
            ./src/sway-programs-api/contract-ids.json

```
{
  "myContract1": "0x..",
  "myContract2": "0x.."
}
```

Use it when instantiating your contracts:

```
import { SampleAbi__factory } from './sway-programs-api';
import contractsIds from './sway-programs-api/contract-ids.json';

/**
  * Get IDs using:
  *   contractsIds.<my-contract-name>
  */

const wallet = new Wallet.fromPrivateKey(process.env.PRIVATE_KEY);
const contract = SampleAbi__factory.connect(contractsIds.sample, wallet);

const { value } = await contract.functions.return_input(1337).dryRun();

expect(value.toHex()).toEqual(toHex(1337));
```

Note: It is recommended using the `fuels deploy` command only when you are deploying contracts to a local node. If you are deploying contracts to a live network like the Testnet, it is recommended using the `forc deploy` command instead.

### 4. `fuels dev`

```
npx fuels@0.84.0 dev
```

The fuels dev command does three things:

    1. Auto-start a short-lived fuel-core node (docs)
    2. Runs build and deploy once at the start
    3. Watches your Forc workspace and repeats previous step on every change

You can learn more about the fuels CLI [here](https://fuellabs.github.io/fuels-ts/guide/fuels-cli/)
