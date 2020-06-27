# Practical revault

A protocol sketch for communication between the different entities in the Revault
architecture.

JSON is used to straightforwardly describe messages here but the actual encoding
isn't decided yet.


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
the `cancel` and `emergency` transactions before sharing its signature for the
`unvault_tx`.

```json
{
    "method": "sig",
    "params": {
        "signature": "ecdsa signature as hex",
        "id": "tx uuid"
    }
}
```

The `tx uuid` is `sha256(txid)`. It's used when polling.

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
    "result": ["sigA", null, "sigC"]
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




-----------------------

OLD STUFF



The sync server allows wallets to agree on feerate and exchange signatures without the
need for them to be interconnected.

As each wallet will verify feerate and signatures locally, the server isn't trusted and
can be managed by the organisation.


### Rough flow

```
  WALLET                      SYNC_SERVER
    ||   -A-- get_feerate -->    ||   // A: I'm about to share a sig, what's a good feerate everyone can agree upon ?
    ||   -B-- get_feerate -->    ||
    ||   -C-- get_feerate -->    ||
            ...(polling)
            ...(polling)
    ||
    ||   <-B- feerate -------    ||   // Server: Ok, they're all ready. Here is a decent feerate for this tx.
    ||   <-C- feerate -------    ||
    ||   <-A- feerate -------    ||
    ||
    ||   -B-- ack_feerate -->    ||   // B: ACK. This feerate is reasonnable
    ||   -C-- err_feerate -->    ||   // C: NACK. This feerate is not reasonnable
    ||   -A-- ack_feerate -->    ||
    ||
    ||   -B-- feerate_status --> ||   // B: Is everyone OK with this ?
    ||   -A-- feerate_status --> ||
    ||   -C-- feerate_status --> ||   // C: Has my err been treated ?
            ...(polling)
            ...(polling)
    ||
    ||   -A-- get_feerate -->    ||   // A: Arg someone is unhappy, let's try again
    ||   -B-- get_feerate -->    ||
    ||   -C-- get_feerate -->    ||
            ...(polling)
            ...(polling)
    ||
    ||   <-B- feerate -------    ||   // Server: Ok, here is a new feerate.
    ||   <-C- feerate -------    ||
    ||   <-A- feerate -------    ||
    ||
    ||   -B-- ack_feerate -->    ||
    ||   -C-- ack_feerate -->    ||   // C: Now I'm happy.
    ||   -A-- ack_feerate -->    ||
    ||
    ||   -A-- sig   -------->    ||   // A: Here is my sig at that feerate.
    ||   -C-- sig   -------->    ||
    ||   -B-- sig   -------->    ||
    ||
    ||   -C-- get_sigs  ---->    ||   // C: I gave my sig but now am waiting for the late ones...
    ||   -A-- get_sigs  ---->    ||
    ||   -B-- get_sigs  ---->    ||
            ...(polling)
            ...(polling)
    ||   -A-- err_sig   ---->    ||   // A: Huh lol. This sig isn't valid.
            ...(polling)
            ...(polling)
    ||   <-- getsigs error---    ||   // Server: Someone's is unhappy with the sig, try again.
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

#### Feerate

##### `get_feerate`

Sent by a wallet before signing a transaction, to synchronize the feerate used to sign an
upcoming transaction.

```json
{
    "method": "get_feerate",
    "params": {
        "type": "txtype"
    }
}
```

`txtype` is either `unvault`, `emergency`, `cancel`, or `spend`.


##### `feerate`

The response to `get_feerate` from the server to the wallet, which contains the actual feerate in sat/vB
(FIXME: use weights ?).

```json
{
    "result": {
        "feerate": 0,
    }
}
```


##### `ack_feerate`

```json
{}
```

##### `err_feerate`

```json
{}
```
