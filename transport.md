# Transport

Definition of transport used by communications happening in a version 0 Revault
deployment. All the entities of the network being known in advance, each of them has
a static public key exchanged during the deployment setup to then establish
[Noise](Noise_protocol) [KK](Noise_kk) authenticated and encrypted channels.

### Entity Types

The humans participating in an instance of the Revault protocol can have the role of Stakeholder and/ or Manager. Each Stakeholder will operate the following devices: portable device with revaultd, hardware wallet, co-signing server, and a watchtower. Managers will operate only a portable device with revaultd and a hardware wallet. The Coordinator will be operated by the organization using revault.

In a future version of the protocol the Coordinator and additional watchtowers may be outsourced to a service provider.

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

Authenticated and encrypted channels are established using [Noise KK](Noise_kk).
The `KK` notation states that both the initiator's and the responder's static public
keys are known to each other during the handshake.

Consider the flow of [messages](https://github.com/re-vault/practical-revault/blob/master/messages.md) for spending and for routine transaction signing. We use the following definition of directionality for communications, with initiators on the left and responders on the right. 

```
Stakeholder -> their Watchtower

Stakeholder -> Coordinator

Manager -> Cosigner

Manager -> Coordinator

Watchtower -> Coordinator
```

This determines with which entities the static public keys must be shared (as part of configuration information) during the initialization ceremony. 

We use noise channels with `Curve25519` for Diffie-Hellman functions, `ChaCha20-Poly1305`
for cipher functions and `SHA256` for hash functions.

[Noise_protocol]:http://noiseprotocol.org/noise.html#application-responsibilities
[Noise_kk]:https://noiseexplorer.com/patterns/KK/
