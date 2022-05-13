# Revault transactions

All transactions are version 2 and use version 0 native Segwit scripts.

We use [miniscript](http://bitcoin.sipa.be/miniscript/) and output descriptors to generally
describe outputs spending policies.

We denote `N` the number of stakeholders, `M` the number of fund managers (allowed
to unlock the unvault transaction output along with the cosigning servers after a timeout),
and `X` the CSV value in the unvault transaction. Each transaction type here signals 
replace-by-fee (RBF).

While a larger number of participants is technically possible, we explicitly limit the number of
stakeholders and managers to `20`.

- [deposit_tx](#deposit_tx)
- [unvault_tx](#unvault_tx)
- [spend_tx](#spend_tx)
- [cancel_tx](#cancel_tx)
- [emergency_txs](#emergency_txs)
    - [deposit_emergency_tx](#deposit_emergency_tx)
    - [unvault_emergency_tx](#unvault_emergency_tx)
- [bypass_tx](#bypass_tx)


## deposit_tx

The transaction which deposits into the vault.

#### IN

N/A

#### OUT

At least one output paying to `deposit_descriptor`, with:
```
deposit_descriptor = wsh(deposit_witness_script)
deposit_witness_script = thresh(N, pubkey1, pubkey2, ..., pubkeyN)
```


## unvault_tx

The transaction which spends the [`deposit_tx`](#deposit_tx) deposit output, and creates an
unvault output spendable by the `N` stakeholders or the managers (along with the cosigning
servers) after `X` blocks.

The Unvault transaction is signed using a fixed `6 sat/WU` feerate. This is a
completely arbitrary value that was chosen to avoid blocking operations too early
in case of a huge load of transactions on the network and an increase of the
network mempools minimum feerate.  
This transaction's fees can be bumped if not competitive (using the CPFP output) but
it will likely not be relayed if the network mempools minimum feerate goes above
`24 sat/vb` until [package relay][package_relay] is deployed on the Bitcoin network.

- version: 2
- locktime: 0

#### IN

- count: 1
- inputs[0]:
    - txid: `<deposit_tx txid>`
    - vout: `<deposit_tx vout>`
    - sequence: `0xfffffffd`
    - scriptSig: `<empty>`
    - witness: `satisfy(deposit_descriptor)`

#### OUT

- count: 2
- outputs[0]:
    - value: `<deposit_tx output value - tx_fee - 330>`
    - scriptPubkey: `unvault_descriptor`

- outputs[1]:
    - value: `30000`
    - scriptPubkey: `cpfp_descriptor`

With:
```
unvault_descriptor = wsh(unvault_witness_script)
unvault_witness_script = or(1@thresh(len(stakeholders), stakeholders), 9@and(thresh(len(managers), managers), and(thresh(len(cosigners), cosigners), older(X))))
```

```
cpfp_descriptor = wsh(cpfp_witness_script)
cpfp_witness_script = thresh(1, pubkey1, pubkey2, ..., pubkeyM) # The pubkeys being the managers'
```

## spend_tx

The transaction which spends one or many [`unvault_tx`](#unvault_tx) `output[0]` by the [`M` + cosigners]
path, only spendable after `X` blocks.
The CPFP output value is adjusted depending on the actual transaction size.

- version: 2
- locktime: 0

#### IN

- count: 1
- inputs[0]:
    - txid: `<unvault_tx txid>`
    - sequence: `X`
    - scriptSig: `<empty>`
    - witness: `satisfy(unvault_descriptor)`

#### OUT

- count: 1
- outputs[0]:
    - value: `2 * 32 * tx_size_vbytes`
    - scriptPubkey: `cpfp_descriptor`
- outputs[1]:
    - value: N/A (may be many of them)
    - scriptPubkey: N/A
- outputs[n]:
    - value: N/A (may be many of them)
    - scriptPubkey: N/A


## cancel_tx

The transaction which spends the [`unvault_tx`](#unvault_tx) `output[0]` using the N-of-N path and 
pays back to a deposit output (it is therefore another vault deposit transaction).

The Cancel transaction is signed using the `ALL | ANYONECANPAY` signature hash flag, to
allow watchtowers (or anyone else) to attach fee-bumping inputs.

The Cancel transaction is signed at a fixed `22 sat/WU` feerate. This is in order to
reduce the funds burden on *each* of the watchtowers.

- version: 2
- locktime: 0

#### IN

- count: 1
- inputs[0]:
    - txid: `<unvault_tx txid>`
    - vout: 0
    - sequence: `0xfffffffd`
    - scriptSig: `<empty>`
    - witness: `satisfy(unvault_descriptor)`

#### OUT

- count: 1
- outputs[0]:
    - value: `<unvault_tx outputs[0] value - tx_fee>`
    - scriptPubkey: `deposit_descriptor`


## emergency_txs

Emergency transactions are used as deterrents against threats targeting stakeholders'
funds. They lock coins to what we call an EDV (Emergency Deep Vault): a script chosen
by the participants and kept obfuscated by the properties of P2WSH, as the emergency
transactions are never meant to be used.

Both Emergency transactions are signed at a fixed `75 sat/WU` feerate.

Both Emergency transaction are signed using the `ALL | ANYONECANPAY` signature hash flag,
to allow watchtowers (or anyone else) to attach fee-bumping inputs.

The Emergency `scriptPubKey` is not known to the managers.


### deposit_emergency_tx

The transaction which spends the [`deposit_tx`](#deposit_tx) output to the EDV by the `N`-of-`N` path.

- version: 2
- locktime: 0

#### IN

- count: 1
- inputs[0]:
    - txid: `<deposit_tx txid>`
    - vout: `<deposit_tx vout>`
    - sequence: `0xfffffffd`
    - scriptSig: `<empty>`
    - witness: `satisfy(deposit_descriptor)`

#### OUT

- count: 1
- outputs[0]:
    - value: `<deposit_tx output value - fees>`
    - scriptPubkey: `0x00 SHA256(<EDV_script>)`


### unvault_emergency_tx

This transaction spends the [`unvault_tx`](#unvault_tx) `output[0]` to the EDV by the `N`-of-`N` path.

- version: 2
- locktime: 0

#### IN

- count: 1
- inputs[0]:
    - txid: `<unvault_tx txid>`
    - vout: 0
    - sequence: `0xfffffffd`
    - scriptSig: `<empty>`
    - witness: `satisfy(unvault_descriptor)`

#### OUT

- count: 1
- outputs[0]:
    - value: `<unvault_tx outputs[0] value - fees>`
    - scriptPubkey: `0x00 SHA256(<EDV_script>)`


## bypass_tx

Bypass txs are used in the case where stakeholders need immediate access to funds. Bypass 
txs avoid the controls on expenses set by stakeholders and enforced by watchtowers. A 
Bypass tx spends the [`deposit_tx`](#deposit_tx) and pays to arbitrary addresses.

- version: 2
- locktime: 0

#### IN

- count: 1
- inputs[0]:
    - txid: `<deposit_tx txid>`
    - vout: `<deposit_tx vout>`
    - sequence: `0xfffffffd`
    - scriptSig: `<empty>`
    - witness: `satisfy(deposit_descriptor)`

#### OUT

Unspecified


[package_relay]: https://github.com/bitcoin/bitcoin/issues/14895
