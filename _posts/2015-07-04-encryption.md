---
layout: post
title:  "End-to-end encrypted messaging"
date:   2015-07-04 00:33:42
category: Ideas
---

## Description for an end-to-end encrypted datagram service supporting multicast

### Features
* low network overhead for the security offered
    * ~10 bytes for every group member
    * ~250bytes for an encrypted public key for multicast
    * ~250bytes for the signature (similar to SSL/TLS)
* supports multiple recipients without sender data replication
* anonymous data content (end-to-end encrypted)
* anonymous senders
* only the endpoints know the group compositions, none of the central servers or proxies do

### Introduction

#### Node overview
* S : Source, the device sending the message
* P1 : First proxy (randomly chosen by S)
* P2 : Second proxy (responsible for the message division, specified by S to P1)
* CS : All user devices have a connection to this node in order to receive messages
* T : Target, the destination of the message

#### Scrambling
1. Generate a temporary key
2. Encode the payload with the key
3. Prepend the key to the payload
4. Encrypt the key and payload combination with a supplied public key 

#### Setup:
* All devices keep a connection with the central server for receiving messages.
* All  devices not behind a NAT (ie. servers) without bandwidth limits (ie. mobile devices) act as proxies.
* Get data from the central server: its public key and a list of proxies with their public keys.
* Generate a local key pair for signing.

### Operations

#### Creating a group
1. generate a key pair locally
2. add yourself to your local recipient list, which contains the ids and public keys of each member

#### Adding a user
The user already in the group
1. prompt for a target id to add to the group
2. prompt for a passphrase
3. encode the group key pair and recipient list with the passphrase
4. send the message to the target id

The user being added
1. Sees that he has received a message for which none of his private keys work
2. prompt for a passphrase
3. if it works, add the recipient list and private key to its cache
4. for every recipient except itself send scrambled with private_key<sub>G</sub>
    * the updated recipient list
    * the new users public key

#### Sending a message
1. Contents of the initial message, which are sent from S to random proxy P1
    * The message, signed with the private_key<sub>S</sub>, encrypted with public_key<sub>G</sub>
    * S encrypted with public_key<sub>G</sub>
    * The recipients, encryped with the public_key<sub>CS</sub>
    * A specification for which P2 to use (random)
    * public_key<sub>G</sub> encoded with public_key<sub>P2</sub>
2. P1 forwards the message to P2, contents:
    * The message encrypted with public_key<sub>G</sub>
    * S encrypted with public_key<sub>G</sub>
    * The recipients, encryped with public_key<sub>CS</sub>
    * public_key<sub>G</sub> encoded with public_key<sub>P2</sub>
3. P2 splits the message into one for every recipient, contents of a single message:
    * The message encrypted with public_key<sub>G</sub>
    * One of the recipients, still encrypted with public_key<sub>CS</sub>
4. P2 scrambles the payload of every message using the groups public key (which gets decoded with the private key of P2)
5. P2 scrambles the entire message with the public_key<sub>CS</sub>
6. P2 sends every message to CS (possibly through a randomized additional proxy if there is low traffic to avoid traffic pattern detection)
7. CS descrambles and then decodes the destition of all messages and sends them to the different T
8. T decodes the message using private_key<sub>G</sub> and descrambles (step 4), decodes again (step 1) and obtains the cleartext.
9. T verifies whether the signature is correct and passes on the message to the receiving application

### Who knows what?
#### S
* Message contents
* All recipients
* Message signature
* Choice of P1 and P2

#### P1
* S
* Choice of P1 and P2

#### P2
* Choice of P1 and P2
* Group public key

#### CS
* Choice of P2
* A single recipient

#### T
* Message contents
* Message origin
* Message signature


### Possible problems:
* Source bandwidth increases with group size (if every group member would use their own key pair for encryption)
    * Fixed by adding the shared group key pair
* Keeping participants anonymous whilst verifying their authenticity 
    * Fixed by the requirement for a passphrase when adding people (which has to be distributed over another secure channel)
* Sending data to anonymous receivers 
    * No need for anonymous receivers due unknown message contents and anonymous transmitters
* Latency due to a triple proxy (P1, P2, CS)
    * can be shortened to a double proxy by sacrificing multicast possibilities (eliminating P2)
    * But the double proxy still ensures anonymity
* If you have control over the paths between P1, P2 and CS you can trace the path of the packet.
    * Their input and output content at P2 is entirely scrambled and combined with a high traffic level and a short, randomized wait, this prevents traffic flow analysis.
* If you have control over all servers you can trace the path of the packet
    * Indeed. As long as not all the servers get hijacked the system is still anonymous though and due to the random choice of P1 and P2.