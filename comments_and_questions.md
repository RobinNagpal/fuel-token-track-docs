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
