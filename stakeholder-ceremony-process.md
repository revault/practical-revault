# Stakeholder Ceremony Process

## Device list

| Device Name                 | Device type | 
| --- | --- |
| OS - _my name_              | class U3 and or V30 or faster |
| Software - _my name_        | ? |
| Laptop 1                    | Any with SD reader |
| Laptop 2                    | Any with SD reader |
| Vault Backup 1              | Metal |
| Vault Backup 2              | Metal |
| Emergency Backup 1          | Archival Paper |
| Emergency Backup 2          | Archival Paper |
| Signing Device - _my name_  | Hardware wallet |
| Noise Pubkey - _my name_    | Any size any speed |
| xpub - _my name_    1       | Any size any speed |
|                      .        ||
|                      .        ||
|                      .        ||
| xpub - _my name_  N_s + N_m | Any size any speed | 
| Revault Device - _my name_  | Any size any speed |
| Watchtower - _my name_      | Any size any speed |


## Stakeholder start state:

| Device Name                 | Contents |
| --- | --- | 
| OS - _my name_              | Verified OS |
| Software - _my name_        | Verified revault, watchtower and ceremony tool binaries, EFF diceware list |
| Laptop 1                    | None |
| Laptop 2                    | None |
| Vault Backup 1              | None |
| Vault Backup 2              | None |
| Emergency Backup 1          | None |
| Emergency Backup 2          | None |
| Signing Device - _my name_  | None |
| Noise Pubkey - _my name_    | None |
| xpub - _my name_    1       | None |
|                      .        ||
|                      .        ||
|                      .        ||
| xpub - _my name_  N_s + N_m | None |
| Revault Device - _my name_  | None |
| Watchtower - _my name_      | None |


## Prepare Laptops

- Unseal the laptop boxes
  - DO NOT BOOT.
  - use the screwdriver to open the laptops
  - remove the wifi card if possible, if not at least disconnect the antennas.
- Install OS transport device (20 min per laptop?)
- Insert software transport device, copy all file to laptop

| Device Name                 | Contents | 
| --- | --- |
| OS - _my name_              | NA |
| Software - _my name_        | NA |
| Laptop 1                    | OS, revault, watchtower, ceremony tool, EFF diceware list |
| Laptop 2                    | OS, revault, watchtower, ceremony tool, EFF diceware list |
| Vault Backup 1              | None |
| Vault Backup 2              | None |
| Emergency Backup 1          | None |
| Emergency Backup 2          | None |
| Signing Device - _my name_  | None |
| Noise Pubkey - _my name_    | None |
| xpub - _my name_    1       | None |
|                      .        ||
|                      .        ||
|                      .        ||
| xpub - _my name_  N_s + N_m | None |
| Revault Device - _my name_  | None |
| Watchtower - _my name_      | None |

## Generate Seed Mnemonic and derive keys

Steps:

- (if required, label the dice)
- open the EFF Diceware list, on both latops
- roll 100 dice (20 rolls of 5 dice or equivalent), while stamping/writing on the backups.
- on the EFF Diceware list, for each 5 dice scroll to find the corresponding word. (Scroll, do not type anything).
  - copy the word, paste it on the key generation software (need name and format for this.). Do this on both laptops.
  - write down the word on a backup paper sheet
  - once all words have been written down, and copy-pasted, do one last check they match.
- click "generate" on the software on both computers
  - check that the results are identical on both computers
  - write down the resulting mnemonics (optional, recommended) on paper

| Device Name                 | Contents | 
| --- | --- |
| OS - _my name_              | NA |
| Software - _my name_        | NA |
| Laptop 1                    | OS, revault, watchtower, ceremony tool, EFF diceware list, mnemonic seed, xpriv, noise privkey, wt xpriv, wt noise privkey |
| Laptop 2                    | OS, revault, watchtower, ceremony tool, EFF diceware list, mnemonic seed, xpriv, noise privkey, wt xpriv, wt noise privkey |
| Vault Backup 1              | mnemonic seed |
| Vault Backup 2              | mnemonic seed |
| Emergency Backup 1          | (alt) mnemonic seed |
| Emergency Backup 2          | (alt) mnemonic seed |
| Signing Device - _my name_  | None |
| Noise Pubkey - _my name_    | None |
| xpub - _my name_    1       | None |
|                      .        ||
|                      .        ||
|                      .        ||
| xpub - _my name_  N_s + N_m | None |
| Revault Device - _my name_  | None |
| Watchtower - _my name_      | None |

## Load public data to share

| Device Name                 | Contents | 
| --- | --- |
| OS - _my name_              | NA |
| Software - _my name_        | NA |
| Laptop 1                    | OS, revault, watchtower, ceremony tool, EFF diceware list, mnemonic seed, xpriv, noise privkey, wt xpriv, wt noise privkey |
| Laptop 2                    | OS, revault, watchtower, ceremony tool, EFF diceware list, mnemonic seed, xpriv, noise privkey, wt xpriv, wt noise privkey |
| Vault Backup 1              | mnemonic seed |
| Vault Backup 2              | mnemonic seed |
| Emergency Backup 1          | (alt) mnemonic seed |
| Emergency Backup 2          | (alt) mnemonic seed |
| Signing Device - _my name_  | None |
| Noise Pubkey - _my name_    | noise pubkey |
| xpub - _my name_    1       | xpub |
|                      .        ||
|                      .        ||
|                      .        ||
| xpub - _my name_  N_s + N_m | xpub |
| Revault Device - _my name_  | None |
| Watchtower - _my name_      | None |


## Share all pub data among participants and pick up coordinator noise pubkey, move to laptops

| Device Name                 | Contents | 
| --- | --- |
| OS - _my name_              | NA |
| Software - _my name_        | NA |
| Laptop 1                    | OS, revault, watchtower, ceremony tool, EFF diceware list, mnemonic seed, xpriv, noise privkey, wt xpriv, wt noise privkey, {xpub for s in S}, {xpub for m in M}, coordinator noise pubkey |
| Laptop 2                    | OS, revault, watchtower, ceremony tool, EFF diceware list, mnemonic seed, xpriv, noise privkey, wt xpriv, wt noise privkey, {xpub for s in S}, {xpub for m in M}, coordinator noise pubkey |
| Vault Backup 1              | mnemonic seed |
| Vault Backup 2              | mnemonic seed |
| Emergency Backup 1          | (alt) mnemonic seed |
| Emergency Backup 2          | (alt) mnemonic seed |
| Signing Device - _my name_  | None |
| Coordinator Noise Pubkey    | Coordinator noise pubkey |
| xpub - stk 1                | xpub stk 1 |
|                      .        ||
|                      .        ||
|                      .        ||
| xpub - stk N_s              | xpub stk N_s |
| xpub - man 1                | xpub man 1 |
|                      .        ||
|                      .        ||
|                      .        ||
| xpub - man N_m              | xpub man N_m |
| Revault Device - _my name_  | None |
| Watchtower - _my name_      | None |

## Generate miniscript descriptors and emergency descriptor

- Click generete on both laptops
- Verify resulting descriptors match

| Device Name                 | Contents | 
| --- | --- |
| OS - _my name_              | NA |
| Software - _my name_        | NA |
| Laptop 1                    | OS, revault, watchtower, ceremony tool, EFF diceware list, mnemonic seed, xpriv, noise privkey, wt xpriv, wt noise privkey, {xpub for s in S}, {xpub for m in M}, coordinator noise pubkey, deployment miniscript descriptor, emergency descriptor |
| Laptop 2                    | OS, revault, watchtower, ceremony tool, EFF diceware list, mnemonic seed, xpriv, noise privkey, wt xpriv, wt noise privkey, {xpub for s in S}, {xpub for m in M}, coordinator noise pubkey, deployment miniscript descriptor, emergency descriptor |
| Vault Backup 1              | mnemonic seed |
| Vault Backup 2              | mnemonic seed |
| Emergency Backup 1          | (alt) mnemonic seed |
| Emergency Backup 2          | (alt) mnemonic seed |
| Signing Device - _my name_  | None |
| Coordinator Noise Pubkey    | Coordinator noise pubkey |
| xpub - stk 1                | xpub stk 1 |
|                      .        ||
|                      .        ||
|                      .        ||
| xpub - stk N_s              | xpub stk N_s |
| xpub - man 1                | xpub man 1 |
|                      .        ||
|                      .        ||
|                      .        ||
| xpub - man N_m              | xpub man N_m |
| Revault Device - _my name_  | None |
| Watchtower - _my name_      | None |

# Load the signing device

| Device Name                 | Contents | 
| --- | --- |
| OS - _my name_              | NA |
| Software - _my name_        | NA |
| Laptop 1                    | OS, revault, watchtower, ceremony tool, EFF diceware list, mnemonic seed, xpriv, noise privkey, wt xpriv, wt noise privkey, {xpub for s in S}, {xpub for m in M}, coordinator noise pubkey, deployment miniscript descriptor, emergency descriptor |
| Laptop 2                    | OS, revault, watchtower, ceremony tool, EFF diceware list, mnemonic seed, xpriv, noise privkey, wt xpriv, wt noise privkey, {xpub for s in S}, {xpub for m in M}, coordinator noise pubkey, deployment miniscript descriptor, emergency descriptor |
| Vault Backup 1              | mnemonic seed |
| Vault Backup 2              | mnemonic seed |
| Emergency Backup 1          | (alt) mnemonic seed |
| Emergency Backup 2          | (alt) mnemonic seed |
| Signing Device - _my name_  | xpriv, deployment miniscript descriptors |
| Coordinator Noise Pubkey    | Coordinator noise pubkey |
| xpub - stk 1                | xpub stk 1 |
|                      .        ||
|                      .        ||
|                      .        ||
| xpub - stk N_s              | xpub stk N_s |
| xpub - man 1                | xpub man 1 |
|                      .        ||
|                      .        ||
|                      .        ||
| xpub - man N_m              | xpub man N_m |
| Revault Device - _my name_  | None |
| Watchtower - _my name_      | None |

# Load the revault transport device

| Device Name                 | Contents | 
| --- | --- |
| OS - _my name_              | NA |
| Software - _my name_        | NA |
| Laptop 1                    | OS, revault, watchtower, ceremony tool, EFF diceware list, mnemonic seed, xpriv, noise privkey, wt xpriv, wt noise privkey, {xpub for s in S}, {xpub for m in M}, coordinator noise pubkey, deployment miniscript descriptor, emergency descriptor |
| Laptop 2                    | OS, revault, watchtower, ceremony tool, EFF diceware list, mnemonic seed, xpriv, noise privkey, wt xpriv, wt noise privkey, {xpub for s in S}, {xpub for m in M}, coordinator noise pubkey, deployment miniscript descriptor, emergency descriptor |
| Vault Backup 1              | mnemonic seed |
| Vault Backup 2              | mnemonic seed |
| Emergency Backup 1          | (alt) mnemonic seed |
| Emergency Backup 2          | (alt) mnemonic seed |
| Signing Device - _my name_  | xpriv, deployment miniscript descriptors |
| Coordinator Noise Pubkey    | Coordinator noise pubkey |
| xpub - stk 1                | xpub stk 1 |
|                      .        ||
|                      .        ||
|                      .        ||
| xpub - stk N_s              | xpub stk N_s |
| xpub - man 1                | xpub man 1 |
|                      .        ||
|                      .        ||
|                      .        ||
| xpub - man N_m              | xpub man N_m |
| Revault Device - _my name_  | revault binary, noise privkey, deployment miniscript descriptor, emergency descriptor, wt noise pubkey, coordinator noise pubkey |
| Watchtower - _my name_      | None |

# Load watchtower transport device

| Device Name                 | Contents | 
| --- | --- |
| OS - _my name_              | NA |
| Software - _my name_        | NA |
| Laptop 1                    | OS, revault, watchtower, ceremony tool, EFF diceware list, mnemonic seed, xpriv, noise privkey, wt xpriv, wt noise privkey, {xpub for s in S}, {xpub for m in M}, coordinator noise pubkey, deployment miniscript descriptor, emergency descriptor |
| Laptop 2                    | OS, revault, watchtower, ceremony tool, EFF diceware list, mnemonic seed, xpriv, noise privkey, wt xpriv, wt noise privkey, {xpub for s in S}, {xpub for m in M}, coordinator noise pubkey, deployment miniscript descriptor, emergency descriptor |
| Vault Backup 1              | mnemonic seed |
| Vault Backup 2              | mnemonic seed |
| Emergency Backup 1          | (alt) mnemonic seed |
| Emergency Backup 2          | (alt) mnemonic seed |
| Signing Device - _my name_  | xpriv, deployment miniscript descriptors |
| Coordinator Noise Pubkey    | Coordinator noise pubkey |
| xpub - stk 1                | xpub stk 1 |
|                      .        ||
|                      .        ||
|                      .        ||
| xpub - stk N_s              | xpub stk N_s |
| xpub - man 1                | xpub man 1 |
|                      .        ||
|                      .        ||
|                      .        ||
| xpub - man N_m              | xpub man N_m |
| Revault Device - _my name_  | revault binary, noise privkey, deployment miniscript descriptor, emergency descriptor, wt noise pubkey, coordinator noise pubkey |
| Watchtower - _my name_      | watchtower binary, wt xpriv, wt noise privkey, my noise pubkey, deployment miniscript descriptor, coordinator noise pubkey, emergency descriptor |


## Complete Backups

| Device Name                 | Contents | 
| --- | --- |
| OS - _my name_              | NA |
| Software - _my name_        | NA |
| Laptop 1                    | OS, revault, watchtower, ceremony tool, EFF diceware list, mnemonic seed, xpriv, noise privkey, wt xpriv, wt noise privkey, {xpub for s in S}, {xpub for m in M}, coordinator noise pubkey, deployment miniscript descriptor, emergency descriptor |
| Laptop 2                    | OS, revault, watchtower, ceremony tool, EFF diceware list, mnemonic seed, xpriv, noise privkey, wt xpriv, wt noise privkey, {xpub for s in S}, {xpub for m in M}, coordinator noise pubkey, deployment miniscript descriptor, emergency descriptor |
| Vault Backup 1              | mnemonic seed, deployment miniscript descriptor? |
| Vault Backup 2              | mnemonic seed, deployment miniscript descriptor? |
| Emergency Backup 1          | (alt) mnemonic seed, emergency descriptor |
| Emergency Backup 2          | (alt) mnemonic seed, emergency descriptor |
| Signing Device - _my name_  | xpriv, deployment miniscript descriptors |
| Coordinator Noise Pubkey    | Coordinator noise pubkey |
| xpub - stk 1                | xpub stk 1 |
|                      .        ||
|                      .        ||
|                      .        ||
| xpub - stk N_s              | xpub stk N_s |
| xpub - man 1                | xpub man 1 |
|                      .        ||
|                      .        ||
|                      .        ||
| xpub - man N_m              | xpub man N_m |
| Revault Device - _my name_  | revault binary, noise privkey, deployment miniscript descriptor, emergency descriptor, wt noise pubkey, coordinator noise pubkey |
| Watchtower - _my name_      | watchtower binary, wt xpriv, wt noise privkey, my noise pubkey, deployment miniscript descriptor, coordinator noise pubkey, emergency descriptor |


## Destroy all unnecessary sensitive material

| Device Name                 | Contents | 
| --- | --- |
| OS - _my name_              | Destroyed |
| Software - _my name_        | Destroyed |
| Laptop 1                    | Destroyed |
| Laptop 2                    | Destroyed |
| Vault Backup 1              | mnemonic seed, deployment miniscript descriptor? |
| Vault Backup 2              | mnemonic seed, deployment miniscript descriptor? |
| Emergency Backup 1          | (alt) mnemonic seed, emergency descriptor |
| Emergency Backup 2          | (alt) mnemonic seed, emergency descriptor |
| Signing Device - _my name_  | xpriv, deployment miniscript descriptors |
| Coordinator Noise Pubkey    | Destroyed |
| xpub - stk 1                | Destroyed |
|                      .        ||
|                      .        ||
|                      .        ||
| xpub - stk N_s              | Destroyed |
| xpub - man 1                | Destroyed |
|                      .        ||
|                      .        ||
|                      .        ||
| xpub - man N_m              | Destroyed |
| Revault Device - _my name_  | revault binary, noise privkey, deployment miniscript descriptor, emergency descriptor, wt noise pubkey, coordinator noise pubkey |
| Watchtower - _my name_      | watchtower binary, wt xpriv, wt noise privkey, my noise pubkey, deployment miniscript descriptor, coordinator noise pubkey, emergency descriptor |