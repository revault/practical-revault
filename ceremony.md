# Specification to Initialize Revault Infrastructure

WORK-IN-PROGRESS - DO NOT USE (yet)

THERE IS NOT SUCH THING AS PERFECT SECURITY.
thIs iS NoT SEcuRitY ADviCe.

# Motivation

Key management is fundamental to bitcoin custody. As an open-source, multi-party custody protocol, Revault builds on the strong foundation of a rigorous key-generation and exchange ceremony as the root of trust for the entire security infrastructure. 

This document is concerned with how keys are generated and transported to the devices on which they will be stored. The entire process involves mitigations for a wide array of risks, which will be discussed in a separate document. Moreover, integrity verification of the software stack (OS, Bitcoin, Revault) which ultimately handles the secrets must be carried out as part of the initialization. 

It is critical that _confidentiality_ of secrets is maintained by secret-holders, and that secret-holders use backups to ensure _availability_ in case of losses and failures, and that the _integrity_ of secrets is verified during the ceremony (and from time to time afterwards). A forensic trail must be laid in case a critical failure occurs to help understand what went wrong and where legal liability falls. 

By adhering to the ceremony specification, a team deploying Revault creates confidence in their ongoing custodial operations. Individuals can be confident that others in their team aren't naively cutting corners that could cause catastrohpic failures. With the inclusion of notaries or lawyers in the process, teams and their constituent individuals may aquire legal protection when failures occur.

# Revault

## Vocabulary

- Participant: all participants in a Revault deployment will have one or two roles as a _stakeholder_ and/ or _manager_.
- Stakeholder: Controls a secret for the high-threshold multi-signature which is the primary protection for funds in custody.
- Manager: Controls a secret for funds which are delegated to them
- Emergency Wallet (EW): An external wallet to the Revault deployment which is used as a deterrent against physical threats.
- Watchtower: Automated server to enforce unvault policies set by it's operator
- Coordinator Admin: _NOT A PARTICIPANT_, administrates the coordinator
- Coordinator: Automated server to reduce coordination complexity for communication 
- Signing device: Offline signing device, with a firmware that supports Revault transactions
- Transport device: A secure-digital (SD or MicroSD) card to temporarily store and transport critical data and software

## Deployment

### Parameters and Notation

<!-- This might be useful later for a formal model of number of steps & devices -->

- N_s: Number of stakeholders
- N_m: Number of managers
- k: manager signature threshold
- t_l: relative time-lock length
- s: stakeholder id
- S: stakeholder set
- m: manager id
- M: manager set
- c: coordinator id
- C: coordinator set
- l1_s: first laptop of stakeholder s
- l2_s: second laptop of stakeholder s
- l1_m: first laptop of manager m
- l2_m: second laptop of manager m
- rd: revault device id
- RD: revault device map to participant {(rd, s) or (rd, m)}
- sd: signing device id
- SD: signing device map to participant {(sd, s) or (sd, m)}
- b: backup id
- B: backup map to paricipant {(b, s) or (b, m)
- EA': emergency address (partial)
- EA: emergency address (complete)
- td: transport device id
- TD: transport device map to target {(td, s) or (td, m) or (td, w) or (td, c) or (td, EA') or (td, EA) or (td, l1_s) or (td, l2_s) or (td, l1_m) or (td, l2_m)}
- w: watchtower id
- W: watchtower map to stakeholder {(w,s)}
- N_w: number of watchtowers

### Infrastructure

- At least 1 coordinator server per deployment
- At least 1 watchtower per stakeholder
- 1 revault device (revaultd and GUI) per participant
- 1 signing device (typically hardware wallet) per participant
- At least 2 backup per participant

## Process

### shopping list

- each participant to procure **1 signing device**, directly from manufacturer, if possible not in their own name/address, if possible in person, cash. Participants should not all procure their signing device from the same manufacturer. _Typical recommendation is to use a hardware wallet as the signing device. Offline computers or otherwise may be used but are not recommended for usability (in a secure manner)_

- each participant to procure **2 brand new laptops** (or other general purpose computing device) of different brands, from 2 different shops, if possible in person, in cash, and not at the store closest to their place of residence or work. Any laptop should work AS LONG AS it has an SD (or MicroSD) card reader. Preferably no Apple computer.

- each stakeholder to procure a transport device for:
    + each other stakeholder and manager to share: their xpub. 
    + their revault device to include: the miniscript descriptors, the noise pubkey of their watchtower, the noise pubkey of the coordinator, their noise privkey, verified revault binary
    + the coordinator to share: their noise pubkey
    + their watchtower to share: wt xpriv, wt noise private key, deployment miniscript descriptors, coordinator noise pubkey, stakeholder noise pubkey, verified WT binary
    + the ceremony to include: a verified OS (td class U3 and or V30 or faster)
    + the ceremony to include: revault & watchtower & ceremony tool binaries

- each manager to procure a transport device for:
    + each other stakeholder and manager to share: their xpub
    + their revault device to include: the miniscript descriptors, coordinator noise pubkey, their noise privkey, verified revault binary
    + the coordinator to share: their noise pubkey
    + the ceremony to include: an OS (td class U3 and or V30 or faster)
    + the ceremony to include: revault & ceremony tool binaries

- coordinator admin to procure a transport device for:
    + each other stakeholder and manager to share: coordinator noise pubkey
    + the coordinator setup, including: an OS (td class U3 and or V30 or faster)
    + the coordinator setup, including: coordinator & ceremony tool binaries

- **each participant** to procure a "Revault Kit" or its content separately:
  - printed instructions (this can be tampered with - compare and verify with digital doc!)
  - at least 5 unmarked 6-sided dice (or casino grade dice, or equivalent). More dice is better in case they are not proven balanced.
  - backup media. The Revault kit includes both
    - Metal backups (at least 2 for the main secret + 2 for emergency key. The more the better.) + numbered stamps (durability and readability)
    - Paper backups for seed words (recovery convenience), archival grade and preferably water resistant
    - a transparent pen with archival grade ink
  - 10+ unique tamper-evident envelopes per participant (or better tamper-evident seals)
  - "privacy booths", such as cardboard screen to prevent other participants to watch what is being rolled/written
  - screwdrivers (for opening the laptops)
  - thick tape to mask cameras

- a minimalist room with a basic table and basic chairs, where you can make sure there is no camera nor microphone. Thick curtains or no windows.

- QUESTION: who will procure the transport device to tale emergency address info into the ceremony?

### Private Keys, Public Keys, Miniscript Descriptors

At the beginning of the ceremony:

- there will be an arbitrary partial (missing participant xpubs) miniscript descriptor (to use in generating the emergency address) on the following devices:
    + at least one transport device per stakeholder that stores the partial emergency address, EA'.
- there will be a transport device with the coordinator pubkey on for each participant (prepared by coordinator admin ahead of time)

At the end of the ceremony:

- there will be unique 256-bit entropy as a mnemonic seed on
    + all b in B
- there will be unique bitcoin extended private keys on the following devices:
    + all td in TD that map to watchtower targets
    + all sd in SD
- the set of all miniscript descriptors (_except_ the emergency descriptor) for the deployment will be on the following devices:
    + all td in TD that map to stakeholder, manager, or watchtower targets
    + all sd in SD
- the emergency miniscript descriptor will be on the following devices:
    + all td in TD that map to stakeholder or watchtower targets
    + all sd in SD that map to stakeholder targets
- there will be unique noise private keys on the following devices:
    + all td in TD
- there will be the cordinator's noise public key on the following devices:
    + all td in TD that map to stakeholder and manager targets
- there will be the noise public key of the participants set on the following devices:
    + all td in TD that map to coordinator targets
- the noise public key for each stakeholder will be on the td in TD which maps to their watchtower
- the noise public key for each watchtower will be on the td in TD which maps to their stakeholder

Any td in TD that is not listed above should be destroyed after the ceremony, along with all laptops used to manage sensitive key material. 

### Pre-ceremony work for all stakeholders (may be assisted by Revault team)

- unseal the faster td card
- label it as "OS - _my name_"
- download a Linux operating system with live environment
- verify the signatures/hashes of the downloaded ISO
- "burn" the ISO to td

- unseal another td
- label it as "Software - _my name_"
- download the ceremony tool binary 
- download the revault binary
- download bitcoin binary
- download the watchtower binary
- verify the signatures/hashes of each binary
- copy the binaries to the td

- check both tds again on a trusted laptop (the Revault team can assist)

- Unseal remaining tds and label as:
  - "Revault Device - _my name_" 
  - "Noise Pubkey - _my name_"
  - "xpub - _my name_"
  - "Watchtower - _my name_" 
  
- put the SD cards in read-only mode (from the physical LOCK tab) if available.
- install the revault binary on their existing work computer (if not using a dedicated laptop for Revault)
  - :warning: if full archive node, need to have a ton of free space >>500GB for long-term usage
  - user will face the Ceremony screen, nothing to do at this stage. Sync in background until done.

- stakeholders should get the partial emergency descriptor ready on a td

### Pre-ceremony work for all managers (may be assisted by Revault team)

- unseal the faster td card
- label it as "OS - _my name_"
- download a Linux operating system with live environment
- verify the signatures/hashes of the downloaded ISO
- "burn" the ISO to td

- unseal another td
- label it as "Software - _my name_"
- download the ceremony tool binary 
- download the revault binary
- download bitcoin binary
- verify the signatures/hashes of each binary
- copy the binaries to the td

- check both tds again on a trusted laptop (the Revault team can assist)
  
- Unseal remaining tds and label as:
  - "Revault Device - _my name_" 
  - "Noise Pubkey - _my name_"
  - "xpub - _my name_"

- put the SD cards in read-only mode (from the physical LOCK tab) if available.
- install the revault binary on their existing work computer (if not using a dedicated laptop for Revault)
  - :warning: if full archive node, need to have a ton of free space >>500GB for long-term usage
  - user will face the Ceremony screen, nothing to do at this stage. Sync in background until done.

### Pre ceremony work for the coordinator admin (may be assisted by Revault team)

- unseal the faster td card
- label it as "OS - _my name_"
- download a Linux operating system with live environment
- verify the signatures/hashes of the downloaded ISO
- "burn" the ISO to td
- set up coordinator
    + boot server with OS td
    + download coordinator binary on machine
    + verify signature/hash of binary
    + generate a noise privkey and pubkey
    + copy noise pubkey to one td per participant and label as "coordinator noise pubkey"


## Questions

- What are the requirements for different types of tds (those that *just* cointain xpub or noise pubkey, and those that contain ISO or binaries)?
- Do we make full size SD Card mandatory for write lock switch?
- What about other types of deployment (e.g. no self-hosted watchtowers or with some co-signers)?
- What do participants need to back up for the emergency? 

- Do we want a sys admin to generate coordinator keys, and to fill its td during the ceremony? Or can another participant be trusted with this? How does the coordinator recieve all the noise pubkeys?
- Should the coordinator admin be in the ceremony at all?
    + alternatively, ceremony could output all noise pubkeys of participants and wts and hand to coordinator admin
    + coordinator pubkey can be brought in advance
    + unnecessary risk to bring admin into ceremony and unnecessary for their keys to be generated this way since they are kept on hot machine anyway. 
    + Long-term the secure communication will exclude the coordinator anyway. 

- Who will bring the partial emergency script to the ceremony? 
- Should participants set up revault binary before ceremony? (that means verifying it, when they should verify in ceremony too...)

- What verification steps are necessary for each piece of data?
- What assumptions are made about transport devices that come into the ceremony from outside?

- What about the set-up? and the time between the set-up? 
    + participants need time to securely store their backups (perhaps a week or more)
    
- Should the mnemonic seed be generated once for both vault & emergency backups? Should they both be derived from the same mnemonic? Would using a tool derive these introduce unnecessary risk the process?
    + Want to avoid poor UX of creating two separate mnemonic seeds if possible, without introducing risk
- for metal backups, do users select from pre-etched tiles? 
- Should the instruction set be agnostic to choice of back-up material?

- What should be used for tamper-evident seals, and what should be sealed, and when?
- What about forensic trail?
- What about notary/ lawyers?

- What testing/ verification should be done at the end of the ceremony?

    