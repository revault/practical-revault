# Stakeholder Deployment Process

## Device list

| Device Name                 | Device type | 
| --- | --- |
| OS SD  - _my name_          | class U3 and or V30 or faster |
| Software SD - _my name_     | SD card |
| "Used" Laptop               | Any with SD I/O |
| New Laptop 1                | Any with SD I/O |
| New Laptop 2                | Any with SD I/O |
| Vault Backup 1              | Metal |
| Vault Backup 2              | Metal |
| Emergency Backup 1          | Archival Paper |
| Emergency Backup 2          | Archival Paper |
| Signing Device - _my name_  | Hardware wallet |
| Revault SD - _my name_      | Any size any speed |
| Watchtower SD - _my name_   | Any size any speed |
| Emergency Pubkey SD - _my name_  1  | SD card |
|                         ...         |      |
| Emergency Pubkey SD - _my name_  N_s| SD card |
| Emergency Pubkey SD - external 1    | SD card |
|                         ...         |        |
| Emergency Pubkey SD - external X    | SD card |
| Noise Pubkey SD - _my name_         |SD card |


- "Used" laptop could be the live deployment laptop
- Retrieve emergency pubkey SD card from each external party associated with _this stakeholder_ participating in the emergency policy. 

## 1. Stakeholder start state:

- at secret date and time, go to secure location with these devices

| Device Name                 | Contents |
| --- | --- | 
| OS - _my name_              | None |
| Software - _my name_        | None |
| "Used" Laptop               | Any  |
| New Laptop 1                | None |
| New Laptop 2                | None |
| Vault Backup 1              | None |
| Vault Backup 2              | None |
| Emergency Backup 1          | None |
| Emergency Backup 2          | None |
| Signing Device - _my name_  | None |
| Revault SD - _my name_      | None |
| Watchtower SD - _my name_   | None |
| Emergency Pubkey SD - _my name_  1  | None |
|                         ...         |      |
| Emergency Pubkey SD - _my name_  N_s| None |
| Emergency Pubkey SD - external 1    | extern pubkey 1 |
|                         ...         |        |
| Emergency Pubkey SD - external X    | extern pubkey X |
| Noise Pubkey SD - _my name_         | None |


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
- download the watchtower binary
- verify the signatures/hashes of each binary

- unseal another SD card
- label it as "Software - _my name_"
- copy the binaries to the SD card

- Shutdown "Used" Laptop completely

- Unseal remaining SD cards and label:
  - one of them "Revault SD - _my name_" 
  - one of them "Watchtower SD - _my name_" 
  <!-- - (if using) one of them "Cosigner - _my name_" -->
  - one of them "Emergency Pubkey SD - _my name_"

| Device Name                 | Contents |
| --- | --- | 
| OS SD- _my name_              | Verified OS |
| Software SD - _my name_       | Verified revault, watchtower and ceremony tool binaries |
| "Used" Laptop               | Any  |
| New Laptop 1                | None |
| New Laptop 2                | None |
| Vault Backup 1              | None |
| Vault Backup 2              | None |
| Emergency Backup 1          | None |
| Emergency Backup 2          | None |
| Signing Device - _my name_  | None |
| Revault SD - _my name_      | None |
| Watchtower SD - _my name_   | None |
| Emergency Pubkey SD - _my name_  1  | None |
|                         ...         |      |
| Emergency Pubkey SD - _my name_  N_s| None |
| Emergency Pubkey SD - external 1    | extern pubkey 1 |
|                         ...         |        |
| Emergency Pubkey SD - external X    | extern pubkey X |
| Noise Pubkey SD - _my name_         | None |

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
| New Laptop 1                | OS, revault, watchtower, ceremony tool |
| New Laptop 2                | OS, revault, watchtower, ceremony tool |
| Vault Backup 1              | None |
| Vault Backup 2              | None |
| Emergency Backup 1          | None |
| Emergency Backup 2          | None |
| Signing Device - _my name_  | None |
| Revault SD - _my name_      | None |
| Watchtower SD - _my name_   | None |
| Emergency Pubkey SD - _my name_  1  | None |
|                         ...         |      |
| Emergency Pubkey SD - _my name_  N_s| None |
| Emergency Pubkey SD - external 1    | extern pubkey 1 |
|                         ...         |        |
| Emergency Pubkey SD - external X    | extern pubkey X |
| Noise Pubkey SD - _my name_         | None |

## 4. Generate Seed Mnemonic and derive keys

Steps:

- (if required) label the dice
- start the ceremony tool on both new laptops
- prepare the vault and emergency backups (to write/ stamp words)
- roll 100 dice (20 throws of 5 dice) and for each throw:
  - type 5 numbers into both laptops as indicated by ceremony tool
    + read dice left to right, exactly how they fall
    + check the displayed word matches on both laptops with each set of 5 numbers (if not, restart ceremony tool and dice rolling)
  - at the end, two mnemonics displayed; one for vault and one for emergency
  - compare on both laptops and ensure they match (if not, restart ceremony tool and dice rolling)
  - write down/ stamp the word onto the correct backups

| Device Name                 | Contents | 
| --- | --- |
| OS SD - _my name_           | NA |
| Software SD - _my name_     | NA |
| "Used" Laptop               | NA |
| New Laptop 1                | OS, revault, watchtower, ceremony tool, mnemonic seed, vault (xpriv, xpub), emergency (xpriv, xpub), noise (privkey, pubkey), WT noise (privkey, pubkey) |
| New Laptop 2                | OS, revault, watchtower, ceremony tool, mnemonic seed, vault (xpriv, xpub), emergency (xpriv, xpub), noise (privkey, pubkey), WT noise (privkey, pubkey) |
| Vault Backup 1              | vault mnemonic seed |
| Vault Backup 2              | vault mnemonic seed |
| Emergency Backup 1          | emergency mnemonic seed |
| Emergency Backup 2          | emergency mnemonic seed |
| Signing Device - _my name_  | None |
| Revault SD - _my name_      | None |
| Watchtower SD - _my name_   | None |
| Emergency Pubkey SD - _my name_  1  | None |
|                         ...         |      |
| Emergency Pubkey SD - _my name_  N_s| None |
| Emergency Pubkey SD - external 1    | extern pubkey 1 |
|                         ...         |        |
| Emergency Pubkey SD - external X    | extern pubkey X |
| Noise Pubkey SD - _my name_         | None |


## 5. Move sensitive data

- import vault seed into signing device
- Using ceremony tool, insert the designated SD card to copy data:
  + copy all external emergency pubkeys from SD cards onto new laptops
  + import emergency pubkeys to signing device
  + copy my emergency 0 pubkey to N_s SD cards named "Emergency pubkey - _my name_"
  + copy revault binary, Vault 0 xpub, emergency 0 pubkey, noise 0 (privkey,pubkey) and WT 0 noise pubkey to "Revault SD - _my name_" to prepare for set-up
  + copy watchtower binary, vault 0 xpub, noise 0 pubkey, WT 0 (xpriv, xpub), WT 0 noise (privkey, pubkey) to "watchtower SD - _my name_" to prepare for set-up
  + copy my noise pubkey 0 to "Noise Pubkey SD - _my name_"
 

| Device Name                 | Contents | 
| --- | --- |
| OS SD - _my name_           | NA |
| Software SD - _my name_     | NA |
| "Used" Laptop               | NA |
| New Laptop 1                | OS, revault, watchtower, ceremony tool, mnemonic seed, vault (xpriv, xpub), emergency(xpriv, xpub), noise (privkey, pubkey), WT (xpriv, xpub), WT noise (privkey, pubkey) |
| New Laptop 2                | OS, revault, watchtower, ceremony tool, mnemonic seed, vault (xpriv, xpub), emergency(xpriv, xpub), noise (privkey, pubkey), WT (xpriv, xpub), WT noise (privkey, pubkey) |
| Vault Backup 1              | vault mnemonic seed |
| Vault Backup 2              | vault mnemonic seed |
| Emergency Backup 1          | emergency mnemonic seed |
| Emergency Backup 2          | emergency mnemonic seed |
| Signing Device - _my name_  | vault (xpriv, xpub), emergency 0-100 pubkeys |
| Revault SD - _my name_      | revault, vault 0 xpub, emergency 0 pubkey, noise 0 (privkey, pubkey), WT 0 noise  pubkey |
| Watchtower SD - _my name_   | watchtower, vault 0 xpub, noise 0 pubkey, WT 0 noise (privkey, pubkey) |
| Emergency Pubkey SD - _my name_  1  | emergency 0 pubkey, extern pubkey 1, ..., extern pubkey X |
|                         ...         |      |
| Emergency Pubkey SD - _my name_  N_s| emergency 0 pubkey, extern pubkey 1, ..., extern pubkey X |
| Emergency Pubkey SD - external 1    | extern pubkey 1 |
|                         ...         |        |
| Emergency Pubkey SD - external X    | extern pubkey X |
| Noise Pubkey SD - _my name_         | noise pubkey |

## 6. Destroy new laptops & external emergency SD cards

- Totally destory the new laptops (recommendations?)
- Totally destroy all emergency pubkey SD cards from external parties
- Leave the secure location with all devices because next steps are interactive

| Device Name                 | Contents | 
| --- | --- |
| OS SD - _my name_           | NA |
| Software SD - _my name_     | NA |
| "Used" Laptop               | NA |
| New Laptop 1                | Destroyed |
| New Laptop 2                | Destroyed |
| Vault Backup 1              | vault mnemonic seed |
| Vault Backup 2              | vault mnemonic seed |
| Emergency Backup 1          | emergency mnemonic seed |
| Emergency Backup 2          | emergency mnemonic seed |
| Signing Device - _my name_  | vault (xpriv, xpub), emergency 0-100 pubkeys |
| Revault SD - _my name_      | revault, vault 0 xpub, emergency 0 pubkey, noise 0 (privkey, pubkey), WT 0 noise  pubkey |
| Watchtower SD - _my name_   | watchtower, vault 0 xpub, noise 0 pubkey, WT 0 noise (privkey, pubkey) |
| Emergency Pubkey SD - _my name_  1  | emergency 0 pubkey, extern pubkey 1, ..., extern pubkey X |
|                         ...         |      |
| Emergency Pubkey SD - _my name_  N_s| emergency 0 pubkey, extern pubkey 1, ..., extern pubkey X |
| Emergency Pubkey SD - external 1    | Destroyed |
|                         ...         |        |
| Emergency Pubkey SD - external X    | Destroyed|
| Noise Pubkey SD - _my name_         | noise pubkey |

## 7. Coordinator communication data-exchange

- retrieve "Noise Pubkey SD - coordinator" from coordinator admin
- give "Noise Pubkey SD - _my name_" to coordinator admin

## 8. Vault xpub exchange

- Turn on "Used" laptop
- insert Revault SD and open revault binary (or yet another revault set-up tool?)
- (automatically) communicate with each other participant 
  - send my vault xpub 0 
  - get all stakeholders' vault xpub 0
  - get all other managers' unvault xpub 0
  - Communication routed through coordinator (to avoid copy paste of everything)   
- import vault xpubs to signing device


| Device Name                 | Contents | 
| --- | --- |
| OS SD - _my name_           | NA |
| Software SD - _my name_     | NA |
| "Used" Laptop               | revault binary, all participants vault/unvault 0 xpubs |
| New Laptop 1                | Destroyed |
| New Laptop 2                | Destroyed |
| Vault Backup 1              | vault mnemonic seed |
| Vault Backup 2              | vault mnemonic seed |
| Emergency Backup 1          | emergency mnemonic seed |
| Emergency Backup 2          | emergency mnemonic seed |
| Signing Device - _my name_  | my vault (xpriv, xpub), emergency 0-100 pubkeys, all participants' vault/ unvault 0 xpubs |
| Revault SD - _my name_      | revault, vault 0 xpub, emergency 0 pubkey, noise 0 (privkey, pubkey), WT 0 noise  pubkey |
| Watchtower SD - _my name_   | watchtower, vault 0 xpub, noise 0 pubkey, WT 0 noise (privkey, pubkey) |
| Emergency Pubkey SD - _my name_  1  | emergency 0 pubkey, extern pubkey 1, ..., extern pubkey X |
|                         ...         |      |
| Emergency Pubkey SD - _my name_  N_s| emergency 0 pubkey, extern pubkey 1, ..., extern pubkey X |
| Emergency Pubkey SD - external 1    | Destroyed |
|                         ...         |        |
| Emergency Pubkey SD - external X    | Destroyed|
| Noise Pubkey SD - coordinator       | noise pubkey |


## 9. Stakeholder emergency data-exchange

- Stakeholders each give all other stakeholders a copy of their emergency miniscript
- Stakeholders import this data into their revault device and signing device
- emergency address is generated deterministally from all emergency miniscript fragments


## 10. Descriptor and Address verification

- Stakeholders check with each other stakeholder that the emergency address matches each others' (confidentially)
- Participants check with each other participant that the wallet descriptors match each others'


## 11. Safeguard back-ups

- Complete emergency back-ups with emergency miniscript info
- Move each emergency back-up to a separate secure environment
- Move each vault back-up to a separate secure environment


## 12. Set-up laptop

- Insert "Revault SD _my name_" into deployment laptop
- Install revault binary and follow set-up wizard
- insert "Noise Pubkey SD - Coordinator"

## 13. Set-up watchtower

- Insert "Watchtower SD _my name_" into WT server
- Install watchtower binary and follow set-up wizard
- insert "Noise Pubkey SD - Coordinator"


## 14. Test all spending paths

- Use signet or testnet
- Ensure coordinator is running
- Test signing revocation transactions
- Test delegation process
- Test manual cancel
- Test manual emergency
- Test automatic cancel (breach revault policy)


