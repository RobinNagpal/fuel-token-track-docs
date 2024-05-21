# Questions

1. What should be the recommended project structure for a Fuels project?
2. Do we use workspace for every Fuels project?
3. How should the frontend be structured in a Fuels project?
4. what is json rpc? its use?
5. why are we even using fuel
6. why sway instead of solidity
7. what are we trying to achieve with fuel
8. is it a l2 chain or a rollup
9. what else can we achieve other than the current functions
10. can we achieve the same thing we are doing using foundry
11. use of forc-fmt and forc-lsp?
12. can we use some personal wallet other than fuel ones?
13. what's the flow now? generate contract, build, deploy and then?
14. hardcoded contract id, gas price, gas limit?
15. how to use another contract in frontend
16. how to reconnect?
17. can we use fuel to do compound stuff?
18. what if i want to change the implementation of the contract? do i have to deploy a new one?
19. fuel is simulation or real deployment can also be done?
20. can we retrofit? like if i have a contract already deployed on ethereum, can i use it on fuel?

# Issues

1. there should be usage examples as well for the frontend cause i dont know what address or contract ID to use
2. indexing thing deprecated?
3. getting "Contract not loaded" on any input (wallet is null)
4. where are the minted tokens going? how to check them?
5. add localhost address in wallet if running local node
6. adding test eth to wallet (setting up wallet and using beta-faucet to credit eth)
7. local deployment not working (only testnet for now)
8. no need to select whether address or contractID, works either way for minting
9. empty form submission gives console error
10. mint/burn/transfer completion toast
11. showing balance field is not showing value in decimals so currently showing '0'
12. burn token doesnt take address (but using contractId behind the scene) so not clear whether burning from pool or a contract
13. tokens are getting transfered to a fix account instead of the address i am passing
14. when burn amount is more than the tokens in pool, fuel wallet crashes
15. total number of minted tokens is not shown
16. dropdown bug

# Explanation Improvement

1. flow of executing the commands is not clear (like which directory), some deployment commands are to be run in contract itself and some in frontend
2. if deployed on testnet or local node (using `forc-core run`), contractID wont come from contract-ids.json file while for deployment using `npx fuels@0.84.0 dev` or `npx fuels@0.84.0 build --deploy`, contract ID comes from contract-ids.json file
3. connect with same wallet account which was used to create the wallet with `fuelup`
4. on windows, project has to be cloned and run in wsl directory
5. i guess we should create fuel config inside the frontend rather than root
6. there should be a quick setup section for running the project after the first time
7. i guess we should separate out information from the project setup steps as it breaks the flow
8. currently there are only chunks of code given for the contract, there should be whole contract example as well
9. fuel config explanation section should be in frontend and not in 02-problem
10. for the useEffect section in 04-frontend, file is not specified

# Issues with following the docs

1. under creating a new project heading (in 02), `fuel.config.ts` is needed in frontend this is useless here `mkdir frontend` as its part of frontend setup, also this command requires the frontend setup first so to make sense of it `npx fuels@0.84.0 init --contracts ./contracts/TokenTrack/ --output ./frontend/src/sway-contracts-api`
2. under heading Contract Code - Implementation of ABI (in 03), functions are shown but its not written that these functions needs to be inside `impl MyContract for Contract {}`
3. this import `use std::{hash::Hash,};` is missing from docs
4. command for creating the contract doesnt align with the directory change command later i.e., `forc new TokenTrack` & `cd token-track`
5. not all functions' implementations are given so getting an error on building cause all the functions in the ABI are not implemented
6. enum defined `Identity` is redundant
7. `forc deploy` command doesnt took me through the process of wallet setup as written under Deploying on testnet (in 03) and gives error, so better to use `forc deploy --testnet` which will ask for wallet creation.
8. `chainConfig.json` file setup is unclear
9. we should write that these commands need to be on separate terminals `fuel-core run --db-type in-memory --debug` & `forc deploy --unsigned --node-url 127.0.0.1:4000/graphql`
10. how to check wallet address? which i setup using `forc deploy --testnet`. confused between terminal wallet and fuel wallet
11. folder name mismatch `sway-api` or `sway-contract-api` in 02 & 04
12. this code is written in docs but its not present in the code, `The following code defines a hook useActiveWallet which manages the active wallet within your Fuel Dapp:`
13. under Functionality heading (in 04), its not written which file to write code in
14. the frontend code is more like explanation of the written code and not telling how to write
15. some code is too naive like error handling heading, resetting values
16. txParams error as gasPrice is not present in the type

success dish moment daughter marine comfort fork use wild mention satisfy bread vacuum credit duck body diamond lonely demise capable carbon list able quality
