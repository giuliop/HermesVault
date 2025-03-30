# The HERMES Token (Draft for Feedback)

**HERMES** is the native token of the **HermesVault** protocol.  
It is a network token fully owned by the protocol and its users (no pre-allocation, no tokens for the team, no tokens for investors).

The token has two core objectives:
1. **Bootstrap Initial Liquidity:** By distributing HERMES tokens to users during the early months, the protocol incentivizes early liquidity, thereby expanding the protocol’s anonymity set
2. **Full Protocol Ownership for Users:** All protocol fees (0.1% of withdrawals) will be continuously used to buy the HERMES token as described below. This ensures the protocol’s economic value is owned entirely by its users

## Tokenomics

HERMES has a fixed total supply of 100,000,000 tokens, allocated as follows:

### User Distribution (50%)

50,000,000 tokens will be distributed to users proportionally based on their provided liquidity during the first three months following MainNet launch:

- **Month 1:** 25,000,000 tokens
- **Month 2:** 15,000,000 tokens
- **Month 3:** 10,000,000 tokens

### Protocol-Owned Liquidity (50%)

50,000,000 tokens will remain under protocol ownership to create initial liquidity in a dedicated HERMES/ALGO liquidity pool:

- At MainNet launch, the protocol contract will make 25,000,000 tokens available for a fixed rate swap at 0.05 ALGO per HERMES token
- After completion of this swap, the contract will activate a liquidity pool using the collected 1,250,000 ALGO and the remaining 25,000,000 HERMES tokens

The liquidity pool will function as a traditional Automated Market Maker (AMM), facilitating token swaps between HERMES and ALGO. Additionally, all protocol fees collected by the protocol (i.e., 0.1% of withdrawals) will be continuously swapped for HERMES tokens via this pool, creating sustained buying pressure on HERMES.

## Token Distribution Mechanics

Distribution to users will be proportional to the liquidity provided to the protocol during each reference month.

There are two primary challenges in distribution:

1. **Privacy Preservation:** Blockchain data cannot measure user liquidity directly since withdrawals from the protocol are private
2. **Leakage Risk:** Claiming distributions directly from the addresses used for deposits would leak information that could be used to infer withdrawal timing and amounts
 
### Privacy-Preserving Distribution Approach

To address these challenges:

- To qualify for the distribution users will have to deposit through the frontend hosted at **hermesvault.org** (users depositing with other frontends will not be eligible)
- This frontend securely stores encrypted data linking deposits to withdrawals for compliance purposes
- Deposits will be associated with a unique hash derived from the user's secret note which will be stored on the smart contract
- To claim distributions privately, users will use the same secret note kept for withdrawals to send a zero-knowledge proof of their eligibility. This will allow users to safely claim tokens using addresses different from their original deposit addresses, ensuring better privacy

## Summary

By distributing half the supply to early liquidity providers and using the other half to seed a permanent AMM (with no pre-allocation for anyone), HERMES aims to:
- Incentivize growth of the protocol anonymity set
- Create a self-sustaining liquidity pool
- Continuously recycle protocol fees in the pool to reward token holders

This draft is open for community feedback. Please review and share any suggestions for improvement!