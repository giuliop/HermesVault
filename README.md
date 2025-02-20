# Hermes Vault
## Private transactions on Algorand

HermesVault enables private transactions on Algorand leveraging zero knowledge proofs.

We stand on the shoulders of giants, leveraging the pioneering work of [Zcash](https://z.cash/).
In fact, we aim to simplify Zcash constructions to achieve a user friendly balance between features and ease of use, compatible with an implementation at the smart contract level instead of at the protocol level.

In a nutshell, HermesVault let users deposit algo tokens (in any amount) in the application smart contract, providing them with a secret note. Later, with that secret note, users can withdraw all or part of their deposit to any address of their choice, including addresses with zero balance and no history with transaction fees paid directlty by the application.

HermesVault has been deployed to TestNet and you can help test it here: [hermesvault.org](https://hermesvault.org/)

HermesVault is fully open-source, read the frontend code at [HermesVault-frontend](https://github.com/giuliop/HermesVault-frontend) and the smartcontract code at [HermesVault-smartcontracts](https://github.com/giuliop/HermesVault-smartcontracts).

HermesVault's smart contracts are permissionless, so anybody can build frontends for HermesVault. Feel free to get in touch if you want to do that !


### Application overview

A user can make deposits of algo tokens in any amount to the application
contract, keeping a secret note. The originating address and the deposited amount are of course public on the blockchain.

Then, with the secret note, a user can withdraw part, or all, of the deposited tokens to any address. The address signing the withdrawal transaction, the
receiving address of the tokens (which might be the same as the signer), and the withdrawn amount will be public, but the source of the withdrawal, that is the original deposited amount and the original depositor address, will remain private.

Moreover, the withdrawal transaction can be signed by a smart signature provided
by the application, in which case only the receiving address and withdrawal amount
will be public.
This let the receiving address can be a zero balance account
(e.g., a new account with no history) since the smart signature will pay the
transaction fees from the withdrawal amount.

If only part of the tokens of the original deposit are withdrawn, the application
will create a new deposit with the "change" amount to be used for future
withdrawals with the same privacy guarantees, based on a new secret receipt held by the user.

### Decentralization path

HermesVault will be a fully decentralized project, 100% owned by its users.

Like [Prometheus](https://en.wikipedia.org/wiki/Prometheus)' gift of fire, this is a gift of privacy to the Algorand community, a public good.

As the project creator I will make no financial gain from this project.

Before MainNet launch, the plan for full decentralization and community ownership will be published.
