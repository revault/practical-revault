# Messages

A description of how information is exchanged between the different entities, as well as the
flow of operations.

These messages are exchanged on top of an [encrypted and authenticated communication
channel](network.md).


- [Watchtower](#watchtower)
- [Cosigning server](#cosigning-server)
- [Coordinator](#coordinator)

## Watchtower

At least one watchtower is ran by every stakeholder, and at most one wallet is paired with
each watchtower. A watchtower needs to be able to sign any revocation transaction before
its corresponding wallet signs the unvaulting transaction.

In addition, a watchtower will by default revault any unvaulting attempt. We need a way
for the watchtowers to poll the spending transaction (as its revaulting policy may depend
on it). For this matter we use the coordinator as a proxy between the managers
and the watchtowers.
When noticing an unvaulting attempt, the watchtower will poll the coordinator
for a potential spend transaction. If there is one and it is valid according to its
revaulting policy, it will not do anything. Otherwise it will broadcast the cancel
transaction.


### Rough flow

```
 STAKEHOLDER's WALLET                      WATCHTOWER
    ||   -- sig emer ------------>   ||  // Here are all sigs for the emergency transaction.
    ||  <--- sig_ack  ---------      ||  // I succesfully re-constructed, checked, and stored this transaction.
    ||   -- sig emer_unvault ---->   ||
    ||  <--- sig_ack  ---------      ||
    ||   -- sig cancel   -------->   ||
    ||  <--- sig_ack  ---------      ||
```

### Messages format

#### `sig`

Sent at any point in time by a stakeholder's wallet to share all signatures for a revocation
transaction with its watchtower. The wallet must wait for the tower's `sig_ack` on all
revocation transactions before sharing its signature for the unvault transaction with the other
participants.

```json
{
    "method": "sig",
    "params": {
        "signatures": {
            "pubkeyA": "ALL|ANYONECANPAY Bitcoin ECDSA signature as hex",
            "pubkeyB": "ALL|ANYONECANPAY Bitcoin ECDSA signature as hex",
            ...
        },
        "txid": "txid",
        "deposit_outpoint": "deposit utxo outpoint"
    }
}
```


#### `sig_ack`

The watchtower must not send an ACK if it did not successfully reconstruct and check
the transaction, *or if it is unable to bump its feerate with its currently-available utxos*.

```json
{
    "result": {
        "ack": true,
        "txid": "txid"
    }
}
```


------


## Cosigning server

A cosigning server is ran by each stakeholder. It is happy to sign any transaction input
it can, but only once.

### Rough flow

```
  WALLET                      COSIG_SERVER
    ||   -A-- sign  -------->    ||   // A: I need you to sign this transaction input
    ||   <-- sign result ----    ||   // Server: *signs* .. Here you go.
    ||
    ||   -B-- sign  -------->    ||   // B: I need you to sign this same transaction input
    ||   <-- sign result ----    ||   // Server: I already signed an input spending this outpoint, here is the existing signature
```

### Messages format

#### `sign`

Sent at any point in time by a manager who'll soon attempt to unvault and spend one or
more vault(s).

As the spend transaction must only spend unvault transaction outputs, the cosigning server
shall sign all inputs (if all of them were not already signed).

```json
{
    "method": "sign",
    "params": {
        "tx": "psbt"
    }
}
```

The server must:
  - if any input was already signed
    - return the empty string `""`
  - else
    - return the PSBT with its signature appended for all inputs

```json
{
    "result": {
        "tx": "psbt"
    }
}
```


------


## Coordinator

The coordinator is a proxy for watchtowers to retrieve spend transactions, and conveniently
permits to share pre-signed transaction signatures without interaction between participants.

As each wallet will verify and store signatures locally, the server isn't trusted and can be
managed by the organisation deploying Revault itself or any third party without risking any
loss of funds.

Acting as a cache in place of -example given- a p2p network, the information stored on the
coordinator is transient.

All [revaulting transactions][revaulting_txs] (the cancel tx and both emergency txs) are signed
paying a fixed `22 sat/WU` feerate and using the `ALL | ANYONECANPAY` signature hash flag. This
is in order to reduce the funds burden on *each* of the watchtowers.

The [unvault transaction][unvault_tx] is signed using a fixed `6 sat/WU` feerate. This is
a completely arbitrary value that was chosen to avoid blocking operations too early in case of
a huge load of transactions on the network and an increase of the mempools minimum feerate.  
This transaction's fees can be bumped if not competitive (using the CPFP output) but
it will likely not be relayed if the mempools minimum feerate goes above `84 000 sat/kw`
until the Bitcoin network deploys [package relay][package_relay].


### Rough flow

```
 STAKEHOLDER's WALLET          COORDINATOR
    ||   -A-- sig   -------->    ||   // A: Here is a sig for this txid !
    ||   -C-- sig   -------->    ||
    ||   -B-- sig   -------->    ||
    ||
    ||   -C-- get_sigs  ---->    ||   // C: I gave my sig but now am waiting for everyone to complete..
            ...(polling)
    ||   -A-- get_sigs  ---->    ||
            ...(polling)
    ||   -B-- get_sigs  ---->    ||
            ...(polling)
            ...(polling)              // Eventually they all retrieve the sigs.
```

```

 MANAGER's WALLET                COORDINATOR
    ||   -- set_spend_tx  ------->   ||  // I'm going to unvault these vaults, here is the
                                            fully signed spend transaction so watchtowers
                                            don't freak out.
```

```
  WATCHTOWER                      COORDINATOR
    ||   -- get_spend_tx  ------->   ||  // Is there any available spend tx for this vault ?
    ||  <--- spend_tx   ----------   ||  // Here is what i know about. I may be lying but worst case you revault.
```


### Messages format

#### `sig`

Sent by a stakeholder wallet at any point in time to share the signature for a transaction
with all other stakeholders and all managers.  
Note that the managers are able to poll the Emergency transactions signatures from the
coordinator. However, since they are not able to reconstruct the transactions (they don't
know the `scriptPubKey`) they can't use them to DOS the operations by broadcasting them
to the Bitcoin network.

The wallet can safely post its signature for the `cancel` and `emergency`s of each
`vault` without waiting for others. However, it must wait for everyone to have signed
the `cancel` and `emergency` transactions and its watchtower to have verified and stored
the signature before possibly sharing its signature for the unvault transaction.

A wallet is not bound to share its signature for the unvault transaction. This flexibility
allows "unactive vaults": a multisig which is not spendable by default but still guarded
by the emergency transaction deterrent.  
A wallet must share its signature for the `cancel` and the unvault `emergency`
transactions nonetheless.  
An inactive vault may later become active by sharing signatures for the `unvault`
transaction.  

Revocation transactions (`cancel` and `emergency`s) are signed with the `ALL|ANYONECANPAY`
flag.

```json
{
    "method": "sig",
    "params": {
        "pubkey": "Secp256k1 public key used to sign the transaction (hex)",
        "signature": "Bitcoin ECDSA signature (without sighash type byte) as hex",
        "txid": "transaction txid"
    }
}
```


No explicit ACK from the server as the wallet can just `get_sigs` for its own signature.


#### `get_sigs`

Sent by a wallet to retrieve all signatures for a specific transaction.

```json
{
    "method": "get_sigs",
    "params": {
        "txid": "transaction txid"
    }
}
```

```json
{
    "result": {
        "signatures": {
            "pubkeyA": {
                "signature": "Bitcoin ECDSA signature"
            },
            "pubkeyC": {
                "signature": "Bitcoin ECDSA signature"
            }
        }
    }
}
```

Note the absence of `pubkeyB` in the above samples.

If a wallet notices its transaction to be absent, it must send it again. It can either
mean the server didn't store it (no explicit ACK) or someone was unhappy with it (no
explicit error from the server).


#### `set_spend_tx`

Sent by a manager to advertise the Spend transaction that will eventually be used for a
set of Unvault.

```json
{
    "method": "set_spend_tx",
    "params": {
        "deposit_outpoint": ["txid:vout", "txid:vout"],
        "spend_tx": "hex of Bitcoin-serialized fully-signed spend transaction"
    }
}
```


#### `get_spend_tx`

Sent by a watchtower to the coordinator after an unvault event to learn
about the spend transaction.

```json
{
    "method": "get_spend_tx",
    "params": {
        "deposit_outpoint": "txid:vout"
    }
}
```


#### `spend_tx`

The response to a `get_spend_tx`.

```json
{
    "result": {
        "spend_tx": "hex of Bitcoin-serialized spend tx"
    }
}
```


## Processes

### Presigning process

#### Revocation transactions signing step

```
stakeholder           coordinator
   +                      +
   | +-sig emer---------> |
   | +-sig emer_unvault-> |
   | +-sig cancel-------> |
   |                      |
   | +--get_sigs--------> |
   |                      +
   |
   |                   watchtower
   |                       +
   | +--sig emer---------> |
   | <--------sig_ack----+ |
   |                       |
   | +--sig emer_unvault-> |
   | <--------sig_ack----+ |
   |                       |
   | +--sig cancel-------> |
   | <-------sig_ack-----+ |
   +                       +
```

| flow                       | request               |
| -------------------------- | --------------------- |
| stakeholder -> coordinator | [sig](#sig-1)         |
| stakeholder -> coordinator | [get_sigs](#get_sigs) |
| stakeholder -> watchtower  | [sig](#sig)           |
| watchtower -> stakeholder  | [sig_ack](#sig_ack)   |

#### Activation signing step

```
stakeholder      coordinator
   +                 +
   | +-sig unvault-> |
   |                 |
   | +--get_sigs---> |
   +                 |
                     | 
manager              |
   +                 |
   | +---get_sigs--> |
   +                 +
```

| flow                       | request               |
| -------------------------- | --------------------- |
| stakeholder -> coordinator | [sig](#sig-1)         |
| stakeholder -> coordinator | [get_sigs](#get_sigs) |

### Spending process

```
     ... Gather sufficient managers signatures ...

manager         cosig_server
   |                 +
   |                 |
   | +----sign-----> |
   | <-sign result-+ |
   |                 +
   |
   |                       coordinator
   |                           +
   |                           |
   | +-set_spend_tx ---------> |
   |                           |
   +                           +

      ... Broadcast the unvault tx ...

                           coordinator              watchtower
                               +                         +
                               |                         |
                               | <-- get_spend_tx ------ |
                               | --------- spend_tx ---> |
                               +                         +
```

| flow                      | request                             |
| ------------------------- | ----------------------------------- |
| manager -> cosig_server   | [sign](#sign)                       |
| manager -> coordinator    | [set_spend_tx](#set_spend_tx)       |
| watchtower -> coordinator | [get_spend_tx](#get_spend_tx)       |



[revaulting_txs]: transactions.md#cancel_tx
[unvault_tx]: transactions.md#unvault_tx
[package_relay]: https://github.com/bitcoin/bitcoin/issues/14895
