# Manager Deployment Process

## Device list

| Device Name                 | Device type | 
| --- | --- |
| OS SD  - _my name_          | class U3 and or V30 or faster |
| Software SD - _my name_     | SD card |
| "Used" Laptop               | Any with SD I/O |
| New Laptop 1                | Any with SD I/O |
| New Laptop 2                | Any with SD I/O |
| Backup 1                    | Metal |
| Backup 2                    | Metal |
| Signing Device - _my name_  | Hardware wallet |
| Revault SD - _my name_      | Any size any speed |
| Noise Pubkey SD - _my name_ | SD card |


- "used" laptop could be the live deployment laptop

## 1. Manager start state:

- at secret date and time, go to secure location with these devices

| Device Name                 | Contents |
| --- | --- | 
| OS - _my name_              | None |
| Software - _my name_        | None |
| "Used" Laptop               | Any  |
| New Laptop 1                | None |
| New Laptop 2                | None |
| Backup 1                    | None |
| Backup 2                    | None |
| Signing Device - _my name_  | None |
| Revault SD - _my name_      | None |
| Noise Pubkey SD - _my name_ | None |


## 2. Preparation

- set-up live linux environment on "Used" laptop
- download a Linux operating system
- verify the signatures/hashes of the downloaded ISO
- unseal the faster SD card
- label it as "OS - _my name_"
- "burn" the ISO to SD card

- download the ceremony tool binary
- download the revault binary 
- download bitcoin binary
- verify the signatures/hashes of each binary

- unseal another SD card
- label it as "Software - _my name_"
- copy the binaries to the SD card
- Shutdown "Used" Laptop completely

- Unseal remaining SD card and label as "Revault SD - _my name_" 

| Device Name                 | Contents |
| --- | --- | 
| OS SD- _my name_              | Verified OS |
| Software SD - _my name_       | Verified revault and ceremony tool binaries |
| "Used" Laptop               | Any  |
| New Laptop 1                | None |
| New Laptop 2                | None |
| Backup 1                    | None |
| Backup 2                    | None |
| Signing Device - _my name_  | None |
| Revault SD - _my name_      | None |
| Noise Pubkey SD - _my name_ | None |


## 3. Set-up new laptops

For both new laptops: 

- Unseal the new laptop box
  - DO NOT BOOT.
  - use the screwdriver to open the laptop
  - remove the wifi card if possible, if not at least disconnect the antennas.
- Insert "OS - _my name_" and install (boot?) OS from SD card
- Insert "Software - _my name_" SD card, copy all files to laptop

| Device Name                 | Contents | 
| --- | --- |
| OS SD - _my name_           | NA |
| Software SD - _my name_     | NA |
| "Used" Laptop               | NA |
| New Laptop 1                | OS, revault, ceremony tool |
| New Laptop 2                | OS, revault, ceremony tool |
| Backup 1                    | None |
| Backup 2                    | None |
| Signing Device - _my name_  | None |
| Revault SD - _my name_      | None |
| Noise Pubkey SD - _my name_ | None |


## 4. Generate Seed Mnemonic and derive keys

Steps:

- (if required) label the dice
- start the ceremony tool on both new laptops
- prepare the backups (to write/ stamp words)
- roll 100 dice (20 throws of 5 dice) and for each throw:
  - type 5 numbers into both laptops as indicated by ceremony tool
    + read dice left to right, exactly how they fall
    + check the displayed word matches on both laptops with each set of 5 numbers (if not, restart ceremony tool and dice rolling)
  - at the end entire mnemonic is displayed
  - compare mnemonic on both laptops and ensure they match (if not, restart ceremony tool and dice rolling)
  - write down/ stamp the word onto the correct backups


| Device Name                 | Contents | 
| --- | --- |
| OS SD - _my name_           | NA |
| Software SD - _my name_     | NA |
| "Used" Laptop               | NA |
| New Laptop 1                | OS, revault, ceremony tool, mnemonic seed, manager (xpriv, xpub), cpfp (xpriv, xpub), noise (privkey, pubkey)|
| New Laptop 2                | OS, revault, ceremony tool, mnemonic seed, manager (xpriv, xpub), cpfp (xpriv, xpub), noise (privkey, pubkey)|
| Backup 1              | mnemonic seed |
| Backup 2              | mnemonic seed |
| Signing Device - _my name_  | None |
| Revault SD - _my name_      | None |
| Noise Pubkey SD - _my name_ | None |


## 5. Move sensitive data

- import mnemonic seed to signing device
- Using ceremony tool, insert the designated SD card to copy data:
  + copy revault binary, unvault xpub 0, cpfp xpub 0 and noise 0 (privkey, pubkey) to "Revault SD - _my name_" to prepare for set-up
  + copy noise 0 pubkey to "Noise Pubkey SD - _my name_" for coordinator


| Device Name                 | Contents | 
| --- | --- |
| OS SD - _my name_           | NA |
| Software SD - _my name_     | NA |
| "Used" Laptop               | NA |
| New Laptop 1                | OS, revault, ceremony tool, mnemonic seed, manager (xpriv, xpub), cpfp (xpriv, xpub), noise (privkey, pubkey)|
| New Laptop 2                | OS, revault, ceremony tool, mnemonic seed, manager (xpriv, xpub), cpfp (xpriv, xpub), noise (privkey, pubkey)|
| Backup 1                    | mnemonic seed |
| Backup 2                    | mnemonic seed |
| Signing Device - _my name_  | unvault (xpriv, xpub)|
| Revault SD - _my name_      | revault, unvault xpub 0, cpfp 0 (xpriv, xpub), noise 0 (privkey, pubkey) |
| Noise Pubkey SD - _my name_ | noise 0 pubkey |

## 6. Destroy new laptops

- Totally destory the new laptops (recommendations?)
- Leave the secure location with all devices because next steps are interactive

| Device Name                 | Contents | 
| --- | --- |
| OS SD - _my name_           | NA |
| Software SD - _my name_     | NA |
| "Used" Laptop               | NA |
| New Laptop 1                | Destroyed |
| New Laptop 2                | Destroyed |
| Backup 1                    | mnemonic seed |
| Backup 2                    | mnemonic seed |
| Signing Device - _my name_  | unvault (xpriv, xpub)|
| Revault SD - _my name_      | revault, unvault xpub 0, cpfp 0 (xpriv, xpub), noise 0 (privkey, pubkey) |
| Noise Pubkey SD - _my name_ | noise 0 pubkey |

## 7. Coordinator communication data-exchange

- retrieve "Noise Pubkey SD - coordinator" from coordinator admin
- give "Noise Pubkey SD - _my name_" to coordinator admin

## 8. Xpub exchange

- Turn on "Used" laptop
- insert Revault SD and open revault binary (or yet another revault set-up tool?)
- (automatically) communicate with each other participant 
  - send my unvault xpub 0 
  - get all stakeholders' vault xpub 0
  - get all other managers' unvault xpub 0
  - Communication routed through coordinator (to avoid copy paste of everything)  
- import vault xpubs to signing device


| Device Name                 | Contents | 
| --- | --- |
| OS SD - _my name_           | NA |
| Software SD - _my name_     | NA |
| "Used" Laptop               | Revault binary, all participants vault/unvault 0 xpubs |
| New Laptop 1                | Destroyed |
| New Laptop 2                | Destroyed |
| Backup 1                    | mnemonic seed |
| Backup 2                    | mnemonic seed |
| Signing Device - _my name_  | unvault (xpriv, xpub), all participants vault/unvault 0 xpubs|
| Revault SD - _my name_      | revault, unvault xpub 0, cpfp 0 (xpriv, xpub), noise 0 (privkey, pubkey) |
| Noise Pubkey SD - _my name_ | noise 0 pubkey |

## 9. Descriptor and Address verification

- Managers check with each other participant that the wallet descriptors match each others'

## Safeguard back-ups

- Move each unvault back-up to a separate secure environment

## Set-up laptop

- Insert "Revault SD _my name_" into deployment laptop
- Install revault binary and follow set-up wizard

## Test all spending paths

- Use signet or testnet
- Ensure coordinator is running
- Test signing revocation transactions
- Test delegation process
- Test manual cancel
- Test manual emergency
- Test automatic cancel (breach revault policy)


