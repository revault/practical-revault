# Messages

A description of how information is exchanged between the different entities, as well as the
flow of operations.

These messages are exchanged on top of an encrypted and authenticated communication
channel.


- [Watchtower](#watchtower)
- [Cosigning server](#cosigning-server)
- [Synchronisation server](#sync-server)



## Watchtower

At least one watchtower is ran by every participant, and at most one wallet is paired with
each watchtower. A watchtower needs to be able to sign any revocation transaction before
its corresponding wallet signs the unvaulting transaction.

In addition, a watchtower will by default revault any unvaulting attempt. We need a way
for a manager to signal its willingness to spend a vault, and for the
watchtower to ACK or NACK it (cheaper and less onchain footprint than try-and-be-canceled).  
For this matter we use the synchronisation server.


### Rough flow

```
  WALLET                      WATCHTOWER
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
```

```
 SPENDER's WALLET                SYNC_SERVER
    ||   -- request_spend    ---->   ||  // I'd like to spend this vault.
    ||   -- get_spend_requests -->   ||  // Just checking you made my request public..
    ||   -- get_spend_opinions -->   ||  // What do watchtowers say about this spend ?
    ||  <--- spend_opinions  -----   ||
                (....)
    ||   -- get_spend_opinions -->   ||
    ||  <--- spend_opinions  -----   ||  // Eventually all watchtowers responded.
```


### Messages format

#### `sig`

Sent at any point in time by a wallet to share all signatures for a revocation transaction with its
watchtower. The wallet must wait for the tower's `sig_ack` on all revocation transactions before
sharing its signature for the unvault transaction with the other participants.

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

The response to a `get_spend_requests`. Returns an arbitrarily-sized (can be 0) array of
objects detailing a spending request.

```json
{
    "result": {
        "requests": [
            {
                "transaction": "fully signed spend transaction",
                "timestamp": 0000000,
                "vault_id": "vault_uid"
            }
        ]
    }
}
```

The `vault_uid` is `sha256(vault txid)`.


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
        "vault_id": "vault_uid",
        "accept": true,
        "reason": "",
        "sig": "ECDSA (secp256k1) signature of this utf-8 encoded json with no space and 'sig:\"\"'"
    }
}
```

The `vault_uid` is `sha256(vault txid)`.


#### `request_spend`

Sent by a manager to signal their willingness to spend a vault.

We use a timestamp as watchtowers might accept the same spending attempt in the future.

```json
{
    "method": "request_spend",
    "params": {
        "transaction": "fully signed spend transaction",
        "timestamp": 0000000,
        "vault_id": "vault_uid"
    }
}
```

The `vault_uid` is `sha256(vault txid)`.


#### `get_spend_opinions`

Sent by a manager when polling for watchtowers agreement regarding the spend
attempt identified by `vault_id`.

```json
{
    "method": "get_spend_opinions",
    "params": {
        "vault_id": "vault_uid"
    }
}
```

The `vault_uid` is `sha256(vault txid)`.


#### `spend_opinions`

The response to `get_spend_opinions`, an arbitrarily-sized (can be 0 if no watchtower
responded) array of the response of each watchtower.

```json
{
    "result": {
        "vault_id": "vault_uid",
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

The `vault_uid` is `sha256(vault txid)`.  
The wallet needs to insert the `vault_uid` field in each opinion object in order to be able
to validate the signature.



------



## Sync server

The sync server allows wallets to exchange signatures without the need for them to be
interconnected.

As each wallet will verify and store signatures locally, the server isn't trusted and can be
managed by the organisation deploying Revault.

All transactions are signed paying a fixed 253perkw feerate.
FIXME: see https://github.com/re-vault/practical-revault/issues/15


### Rough flow

```
  WALLET                      SYNC_SERVER
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

### Messages format

#### `sig`

Sent by a wallet at any point in time to share the signature for a transaction with
all participants.

The wallet can safely post its signature for both the `cancel` and `emergency` of each
`vault` utxo without waiting for others. However, it must wait for everyone to have signed
the `cancel` and `emergency` transactions and its watchtower to have verified and stored
the signature before sharing its signature for the unvault transaction.

Revocation transactions (`cancel` and `emergency`s) are signed with the `ALL|ANYONECANPAY`
flag.

```json
{
    "method": "sig",
    "params": {
        "pubkey": "Secp256k1 public key used to sign the transaction (hex)",
        "signature": "Bitcoin ECDSA signature as hex",
        "id": "tx uid"
    }
}
```

The `tx uid` is `sha256(txid)`. It's used when polling.

No explicit ACK from the server as the wallet can just `get_sigs` for its own signature.


#### `get_sigs`

Sent by a wallet to retrieve all signatures for a specific transaction.

```json
{
    "method": "get_sigs",
    "params": {
        "id": "tx uid"
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
        "id": "tx uid"
    }
}
```



------



## Cosigning server

A cosigning server is ran by each non-fund-manager participant. It is happy to sign any
transaction input, but only once.

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

Sent at any point in time by a manager who'll soon attempt to unvault and spend a vault
utxo.

The `index` specifies which input should be signed by the cosigner, as a single spend
transaction might spend from multiple vaults.

```json
{
    "method": "sign",
    "params": {
        "tx": "psbt",
        "index": 0
    }
}
```

The server shall return the existing signature if it already signed a transaction spending
this txid (no matter the actual transaction), or sign it, store this sig for future use,
then return it (once again, no matter the actual transaction).

```json
{
    "result": {
        "signature": "sig as hex"
    }
}
```
