# eosio.pay2key

The pay2key contract is a standardized EOSIO smart contract, which allows token transacting using only public-private key pairs without the need for an EOSIO account or system action. It is similar in feature to the Ethereum transaction system. To move belonging to a public key, an owner only needs the corresponding private key to the public key. A transaction fee is also paid to move the token to the relayer. A "relayer" an be anyone with an EOS account. The relayer accepts the fee then relays the signed UTXO message to the contract which updates the balance of the appropriate public key. A relayer is similar to a miner in that they ensure the transaction ends up on chain.

There are 4 main actions in the contract: 

1. Creating the token specs when the contract is first deployed 
2. Sending tokens from EOS accounts to a public key
3. Transacting public key to public key
4. Withdrawing from a public key back into an EOS account  

All examples below will use **novusphereio** as the token contract for ATMOS which has a precision of 3, and **nsuidcntract** as the deployed pay2key contract.

## 1. Creating a token

Before you are able to issue a token to a public key, you must create an entry for the token by passing the token contract and the appropriate asset. Once the token has been created, it will have a "chain_id" field in the "stats" table which will be used when interacting with that token type.

```
cleos push action nsuidcntract create \'["novusphereio", "3,ATMOS"]\' -p {EOS_account}@active
```

## 2. Sending tokens from EOS accounts to a public key

Anyone can send the designated token to the UTXO contract, triggering an issuance to the public key of their choosing.
```
cleos push action novusphereio transfer '["{EOS_account}", "nsuidcntract", "100.000 ATMOS", "{PUBLIC_KEY_TO}"]' -p {EOS_account}@active
```
Now that public key will have 100 ATMOS to spend

## 3. Transacting public key to public key

To transfer from one public key to another, the transaction must be signed using your private key. Regarding nonce, every transaction must increase at least by nonce+1. Given this, it's safe to use time, for example `new Date().getTime()` as your nonce.

```
cleos push action nsuidcntract transfer '[{CHAIN_ID}, "{RELAYER_PUBLIC_KEY}", "{PUBLIC_KEY_FROM}", "{PUBLIC_KEY_TO}", "3.000 ATMOS", "1.000 ATMOS", {nonce}, "memo", "{SIGNATURE}"]' -p {EOS_account}@active
```

### Generating Signatures 
To create a signed transaction to spend UTXOs, a signature must be created by the private key of a public key with spendable outputs.
Please see the [Unified ID](https://github.com/Novusphere/docs/blob/master/formal/unified-id.md) document regarding signature format.

## 4. Withdrawing from a public key back into an EOS account

To withdraw tokens from an account, issue a transfer as normal, except make the receiving public key be the NULL address, EOS1111111111111111111111111111111114T1Anm, and the memo the receiving EOS account name.

The contract detects this, modifies the circulating supply, and triggers an inline action to send the tokens to the EOS account specified in the memo field.

```
cleos push action nsuidcntract transfer '[{CHAIN_ID}, "{RELAYER_PUBLIC_KEY}", "{PUBLIC_KEY_FROM}", "EOS1111111111111111111111111111111114T1Anm", "3.000 ATMOS", "1.000 ATMOS", {nonce}, "{EOS_account}", "{SIGNATURE}"]' -p {EOS_account}@active
```

## Notes

**Modified** and forked from [decentralbanknetwork/bank.contracts commit 64](https://github.com/decentralbanknetwork/bank.contracts/tree/master/bank.pay2key).
