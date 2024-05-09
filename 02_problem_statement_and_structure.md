# TokenTrack Project
One of the most common use cases for smart contracts is to create a token. In this example, we will create a contract 
that can mint, burn, and track the balance of a token for a user. We will also create a frontend that will allow us 
to interact with the contract.

#### Here is how the UI will look like:

TODO: Add a screenshot of the UI

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

### fuels.config.ts
TODO: write details about the fuels toolchain and the four commands that we will use in this project.


fuels toolchain - https://docs.fuel.network/docs/nightly/fuels-ts/fuels-cli/





