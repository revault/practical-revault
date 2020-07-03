# Practical revault

A protocol sketch for communication between the different entities in the Revault
architecture.

JSON is used to straightforwardly describe messages here but the actual encoding
isn't decided yet.


## Watchtower

At least one watchtower is ran by every participant. A watchtower needs to be able to
sign any revocation transaction before its corresponding wallet signs the unvaulting
transaction.

In addition, a watchtower will by default revault any unvaulting attempt. We need a way
for an authorized spender to signal its willingness to spend a vault, and for the
watchtower to ACK or NACK it (cheaper and less onchain footprint than try-and-be-canceled).  
For this matter we use the synchronisation server.


### Rough flow

```
  WALLET                      WATCHTOWER
    ||   -- sig emer A ---------->   ||  // Here is A's sig for the emergency transaction.
    ||   -- sig emer_unvault B -->   ||  // Here is B's sig for the unvault emergency transaction.
    ||   -- sig emer B  --------->   ||
    ||   -- sig cancel A   ------>   ||
                (....)
    ||  <--- sig_ack  ---------      || // I succesfully built, checked, and stored revocation transactions.
```

```
  WATCHTOWER                      SYNC_SERVER
    ||   -- get_spend_requests -->   ||  // Is anyone currently willing to spend a vault ?
    ||  <--- spend_requests  -----   ||  // Nope.
    ||
    ||   -- get_spend_requests -->   ||  // Is anyone currently willing to spend a vault ?
    ||  <--- spend_requests  -----   ||  // Yep.
    ||   -- spend_opinion  ------>   ||  // The policy I enforce agree / disagree with this spend attempt.
```

```
 SPENDER's WALLET                SYNC_SERVER
    ||   -- request_spend    ---->   ||  // I'd like to spend this vault.
    ||   -- get_spend_requests -->   ||  // Just checking you made my request public..
    ||   -- get_spend_opinions -->   ||  // What do watchtowers say about this spend ?
    ||  <--- spend_opinions  ---->   ||
                (....)
    ||   -- get_spend_opinions -->   ||
    ||  <--- spend_opinions  ---->   ||  // Eventually all watchtowers responded.
```


### Messages

#### `sig`

Sent at any point in time by a wallet to share a revocation transaction signature with its
watchtower. The wallet must wait for the tower's `sig_ack` before sharing its signature
for the unvault transaction.

```json
{
    "method": "sig",
    "params": {
        "signature": "ALL|ANYONECANPAY Bitcoin ECDSA signature as hex",
        "id": "tx uid"
    }
}
```

The `tx uid` is `sha256(txid)`.


#### `sig_ack`

The watchtower must not send an ACK if it did not build and checked the transactions, or
if it is unable to bump the feerate with its current utxos.
FIXME: What is a reasonable limit for a WT to not send the ACK because of insufficient
funds ?

```json
{
    "result": {
        "ack": true,
        "vault_id": "vault_uid"
    }
}
```

The `vault_uid` is `sha256(vault txid)`.


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

FIXME: should we require a signature here ? Seems to be a belt-and-suspenders check since
there is the enrypted-authenticated message transport protocol.

FIXMENIT: this probably needs a better name :).

```json
{
    "method": "spend_opinion",
    "params": {
        "vault_id": "vault_uid",
        "accept": true,
        "reason": ""
    }
}
```

The `vault_uid` is `sha256(vault txid)`.


#### `request_spend`

Sent by an authorized spender to signal their willingness to spend a vault.

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

Sent by an authorized spender when polling for watchtowers agreement regarding the spend
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
                "reason": ""
            },
            {
                "accepted": true,
                "reason": ""
            },
            {
                "accepted": true,
                "reason": ""
            }
        ]
    }
}
```

The `vault_uid` is `sha256(vault txid)`.



------



## Sync server

The sync server allows wallets to exchange signatures without the need for them to be
interconnected.

As each wallet will verify and store signatures locally, the server isn't trusted and can be
managed by the organisation deploying Revault.

All transactions are signed paying a fixed 253perkw feerate.


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

### Messages

#### `sig`

Sent by a wallet at any point in time to share the signature for a transaction with
everyone.

The wallet can safely post its signature for both the `cancel` and `emergency` of each
`vault` utxo without waiting for others. However, it must wait for everyone to have signed
the `cancel` and `emergency` transactions and its watchtower to have verified and stored
the signature before sharing its signature for the unvault transaction.

Revocation transaction (`cancel` and `emergency`s) are signed with the `ALL|ANYONECANPAY`
flag.

```json
{
    "method": "sig",
    "params": {
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
        "id": "tx uuid"
    }
}
```

The server answers with a (possibly incomplete) list of the signatures for this
transaction.

```json
{
    "result": {
        "signatures": ["sigA", null, "sigC"]
    }
}
```

If a wallet notices its transaction to be absent, it must send it again. It can either
mean the server didn't store it (no explicit ACK) or someone was unhappy with it (no
explicit error from the server).


#### `err_sig`

Sent by a wallet to express its dreadful unhapiness with one of the returned signatures.
It results in the server erasing all the signatures for this transaction and make other
wallets send a new signature.

This should not happen, but hey.

```json
{
    "method": "err_sig",
    "params": {
        "id": "tx uuid"
    }
}
```



------



## Cosigning server

A cosigning server is ran by each non-fund-manager participant. It is happy to sign any
transaction, but only once.

### Rough flow

```
  WALLET                      COSIG_SERVER
    ||   -A-- sign  -------->    ||   // A: I need you to sign this transaction
    ||   <-- sign result ----    ||   // Server: *signs* .. Here you go.
    ||
    ||   -B-- sign  -------->    ||   // B: I need you to sign this same transaction
    ||   <-- sign result ----    ||   // Server: I already signed a transaction spending this txid, here is the existing signature
```

### Messages

#### `sign`

Sent at any point in time by a "trader" who'll soon attempt to unvault and spend a vault
utxo.

```json
{
    "method": "sign",
    "params": {
        "tx": "psbt"
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
