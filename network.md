# Network

This specification describes security-critical design choices for how machines in a revault architecture connect to eachother. It should be clear when and why confidentiality and authentication are required. The general philosophy is to avoid over-use of cryptography where it isn't necessary, but to mitigate risks that affect a business' custodial and related operations. 

### Entity Types

The humans participating in an instance of the Revault protocol can have the role of Stakeholder and/ or Manager. Each Stakeholder will operate the following devices: portable device with revaultd, hardware wallet, co-signing server, and a watchtower. Managers will operate only a portable device with revaultd and a hardware wallet. The sync server will be operated by the organization using revault.

In a future version of the protocol the sync server and additional watchtowers may be outsourced to a service provider.

### Network Map 

> TODO: Insert Diagram Here 

This diagram is an example where there are 4 humans. 2 have the role of Stakeholder, 1 is both a Stakeholder and a Manager, and 1 is only a Manager. Trust boundaries are drawn to distinguish which hardware and software instances are operated by each human. Additional trust boundaries are drawn to show that certain physical boundaries are required for some of the hardware.

### Networking Layers

The design of the Revault network is logically separated into 4 layers:

| Layer          | Description                                                                                       |
|----------------|---------------------------------------------------------------------------------------------------|
| User Interface | Handles the communication between users and the Revault protocol                                  |
| Revault        | Handles application level semantics and authentication of messages passed across multiple machines|
| Noise          | Handles authenticated and encrypted channels between any two connected machines                   |
| Transport      | Handles TCP streams for connections between any two machines                                      |

### Noise Channels

Authenticated and encrypted channels will be established using noise_KK or noise_KX. The X/K notation states whether the initiator's (first token) or the reponder's (second token) static public key is known (K) by the counterparty or is transmitted (X) during the handshake. So, for example, noise_KX means that the initiator's static public key is known to the responder before the handshake.

Consider the flow of [messages](https://github.com/re-vault/practical-revault/blob/master/messages.md) for spending and for routine transaction signing. We use the following definition of directionality for communications, with initiators on the left and responders on the right. 

Stakeholder -> their Watchtower (Noise_KK)

Stakeholder -> Sync Server (Noise_KX)

Manager -> Cosigner (Noise_KX)

Manager -> Sync Server (Noise_KX)  

Watchtower -> Sync Server (Noise_KX)

This determines which static public keys need to be generated during the initialization ceremony and with which entities they must be shared (as part of configuration information). 

> TODO: write and link to ceremony.md

We use noise channels with Curve25519 for Diffie-Hellman functions, ChaChaPoly for cipher functions and SHA256 for hash functions.

### References

[Noise Protocol Framework](http://noiseprotocol.org/noise.html#application-responsibilities)

[Noise Explorer](https://noiseexplorer.com/patterns/KX/)


