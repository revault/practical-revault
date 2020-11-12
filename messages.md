# Messages

A description of how information is exchanged between the different entities, as well as the
flow of operations.

These messages are exchanged on top of an encrypted and authenticated communication
channel.


- [Watchtower](#watchtower)
- [Cosigning server](#cosigning-server)
- [Synchronisation server](#synchronisation-server)



## Watchtower

At least one watchtower is ran by every stakeholder, and at most one wallet is paired with
each watchtower. A watchtower needs to be able to sign any revocation transaction before
its corresponding wallet signs the unvaulting transaction.

In addition, a watchtower will by default revault any unvaulting attempt. We need a way
for a manager to signal its willingness to spend a vault, and for the
watchtower to ACK or NACK it (cheaper and less onchain footprint than try-and-be-canceled).  
For this matter we use the synchronisation server as a proxy between the managers and the
watchtowers.


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

```
  WATCHTOWER                      SYNC_SERVER
    ||   -- get_spend_requests -->   ||  // Is anyone currently willing to spend a vault ?
    ||  <--- spend_requests  -----   ||  // Nope.

    ||   -- get_spend_requests -->   ||  // Is anyone currently willing to spend a vault ?
    ||  <--- spend_requests  -----   ||  // Yep.
    ||   -- spend_opinion  ------>   ||  // The policy I enforce agrees / disagrees with this spend attempt.
    ||   -- get_finalized_spends ->  ||  // Did they share the fully signed PSBT yet ?
    ||  <--- finalized_spends ----   ||  // Nope.
    ||   -- get_finalized_spends ->  ||  // What about now ?
    ||  <--- finalized_spends ----   ||  // Yep here is what they claim it to be.
    ||  --- validate_spend  ----->   ||  // Fine. Go ahead managers!
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
        "vault_txid": "vault transaction txid"
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


#### `get_spend_requests`

Regularly sent by a watchtower to the synchronisation server to learn about spending
attempts.

```json
{
    "method": "get_spend_requests",
    "params": {}
}
```


#### `spend_requests`

The response to a `get_spend_requests`. Returns an arbitrarily-sized (can be 0)
array of objects detailing a spending request.

```json
{
    "result": {
        "requests": [
            {
                "timestamp": 0000000,
                "spend_tx": "unsigned spend tx (PSBT format)"
            }
        ]
    }
}
```


#### `spend_opinion`

Sent by a watchtower to the synchronisation server to signal its acceptance or refusal of
a specific spend. The `reason` field must be set if accept is `false`, otherwise it's
ignored by the sync server.

Note that there might be multiple spend attempts in flight: we support spend transaction
batching.

We require a signature for this message, as its relayed by the synchronisation server to
the wallet.

```json
{
    "method": "spend_opinion",
    "params": {
        "id": "spend transaction txid",
        "accept": true,
        "reason": "",
        "sig": "ECDSA (secp256k1) signature of this utf-8 encoded json with no space and 'sig:\"\"'"
    }
}
```


#### `get_finalized_spends`

Regularly sent by a watchtower to the synchronisation server after having received
a `spend_request` to learn about finalized spending attempts.

```json
{
    "method": "get_finalized_spends",
    "params": {}
}
```


#### `finalized_spends`

The response to a `get_final_spend_requests`. Returns an arbitrarily-sized (can be 0)
array of objects detailing a finalized (ie with a fully-signed spend transaction)
spending request.

```json
{
    "result": {
        "requests": [
            {
                "transaction": "fully signed spend transaction (PSBT format)"
            }
        ]
    }
}
```


#### `validate_spend`

Sent by a watchtower to the synchronisation server to signal its acknowledgement
or refusal of a fully-signed spend transaction.

It has the same entries as the `spend_opinion` message, and should under normal
circumstances (managers didn't try to cheat) have the same content semantic as well.

```json
{
    "method": "spend_validation",
    "params": {
        "id": "spend transaction txid",
        "accept": true,
        "reason": "",
        "sig": "ECDSA (secp256k1) signature of this utf-8 encoded json with no space and 'sig:\"\"'"
    }
}
```




------



## Synchronisation server

The sync server allows stakeholders' wallets to exchange signatures without the need for them
to be interconnected, and managers' wallets to poll watchtowers without direct connections
to them.

As each wallet will verify and store signatures locally, the server isn't trusted and can be
managed by the organisation deploying Revault itself or any third party without risking any
loss of funds.

Acting as a cache in place of -example given- a p2p network, the information stored on the
synchronisation server is transient. It may be gradually deleted, for example with a
lifetime of 2 weeks which we think is a good tradeoff between a smoother user experience
(up to 2 weeks of gap between the first and last signatory for a vault) on one side, and
operation cost *as well as reliance on it* on the other side.

All [revaulting transactions][revaulting_txs] (the cancel tx and both emergency tx) are signed
paying a fixed `253 sat/kw` feerate and using the `ALL | ANYONECANPAY` signature hash flag.

The [unvault transaction][unvault_tx] is signed using a fixed `84 000 sat/kw` feerate. This is
a completely arbitrary value that was chosen to avoid blocking operations too early in case of
a huge load of transactions on the network and an increase of the mempools minimum feerate.  
This transaction's fees can be bumped if not competitive (using the CPFP output) but
it will likely not be relayed if the mempools minimum feerate goes above `84 000 sat/kw`
until the Bitcoin network deploys [package relay][package_relay].


### Rough flow

```
 STAKEHOLDER's WALLET          SYNC_SERVER
    ||   -A-- sig   -------->    ||   // A: Here is a sig for this id !
    ||   -C-- sig   -------->    ||
    ||   -B-- sig   -------->    ||
    ||
    ||   -C-- get_sigs  ---->    ||   // C: I gave my sig but now am waiting for everyone to complete..
            ...(polling)
    ||   -A-- get_sigs  ---->    ||
            ...(polling)
    ||   -B-- get_sigs  ---->    ||
            ...(polling)
    ||   -A-- err_sig   ---->    ||   // A: Huh lol. This sig isn't valid.
    ||   <-- get_sigs error--    ||   // Server: Someone's is unhappy with the sig so I erased all of them, try again.
    ||
    ||   -A-- sig  --------->    ||
    ||   -B-- sig  --------->    ||
    ||   -C-- sig  --------->    ||
    ||
    ||   -A-- get_sigs  ---->    ||
    ||   -B-- get_sigs  ---->    ||
    ||   -C-- get_sigs  ---->    ||
            ...(polling)
            ...(polling)              // Eventually they all retrieve the sigs.
```

```

 MANAGER's WALLET                SYNC_SERVER
    ||   -- request_spend    ---->   ||  // I'd like to spend this vault.
    ||   -- get_spend_opinions -->   ||  // What do watchtowers say about this spend ?
    ||  <--- spend_opinions  -----   ||
                (....)
    ||   -- get_spend_opinions -->   ||
    ||  <--- spend_opinions  -----   ||  // Eventually all watchtowers responded.
              (..sig..)
    ||   -- finalize_spend  ----->   ||  // Ok we and the cosigners signed, here is the tx
    ||   - get_spend_validations ->  ||  // Do watchtowers let me be now ?
    ||   <-- spend_validations ---   ||  // Yep, go ahead.
```

### Messages format

#### `sig`

Sent by a stakeholder wallet at any point in time to share the signature for a transaction
with all participants.

The wallet can safely post its signature for the `cancel` and `emergency`s of each
`vault` utxo without waiting for others. However, it must wait for everyone to have signed
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
        "signature": "Bitcoin ECDSA signature as hex",
        "id": "transaction txid"
    }
}
```

No explicit ACK from the server as the wallet can just `get_sigs` for its own signature.


#### `get_sigs`

Sent by a wallet to retrieve all signatures for a specific transaction.
FIXME: managers' wallets can currently get all signatures !!

```json
{
    "method": "get_sigs",
    "params": {
        "id": "transaction txid"
    }
}
```

The server answers with a (possibly incomplete) mapping of each pubkey to each signature
required for this transaction.

```json
{
    "result": {
        "signatures": {
            "pukeyA": "sig",
            "pubkeyC": "sig"
        }
    }
}
```
Note the absence of `pubkeyB` above.

If a wallet notices its transaction to be absent, it must send it again. It can either
mean the server didn't store it (no explicit ACK) or someone was unhappy with it (no
explicit error from the server).


#### `err_sig`

FIXME: should we crash instead of handling this (potentially adversarial) scenario ?

Sent by a wallet to express its dreadful unhapiness with one of the returned signatures.
It results in the server erasing all the signatures for this transaction and make other
wallets send a new signature.

This should not happen, but hey.

```json
{
    "method": "err_sig",
    "params": {
        "id": "transaction txid"
    }
}
```


#### `request_spend`

Sent by a manager to signal their willingness to spend a vault.

We use a timestamp as watchtowers might accept the same spending attempt in the future.

```json
{
    "method": "request_spend",
    "params": {
        "timestamp": 0000000,
        "spend_tx": "unsigned spend tx (PSBT format)"
    }
}
```


#### `get_spend_opinions`

Sent by a manager when polling for watchtowers agreement regarding the spend
attempt identified by `vault_id`.

```json
{
    "method": "get_spend_opinions",
    "params": {
        "id": "spend transaction txid"
    }
}
```


#### `spend_opinions`

The response to `get_spend_opinions`, an arbitrarily-sized (can be 0 if no watchtower
responded) array of the response of each watchtower.

```json
{
    "result": {
        "id": "spend transaction txid",
        "opinions": [
            {
                "accepted": true,
                "reason": "",
                "sig": "ECDSA (secp256k1) signature of this exact json with no space and 'sig:\"\"'"
            },
            {
                "accepted": true,
                "reason": "",
                "sig": "ECDSA (secp256k1) signature of this exact json with no space and 'sig:\"\"'"
            },
            {
                "accepted": true,
                "reason": "",
                "sig": "ECDSA (secp256k1) signature of this exact json with no space and 'sig:\"\"'"
            }
        ]
    }
}
```

The wallet needs to insert the `id` field in each opinion object in order to be able
to validate the signature.



#### `finalize_spend`

Sent by a manager to finalize their spending attempt by presenting the fully-signed spend
transaction to the watchtowers.

```json
{
    "method": "finalize_spend",
    "params": {
        "transaction": "fully signed spend transaction (PSBT format)"
    }
}
```


#### `get_spend_validations`

Sent by a manager when polling for watchtowers acknowledgement of the fully signed
transaction spending the vault identified by `vault_id`.

```json
{
    "method": "get_spend_validations",
    "params": {
        "id": "spend transaction txid"
    }
}
```


#### `spend_validations`

The response to `get_spend_validations`, an arbitrarily-sized (can be 0 if no watchtower
responded) array of the response of each watchtower.

```json
{
    "result": {
        "id": "spend transaction txid",
        "opinions": [
            {
                "accepted": true,
                "reason": "",
                "sig": "ECDSA (secp256k1) signature of this exact json with no space and 'sig:\"\"'"
            },
            {
                "accepted": true,
                "reason": "",
                "sig": "ECDSA (secp256k1) signature of this exact json with no space and 'sig:\"\"'"
            },
            {
                "accepted": true,
                "reason": "",
                "sig": "ECDSA (secp256k1) signature of this exact json with no space and 'sig:\"\"'"
            }
        ]
    }
}
```

The wallet needs to insert the `id` field in each opinion object in order to be able
to validate the signature.



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
    ||   <-- sign result ----    ||   // Server: I already signed an input spending this txid, here is the existing signature
```

### Messages format

#### `sign`

Sent at any point in time by a manager who'll soon attempt to unvault and spend one or
more vault utxo.

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




[revaulting_txs]: transactions.md#cancel_tx
[unvault_tx]: transactions.md#unvault_tx
[package_relay]: https://github.com/bitcoin/bitcoin/issues/14895
