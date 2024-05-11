# Token Tracker Example

This example demonstrates how to creat a contract that can mint, burn, and track the balance of a token for a user.

The full example is documented in four parts:

| Part | Details                                 | Link                                           |
|------|-----------------------------------------|------------------------------------------------|
| 1    | Introduction and Installation           | [LINK](/01_introduction_and_installation.md)   |
| 2    | Problem Statement and Project Structure | [LINK](/02_problem_statement_and_structure.md) |
| 3    | Contract Code                           | [LINK](/03_smart_contract.md)                  |
| 4    | Frontend Code                           | [LINK](/04_frontend.md)                        |

# Questions for Fuel
1. What should be the exact folder structure that we would recommend to developers? Currently, we can use the `workspace` feature from `forc` or add `contracts`, `predicates`, etc., separately.
2. Should `fuels.config.ts` be present in the parent folder? Below is the structure we currently have:
    ```
    .
    ├── README.md
    ├── contracts
    │   └── TokenTrack
    ├── frontend
    ├── fuels.config.ts
    └── package.json
    ```

# Pending work
- [ ] Expand the frontend section by adding more code and explanations for the three functions.
- [ ] Add information on funding the account, as this will be used to perform transactions.
- [ ] Cross-check the code three times by starting from scratch.
- [ ] Incentivize and ask a few developers to test it out and provide feedback.
