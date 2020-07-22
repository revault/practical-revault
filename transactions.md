# Revault transactions

All transactions are version 2 and use version 0 native Segwit scripts.

We denote `N` the number of participants, `M` the number of authorized spenders (the subset
of the participants allowed to unlock the unvault transaction output along with the
cosigning servers), and `X` the CSV value in the unvault transaction.

- [vault_tx](#vault_tx)
- [unvault_tx](#unvault_tx)
- [spend_tx](#spend_tx)
- [cancel_tx](#cancel_tx)
- [emergency_txs](#emergency_txs)
    - [vault_emergency_tx](#vault_emergency_tx)
    - [unvault_emergency_tx](#unvault_emergency_tx)


## vault_tx

The deposit transaction.

#### IN

N/A

#### OUT

At least one output paying to the following (P2WSH) script:
```
0x00 SHA256(<vault_script>)
```

With `<vault_script>` being:
- If `N <= 20`,
    ```
    N <pubkey 1> <pubkey 2> ... <pubkey N> N CHECKMULTISIG
    ```
- Else,
    ```
    <pubkey 1> CHECKSIGVERIFY <pubkey 2> CHECKSIGVERIFY ... <pubkey N> CHECKSIG
    ```


## unvault_tx

The transaction which spends the [`vault_tx`](vault_tx) deposit output, and creates an
unvault output only spendable by the authorized spenders (along with the cosigning servers)
after `X` blocks.

Note that the `M` pubkeys of the authorized spenders are used first in the output script
for opimisation purpose (they are needed in both spending path anyway).

FIXME(darosior): try miniscript again.

- version: 2
- locktime: 0

#### IN

- count: 1
- inputs[0]:
    - txid: `<vault_tx txid>`
    - vout: N/A
    - sequence: `0xffffffff`
    - scriptSig: `<empty>`
    - witness:
        - If `N <= 20`,
            ```
            0 <sig pubkey 1> <sig pubkey 2> ... <sig pubkey N> <vault_script>
            ```
        - Else,
            ```
            <sig pubkey 1> <sig pubkey 2> ... <sig pubkey N> <vault_script>
            ```

#### OUT

- count: 2
- outputs[0]:
    - value: `<vault_tx output value - tx_fee - 330sats>`
    - scriptPubkey: `0x00 SHA256(<unvault_script>)`, with `<unvault_script>` being:
        ```
        <pubkey 1> CHECKSIGVERIFY ... <pubkey M> CHECKSIGVERIFY
        IF
            # The everyone's-signing path, used for pre-signed transactions
            <pubkey M+1> CHECKSIGVERIFY ... <pubkey N> CHECKSIG
        ELSE
            # The authorized spenders + cosigning servers path, used by the spend transaction
            X CHECKSEQUENCEVERIFY DROP
            <cosigner M+1> CHECKSIGVERIFY ... <cosigner N> CHECKSIG
        ENDIF
        ```
        (Adapted to use `CHECKMULTISIG`s when the number of pubkeys is lesser than or
        equal to `20`)

- outputs[1]:
    - value: `330`sats
    - scriptPubkey: `0x00 SHA256(<cpfp_script>)`, with `<cpfp_script>` being:
        - If `M < 20`
            ```
            M <pubkey 1> <pubkey 2> ... <pubkey M> M CHECKMULTISIG
            ```
        - Else,
            ```
            <pubkey 1> CHECKSIGVERIFY <pubkey 2> CHECKSIGVERIFY ... <pubkey M> CHECKSIG
            ```
        FIXME: do we *really* need a multisig here ? Interaction consequences are
        cumbersome..


## spend_tx

The transaction which spends the unvaulting transaction, only spendable after `X`
blocks.

- version: 2
- locktime: 0

#### IN

- count: 1
- inputs[0]:
    - txid: `<unvault_tx txid>`
    - sequence: `X`
    - scriptSig: `<empty>`
    - witness:
        ```
        0 <sig pubkey 1> ... <sig pubkey M> 1 <sig cosigner M+1> ... <sig cosigner N> <unvault_script>
        ```

#### OUT

- count: 1
- outputs[0]:
    - value: `<unvault_tx outputs[0] value - tx_fee>`
    - scriptPubkey: N/A


## cancel_tx

This transaction spends the `unvault_tx` and pays back to a vault txout (it's thus another
deposit transaction).

- version: 2
- locktime: 0

#### IN

- count: 1
- inputs[0]:
    - txid: `<unvault_tx txid>`
    - vout: 0
    - sequence: `0xfffffffe`
    - scriptSig: `<empty>`
    - witness:
        ```
        0 <sig pubkey 1>... <sig pubkey M> 0 <sig pubkey M+1> ... <sig pubkey N> <unvault_script>
        ```

#### OUT

- count: 1
- outputs[0]:
    - value: `<unvault_tx outputs[0] value - tx_fee>`
    - scriptPubkey: `0x00 SHA256(<vault_script>)`, with `<vault_script>` being:
        - If `N <= 20`,
            ```
            N <pubkey 1> <pubkey 2> ... <pubkey N> N CHECKMULTISIG
            ```
        - Else,
            ```
            <pubkey 1> CHECKSIGVERIFY <pubkey 2> CHECKSIGVERIFY ... <pubkey N> CHECKSIGVERIFY
            ```


## emergency_txs

Emergency transactions are used as deterrents against threat. They lock coins to what we
call an EDV (Emergency Deep Vault): a script chosen by the participants and kept "secret"
as the emergency transactions are never meant to be used.


### vault_emergency_tx

This transaction spends the `vault_tx` to the EDV.

- version: 2
- locktime: 0

#### IN

- count: 1
- inputs[0]:
    - txid: `<vault_tx txid>`
    - vout: N/A
    - sequence: `0xfffffffe`
    - scriptSig: `<empty>`
    - witness:
        - If `N <= 20`,
            ```
            0 <sig pubkey 1> <sig pubkey 2> ... <sig pubkey N> <vault_script>
            ```
        - Else,
            ```
            <sig pubkey 1> <sig pubkey 2> ... <sig pubkey N> <vault_script>
            ```

#### OUT

- count: 1
- outputs[0]:
    - value: `<vault_tx output value - fees>`
    - scriptPubkey: `0x00 SHA256(<EDV_script>)`


### unvault_emergency_tx

This transaction spends the `unvault_tx` to the EDV.

- version: 2
- locktime: 0

#### IN

- count: 1
- inputs[0]:
    - txid: `<unvault_tx txid>`
    - vout: 0
    - sequence: `0xfffffffe`
    - scriptSig: `<empty>`
    - witness:
        ```
        0 <sig pubkey 1>... <sig pubkey M> 0 <sig pubkey M+1> ... <sig pubkey N> <unvault_script>
        ```

#### OUT

- count: 1
- outputs[0]:
    - value: `<unvault_tx outputs[0] value - fees>`
    - scriptPubkey: `0x00 SHA256(<EDV_script>)`
