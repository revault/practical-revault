# Networking

Each machine (co-signing server, watchtower, wallet client) requires the capacity to connect to the others over TCP and to engage in a noise_kk (or noise_xk) handshake (using pre-shared static public keys) to establish an authenticated and encrypted channel. The connections between wallet clients and watchtowers will be mediated by the sync server. All communications will happen over tor. 

> (TODO: define the processes for configuring tor, running an onion service, and connecting to onion services through SOCKS5 proxy.  

## API

A library that any component can use to participate securely in network communications requires an API such as the following:

`listen(onion_address, port)` - Start listening for TCP connections on ip_address:port

`connect(onion_address, port)`   - Connect to ip_address:port

These sockets/streams should be managed asyncronously to allow for handling multiple connections. Details of connection attempts should be logged.

`ceremony_input(public_key, identifier)`  - Ceremony data input interface (static public keys for use in handshake)

`handshake_initiate(ephemeral keypair, remote_static_public_key)`   - initiate handshake

`handshake_respond(ephemeral keypair, remote_static_public_key)`   - respond to handshake initiation
                
The handshake ensures authenticated and encrypted communication (assuming trust in ceremony).

## Usage

During the ceremony, secure communications among all required components should be set up. Users generate their static (pub,priv) key pairs (using their dongle/HM). Users will transfer (pubkey, identifier) information from their dongle/HM to the machines they need to connect to. Each machine will construct their key-value store for the other machines they need to connect to.

To properly guard against malicious insiders, the following steps are performed after the ceremony, where server operators are in distinct physical locations ready to operate their machines. This process should be co-ordinated out-of-band (e.g. by phone). 

1. Servers begin listening on a pre-determined address and port.

2. Clients connect to the server and initiate the handshake. This requires access to their dongle/HM with the private key, which will be used to perform Diffie Hellman calculations. 

3. Server responds to handshake. This too requires access to their dongle/HM with the private key for DH.
