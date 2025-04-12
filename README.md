# Hermes Vault
## Private transactions on Algorand

HermesVault enables private transactions on Algorand leveraging zero knowledge proofs.

We stand on the shoulders of giants, leveraging the pioneering work of [Zcash](https://z.cash/).
In fact, we aim to simplify Zcash constructions to achieve a user friendly balance between features and ease of use, compatible with an implementation at the smart contract level instead of at the protocol level.

In a nutshell, HermesVault let users deposit algo tokens (in any amount) in the application smart contract, providing them with a secret note. Later, with that secret note, users can withdraw all or part of their deposit to any address of their choice, including addresses with zero balance and no history with transaction fees paid directlty by the application from the
original deposit.

HermesVault has been deployed to TestNet and you can help test it here using the reference frontend: [hermesvault.org](https://hermesvault.org/)

HermesVault is fully open-source, read the reference frontend code at [HermesVault-frontend](https://github.com/giuliop/HermesVault-frontend) and the smartcontract code at [HermesVault-smartcontracts](https://github.com/giuliop/HermesVault-smartcontracts).

HermesVault's smart contracts are permissionless, so anybody can build frontends for HermesVault.


### Protocol overview

A user can make deposits of algo tokens in any amount to the application contract, keeping a secret note. The originating address and the deposited amount are of course public on the blockchain.

Then, with the secret note, a user can withdraw part, or all, of the deposited tokens to any address. The address signing the withdrawal transaction, the
receiving address of the tokens (which might be the same as the signer), and the withdrawn amount will be public, but the source of the withdrawal, that is the original deposited amount and the original depositor address, will remain private.

Moreover, the withdrawal transaction can be signed by a smart signature provided by the application, in which case only the receiving address and withdrawal amount will be public.
This let the receiving address can be a zero balance account (e.g., a new account with no history) since the smart signature will pay the transaction fees from the withdrawal amount.

If only part of the tokens of the original deposit are withdrawn, the application will create a new deposit with the "change" amount to be used for future withdrawals with the same privacy guarantees, based on a new secret receipt held by the user.


## A public good

The HermesVault smart contract is a **public good** and charges no protocol fee.
It is an immutable contract, with no owner and no manager, fully permissionless.

To use the smart contract the only fees to be covered are the algorand blockchain fees, in particular:
* For withdrawals, 0.0153 algo to cover the storage minimum balance requirement increase borne by the smart contract
* To use HermesVault smart signature to pay the withdrawal transaction fees, 0.06 algo to fund those

Both fees will be taken from the original deposit, allowing a zero balance account to withdraw.

If the TSS is not used, 0.06 algo will be paid in trasanction fees by the signer.
For deposit, 0.056 algo will be paid in transaction fees by the signer.
Surprisingly, the bulk of these fees are needed to cover the computation cost of the new merkle root after each deposit/withdrawal.
The cost of the zk verification is only 0.008 algo.

Anybody can offer a fontend for HermesVault or compose and integrate it in new applications and workflows. Frontends and applications integrating HermesVault are free to charge their users what they like
