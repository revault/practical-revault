# Manager Ceremony Process

## Device list

| Device Name                 | Device type | 
| --- | --- |
| OS - _my name_              | class U3 and or V30 or faster |
| Software - _my name_        | ? |
| Laptop 1                    | Any with SD reader |
| Laptop 2                    | Any with SD reader |
| Backup 1              | Metal |
| Backup 2              | Metal |
| Signing Device - _my name_  | Hardware wallet |
| Noise Pubkey - _my name_    | Any size any speed |
| xpub - _my name_    1       | Any size any speed |
|                      .        ||
|                      .        ||
|                      .        ||
| xpub - _my name_  N_s + N_m | Any size any speed | 
| Revault Device - _my name_  | Any size any speed |


## Manager start state:

| Device Name                 | Contents | 
| --- | --- |
| OS - _my name_              | Verified OS |
| Software - _my name_        | Verified revault and ceremony tool binaries, EFF diceware list |
| Laptop 1                    | None |
| Laptop 2                    | None |
| Backup 1              | None |
| Backup 2              | None |
| Signing Device - _my name_  | None |
| Noise Pubkey - _my name_    | None |
| xpub - _my name_    1       | None |
|                      .        ||
|                      .        ||
|                      .        ||
| xpub - _my name_  N_s + N_m | None |
| Revault Device - _my name_  | None |


## Prepare Laptops

| Device Name                 | Contents | 
| --- | --- |
| OS - _my name_              | NA |
| Software - _my name_        | NA |
| Laptop 1                    | OS, revault, ceremony tool, EFF diceware list |
| Laptop 2                    | OS, revault, ceremony tool, EFF diceware list |
| Backup 1              | None |
| Backup 2              | None |
| Signing Device - _my name_  | None |
| Noise Pubkey - _my name_    | None |
| xpub - _my name_    1       | None |
|                      .        ||
|                      .        ||
|                      .        ||
| xpub - _my name_  N_s + N_m | None |
| Revault Device - _my name_  | None |

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
| Laptop 1                    | OS, revault, ceremony tool, EFF diceware list, mnemonic seed, xpriv, noise privkey |
| Laptop 2                    | OS, revault, ceremony tool, EFF diceware list, mnemonic seed, xpriv, noise privkey |
| Backup 1              | mnemonic seed |
| Backup 2              | mnemonic seed |
| Signing Device - _my name_  | None |
| Noise Pubkey - _my name_    | None |
| xpub - _my name_    1       | None |
|                      .        ||
|                      .        ||
|                      .        ||
| xpub - _my name_  N_s + N_m | None |
| Revault Device - _my name_  | None |

## Load public data to share

| Device Name                 | Contents | 
| --- | --- |
| OS - _my name_              | NA |
| Software - _my name_        | NA |
| Laptop 1                    | OS, revault, ceremony tool, EFF diceware list, mnemonic seed, xpriv, noise privkey |
| Laptop 2                    | OS, revault, ceremony tool, EFF diceware list, mnemonic seed, xpriv, noise privkey |
| Backup 1              | mnemonic seed |
| Backup 2              | mnemonic seed |
| Signing Device - _my name_  | None |
| Noise Pubkey - _my name_    | noise pubkey |
| xpub - _my name_    1       | xpub |
|                      .        ||
|                      .        ||
|                      .        ||
| xpub - _my name_  N_s + N_m | xpub |
| Revault Device - _my name_  | None |


## Share all pub data among participants and coordinator noise pubkey, move to laptops

| Device Name                 | Contents | 
| --- | --- |
| OS - _my name_              | NA |
| Software - _my name_        | NA |
| Laptop 1                    | OS, revault, ceremony tool, EFF diceware list, mnemonic seed, xpriv, noise privkey, {xpub for s in S}, {xpub for m in M}, coordinator noise pubkey |
| Laptop 2                    | OS, revault, ceremony tool, EFF diceware list, mnemonic seed, xpriv, noise privkey, {xpub for s in S}, {xpub for m in M}, coordinator noise pubkey |
| Backup 1              | mnemonic seed |
| Backup 2              | mnemonic seed |
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

## Generate miniscript descriptors and emergency descriptor

- Click generete on both laptops
- Verify resulting descriptors match

| Device Name                 | Contents | 
| --- | --- |
| OS - _my name_              | NA |
| Software - _my name_        | NA |
| Laptop 1                    | OS, revault, ceremony tool, EFF diceware list, mnemonic seed, xpriv, noise privkey, {xpub for s in S}, {xpub for m in M}, coordinator noise pubkey, deployment miniscript descriptor, emergency descriptor |
| Laptop 2                    | OS, revault, ceremony tool, EFF diceware list, mnemonic seed, xpriv, noise privkey, {xpub for s in S}, {xpub for m in M}, coordinator noise pubkey, deployment miniscript descriptor, emergency descriptor |
| Backup 1              | mnemonic seed |
| Backup 2              | mnemonic seed |
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

# Load the signing device

| Device Name                 | Contents | 
| --- | --- |
| OS - _my name_              | NA |
| Software - _my name_        | NA |
| Laptop 1                    | OS, revault, ceremony tool, EFF diceware list, mnemonic seed, xpriv, noise privkey, {xpub for s in S}, {xpub for m in M}, coordinator noise pubkey, deployment miniscript descriptor, emergency descriptor |
| Laptop 2                    | OS, revault, ceremony tool, EFF diceware list, mnemonic seed, xpriv, noise privkey, {xpub for s in S}, {xpub for m in M}, coordinator noise pubkey, deployment miniscript descriptor, emergency descriptor |
| Backup 1              | mnemonic seed |
| Backup 2              | mnemonic seed |
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

# Load the revault transport device

| Device Name                 | Contents | 
| --- | --- |
| OS - _my name_              | NA |
| Software - _my name_        | NA |
| Laptop 1                    | OS, revault, ceremony tool, EFF diceware list, mnemonic seed, xpriv, noise privkey, {xpub for s in S}, {xpub for m in M}, coordinator noise pubkey, deployment miniscript descriptor, emergency descriptor |
| Laptop 2                    | OS, revault, ceremony tool, EFF diceware list, mnemonic seed, xpriv, noise privkey, {xpub for s in S}, {xpub for m in M}, coordinator noise pubkey, deployment miniscript descriptor, emergency descriptor |
| Backup 1              | mnemonic seed |
| Backup 2              | mnemonic seed |
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
| Revault Device - _my name_  | revault binary, noise privkey, deployment miniscript descriptor, emergency descriptor, coordinator noise pubkey |


## Complete Backups

| Device Name                 | Contents | 
| --- | --- |
| OS - _my name_              | NA |
| Software - _my name_        | NA |
| Laptop 1                    | OS, revault, ceremony tool, EFF diceware list, mnemonic seed, xpriv, noise privkey, {xpub for s in S}, {xpub for m in M}, coordinator noise pubkey, deployment miniscript descriptor, emergency descriptor |
| Laptop 2                    | OS, revault, ceremony tool, EFF diceware list, mnemonic seed, xpriv, noise privkey, {xpub for s in S}, {xpub for m in M}, coordinator noise pubkey, deployment miniscript descriptor, emergency descriptor |
| Backup 1              | mnemonic seed, deployment miniscript descriptor? |
| Backup 2              | mnemonic seed, deployment miniscript descriptor? |
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
| Revault Device - _my name_  | revault binary, noise privkey, deployment miniscript descriptor, emergency descriptor, coordinator noise pubkey |


## Destroy all unnecessary sensitive material

| Device Name                 | Contents | 
| --- | --- |
| OS - _my name_              | Destroyed |
| Software - _my name_        | Destroyed |
| Laptop 1                    | Destroyed |
| Laptop 2                    | Destroyed |
| Backup 1              | mnemonic seed, deployment miniscript descriptor? |
| Backup 2              | mnemonic seed, deployment miniscript descriptor? |
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
| Revault Device - _my name_  | revault binary, noise privkey, deployment miniscript descriptor, emergency descriptor, coordinator noise pubkey |