# Ceremony

An out-of-band provisioning of cryptographic keys is required to bootstrap cryptographic security among participants of re-vault. Countermeasures should be in place to minimize the risks associated with physical, network, and social engineering attacks that aim to compromise this process. Ideally this ceremony would not be too cumbersome for the humans involved and the scope for human error should be minimized.

## Objectives

Prior to the ceremony, it is assumed that the architectural configuration has been specified, the devices and machines have been aquired and the appropriate software has been installed (including blockchain download for all full nodes). 

1. Each non-manager gets the hidden service details (.onion) for the set of co-signing servers that are each operated by a manager.
 
2. Participants generate master (public, private) key pairs for their wallets. These must be backed up for disaster recovery processes.
> Securing back-ups will likely occur outside of the ceremony since each participant will do this independently. (Unless a variant of social recovery is employed among participants (as with [tatoshi wallet](https://tatoshi.io/)). 
3. Participants exchange master public keys. 

4. Participants generate (public, private) key pairs for (authenticated and encrypted) communication for distributed signing processes, as defined by BOLT8.

