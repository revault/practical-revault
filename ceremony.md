# Ceremony

An out-of-band provisioning of cryptographic keys is required to bootstrap cryptographic security among participants of re-vault. Countermeasures should be in place to minimize the risks associated with physical, network, and social engineering attacks that aim to compromise this process. Ideally this ceremony would not be too cumbersome for the humans involved and the scope for human error should be minimized.

## Design Questions

- Should the preparation phase be considered separately from the ceremony? (preparation includes: specifying the architectural configuration, device aquisition, software installation for each device, blockchain download for all full nodes, and deploying the sync server, watchtowers and co-signing servers.)
- Participants generate master (public, private) key pairs for their wallets and for (authenticated and/or encrypted?) communication. These must be backed up for disaster recovery processes. Is this part of the ceremony? 
> If each participant individually secures their own back-ups, then no, this can be done in advance. If a social recovery method is used among re-vault participants (as with [tatoshi wallet](https://tatoshi.io/)) then the ceremony is a good time to share encrypted back-up information.
- Are _authentication_ and _encryption_ requirements for communications between participants in order to collaborate in the daily routine for generating pre-signed transactions (unvault, cancel, emegency unvault, emergency)? 
> If communications are occuring over IP then encryption is a necessary protection from evesdropping entities who pose the risk of operational disruption by broadcasting one of the emergency transactions. 
- Participants must share their wallets' master public keys in order to construct vault addresses. Should this be a part of the ceremony? Is it preferable to have this communicated across encrypted channels _after_ the ceremony? 
> Sharing master public keys only needs to be done once per re-vault protocol instance (if a compromise disrupts operations then a new ceremony would be enacted to rotate from all old keys to a set of fresh keys).
