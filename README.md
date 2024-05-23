# Hermes Vault
## Private transactions on Algorand

This document is a work in progress whitepaper for a system to make private transactions on Algorand.

We stand on the shoulders of giants, leveraging the amazing work of [Zcash](https://z.cash/).  
In fact, we aim to simplify Zcash constructions to achieve a user friendly balance between features and ease of use, compatible with an implementation at the smart contract level instead of at the protocol level.

## Summary
- [Application overview](#application-overview)
- [Implementation overview](#implementation-overview)
- [Technical Details](#technical-details)
- [Decentralization path](#decentralization-path)
- [Compliance considerations](#compliance-considerations)
- [Conclusions](#conclusions)

### Application overview

A user can make deposits of algo tokens in any amount to the application
contract, keeping a secret receipt. The originating address and the deposited amount are of course public on the blockchain.

Then, with the secret receipt, a user can withdraw part, or all, of the deposited tokens to any address. The address signing the withdrawal transaction, the
receiving address of the tokens (which might be the same as the signer), and the withdrawn amount will be public, but the source of the withdrawal, that is the original deposited amount and the original depositor address, will remain private.

Moreover, the withdrawal transaction can be signed by a smart signature provided
by the application, in which case only the receiving address and withdrawal amount
will be public.  
In this scenario the receiving address can be a zero balance account
(e.g., a new account with no history) since the smart signature will pay the
transaction fees from the withdrawal amount.

If only part of the tokens of the original deposit are withdrawn, the application
will create a new deposit with the "change" amount to be used for future
withdrawals with the same privacy guarantees, based on a new secret receipt held by the user.

### Implementation overview
Let's start with a high level description.

We have two zk-circuits, one for deposits, one for withdrawals.  

When a user make a deposit, the application stores a deposit note which encrypts the amount deposited and a secret known by the user in a merkle tree.   
The deposit zk-circuit is used to prove that the encrypted note correctly stores the amount deposited.  

To withdraw, the user uses the withdrawal zk-circuit to prove that he knows the secret associated with a deposit in the tree, that the deposit has not been withdrawn from already, and that they are withdrawing no more than the deposited amount.

If so, the application lets the user withdraw and saves a new deposit in the merkle tree with the change (what is left from the original deposit); it also invalidates the original deposit by storing a "nullifier" that prevents using that deposit again in the future.

More specifically, the on-chain application is composed of:
* a main smart contract (SC) with the application logic
* a deposit verifier contract (DVC) to validate deposits' zk-proofs
* a withdrawal verifier contract (WVC) to validate withdrawals' zk-proofs
* a treasury smart signature (TSS) that can sign withdrawal transactions

Let's assume we have a zk-friendly hash function that takes an arbitrary number of inputs on the AVM, denoted with `hash`.
We'll discuss such a function later in the [technical details](#technical-details) section.

#### Deposits

The user:
1. Generates random integers `K`, `R`
2. Creates a deposit note for the `Amount` of tokens they intend to deposit as `Note = hash(Amount, K, R)`.
3. Computes `Commitment = hash(Note)`
4. Creates a zk-proof with the deposit zk-circuit:
    - Public inputs: `Amount`, `Commitment`
    - Secret inputs: `K`, `R`  

    The proof proves that `Commitment = hash(hash(Amount, K, R))`
5. Sends a transaction group to the blockchain with:
    - a payment of `Amount` tokens to the SC's address
    - an app call to SC's deposit method with the zk-proof
    
SC verifies the proof is valid by calling DVC, and that it received `Amount` tokens.  
If so, it inserts `Commitment` in an on-chain merkle tree.

#### Withdrawals

The user has a deposit `Note`, and its `Amount`, `K`, `R`.

The user:
1. Generates new random integers `K2`, `R2`
2. Creates a new deposit note as `Note2 = hash(Change, K2, R2)` where `Change` is the amount left for future withdrawals from the original deposit after this withdrawal is processed (calculation details below). 
3. Computes `Commitment2 = hash(Note2)`
4. Creates a zk-proof with the withdrawal zk-circuit:
    - Public inputs:
        - `Recipient` -> address for withdrawal
        - `Withdrawal` -> amount for withdrawal
        - `Fee` -> fee charged by the application
        - `Commitment2`
        - `Nullifier = hash(Amount, K)` explained below
        - `Root` of on-chain merkle tree
    - Secret inputs:
        - `Amount`
        - `K`
        - `R`
        - `Change`
        - `K2`
        - `R2`
        - `Index` -> leaf index of `Note` in merkle tree
        - `Path` -> merkle path for `Note`

    The merkle `Path` starts with `Note` (unhashed `Commitment`), then its sibling, then all the parent nodes' siblings up to, but excluding, the root.

    The proof proves that:
    - `Nullifier == hash(Amount, K)`
    - `Commitment2 == hash(hash(Change, K2, R2))`
    - `Path[0] == hash(Amount, K, R)`
    - `Path` is a valid merkle path for `Index` and `Root` 
    - `Change == Amount - Withdrawal - Fee`
    - `Change`, `Amount`, `Withdrawal`, `Fee` are all non-negative (necessary since the zk-circuit operates in modulo arithmetics)
5. Sends an app call to SC's withdrawal method with the zk-proof, which can be signed by the TSS

The SC verifies that:
* The proof is valid by calling WVC
* The `Nullifier` has not been previously used (the application maintains a list of used nullifiers)
* The `Root` is a valid root (for convenience the application can keep a list of the last few roots to support concurrent calls)
* The `Fee` is correct (e.g., 0.1% of `Withdrawal` with a 0.1 algo minimum fee, or whatever the application encodes)

If all is verified, SC will:
* add `Nullifier` to the list of used nullifiers, invalidating the deposit the withdrawal is coming from
* add a new deposit to the merkle tree by inserting `Commitment2` (note that `Change` might be zero, but we are still inserting it to not leak any information)
* send `Withdrawal` tokens to `Recipient`
* send `Fee` tokens to the TSS


### Technical Details

#### Merkle Tree
The commitments to the deposits (user deposits or "change" deposits) are stored in a merkle tree on-chain. For storage efficiency, we can store on-chain only the path from the last inserted leaf to the root; this is sufficient to compute the new root on new insertions.  
Assuming a 32 level tree which can support 2^32 leaves, that is over 4 billion deposits/changes, and a 32 byte tree node representation, the storage requirement will be 32 * 32 = 1024 bytes, excluding the root.
Assuming we maintain on-chain the last 50 roots for user convenience to support concurrent operations, we need an additional 50 * 32 = 1600 bytes

Note that frontends will need to read from the blockchain all the inserted leaves to compute merkle proofs and build withdrawal transactions.

#### Nullifiers

Nullifiers are needed to prevent a deposit to be withdrawn from twice. With a 32 level merkle tree, eventually up to over 4 billion nullifiers need to be stored on-chain (using boxes) and paid by the application; this is one reason why the application needs to charge a fee.

#### Roots

To avoid a situation where concurrent transactions submit a withdrawal zk-proof using the same recent tree root and only the first can succeed, the application maintains a list of e.g., the last 50 roots and checks withdrawal proofs against them.

#### Hash function

To manage the merkle tree on-chain we need a hash function on the AVM that matches a hash function we can use in the zk-circuits. The hash functions currently present on the AVM are not zk-friendly and cannot be implemented in circuits of practical sizes.

We need to bring to the AVM a new zk-friendly hash function, for instance as proposed by this [PR](https://github.com/algorand/go-algorand/pull/5978), to be able to deploy this application.

By using a forked version of the AVM that incorporates the PR above, we were able to deploy a working version of this application, so that's the last missing building block. In addition, one additional change to the AVM discussed below would also be very beneficial.

#### Verifiers

The DVC and WVC are generated by [AlgoPlonk](https://github.com/giuliop/AlgoPlonk) from the circuits' definitions; AlgoPlonk uses [gnark](https://github.com/Consensys/gnark) for circuit compilation and for proof generation and can be used by frontends for the latter.

AlgoPlonk offers a [trusted setup](https://github.com/giuliop/AlgoPlonk#trusted-setup) for both curves BN254 and BLS12-381 but only the BN254 setup is large enough to support the withdrawal circuit. Until a new trusted setup ceremony is run for curve BLS12-381 in the future, only the BN254 can be used for this application.

At the moment the verifiers need to be smart contracts since as smart signatures they exceed the maximum allowed smart signature length of 1 KB. Given that a BN254 verifier consumes ~150,000 opcode budget and that updating a 32 level merkle tree with the hash function defined in the [PR](https://github.com/algorand/go-algorand/pull/5978) above costs ~45,000 more, we end up exceeding the maximum pooled opcode budget.  
For this reason, to implement a working application with the forked AVM we had to split both deposits and withdrawals in two subsequent transactions, one to verify the zk-proof and store a success receipt, and a following one to then insert the deposit/change in the merkle tree.  
It works, at the cost of a less smooth user experience, but ideally we can make the verifiers smart signatures accessing their extra opcode budget pool and do everything in one transaction.  
This will be possible if [this](https://github.com/algorand/go-algorand/issues/5990) is implemented.

### Decentralization path

An application like this ought to be eventually fully decentralized and outside the control of its creators.

A possible decentralization path in stages could be the following:

1. The application is deployed and the creators retain ability to update the application to ensure it works well. This phase should have a set length, e.g., 6 months, or maybe a target TVL to reach

2. The application is made immutable, the creators still can manage the treasury

3. Control of the treasury is ceded to the community

This last step could be accomplished in different ways; here are some preliminary ideas, not an exhaustive list:
* A token is created (e.g., fairly distributed to the application users) and the token owners can vote on how to use the treasury
* Like the above, but instead of voting, the treasury pays itself to the token owners
* The treasury is redirected to the fee sink

### Compliance considerations
Frontends giving access to the application, being centralized entities, will need a compliance strategy to comply with the laws and regulations of the jurisdictions they operate in.

A strategy to be able to selectively disclose the link between a withdrawal and its originating deposit while giving guarantee to the users that the frontend cannot access their funds is the following: the frontend stores `Amount` and secret `K`, but not secret `R`, for deposits, and `Change` and secret `K2`, but not secret `R2`, for withdrawals.

This allows the frontend to reveal the link between withdrawals and deposits if compelled to. At the same time, not having access to secret `R` (or `R2`) the frontend cannot touch users' funds.

### Conclusions

Private transactions on Algorand can be a useful building blocks for many other applications. This working draft describes a possible system to make this work, and a working application implementing it already exists (albeit leveraging a [PR](https://github.com/algorand/go-algorand/pull/5978) not yet included in the AVM).

Let us have any comment and feedback to improve this as we build it!