# Manager Deployment Process

## Device list

| Device Name                 | Device type | 
| --- | --- |
| OS SD  - coordinator            | class U3 and or V30 or faster |
| Software SD - coordinator       | SD card |
| "Used" Laptop                   | Any with SD I/O |
| Coordinator Server              | Server |
| Noise Pubkey SD - coordinator 1 | SD card |
|                     ... 
| Noise Pubkey SD - coordinator N_s + N_m | SD card |


## 1. Coordinator start state:

- at secret date and time, go to secure location with these devices

| Device Name                 | Contents |
| --- | --- | 
| OS SD  - coordinator          | None |
| Software SD - coordinator     | None |
| "Used" Laptop                 | Any |
| Coordinator Server            | Server |
| Noise Pubkey SD - coordinator 1 | None |
|                     ... 
| Noise Pubkey SD - coordinator N_s + N_m | None |


## 2. Preparation

- set-up live linux environment on "Used" laptop
- download a Linux operating system
- verify the signatures/hashes of the downloaded ISO

- unseal the faster SD card
- label it as "OS - coordinator"
- "burn" the ISO to SD card

- unseal another SD card
- label it as "Software - coordinator"
- download the coordinator binary
- verify the signature/hash of the binary
- copy the binary to the SD card

- Shutdown "Used" Laptop completely

- Unseal remaining SDs and label as: "Noise Pubkey - Coordinator"

| Device Name                 | Contents |
| --- | --- | 
| OS SD- _my name_              | Verified OS |
| Software SD - _my name_       | Verified revault and ceremony tool binaries, EFF diceware list |
| "Used" Laptop                 | Any |
| Coordinator Server            | OS, coordinator binary |
| Noise Pubkey SD - coordinator 1 | None |
|                     ... ||
| Noise Pubkey SD - coordinator N_s + N_m | None |


## 3. Set-up coordinator server

- Insert "OS - coordinator" and install OS from SD card
- Insert "Software - coordinator" SD card, copy coordinator binary

| Device Name                 | Contents | 
| --- | --- |
| OS SD  - coordinator          | NA |
| Software SD - coordinator     | NA |
| "Used" Laptop                 | Any |
| Coordinator Server            | OS, coordinator binary |
| Noise Pubkey SD - coordinator 1 | None |
|                     ... ||
| Noise Pubkey SD - coordinator N_s + N_m | None |

## 4. Follow set-up wizard

- Generate noise keys

| Device Name                  | Contents | 
| --- | --- |
| OS SD  - coordinator          | NA |
| Software SD - coordinator     | NA |
| "Used" Laptop                 | Any |
| Coordinator Server            | OS, coordinator binary |
| Noise Pubkey SD - coordinator 1 | None |
|                     ... ||
| Noise Pubkey SD - coordinator N_s + N_m | None |


## 5. Prepare for data-exchange

- copy noise pubkey to SD cards


| Device Name                  | Contents | 
| --- | --- |
| OS SD  - coordinator          | NA |
| Software SD - coordinator     | NA |
| "Used" Laptop                 | Any |
| Coordinator Server            | OS, coordinator binary |
| Noise Pubkey SD - coordinator 1 | Noise Pubkey |
|                     ... ||
| Noise Pubkey SD - coordinator N_s + N_m | Noise Pubkey |


## 6. Data-exchange

- Retrieve "noise pubkey SD" from each participant
- Give "Noise Pubkey SD - coordinator" to each participant

| Device Name                  | Contents | 
| --- | --- |
| OS SD  - coordinator          | NA |
| Software SD - coordinator     | NA |
| "Used" Laptop                 | Any |
| Coordinator Server            | OS, coordinator binary |
| Noise Pubkey SD -  1          | Noise Pubkey |
|                     ... ||
| Noise Pubkey SD -  N_s + N_m  | Noise Pubkey |

## 7. Test all spending paths

- Use signet or testnet
- Ensure coordinator is running
- Test signing revocation transactions
- Test delegation process
- Test manual cancel
- Test manual emergency
- Test automatic cancel (breach revault policy)


