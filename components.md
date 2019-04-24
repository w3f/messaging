
# Messaging for Web 3 layers

We provide a high-level online of the layers involved in a mix network, proceeding roughly from lower to higher.


## 1. libp2p 

Used by relay operation, and at least startup relay PKI, if not all relay PKI

We envision using exclusively libp2p for our transport layer, preferring to adapt rust-libp2p when necessary.  We shall not limit ourselves by Protocol Labs' libp2p implementations or specifications.

We shall stick with TCP for libp2p's networking for foreseeable future, using either Noise or TLS 1.3, but maybe QUIC eventually.  In principle, one might hide traffic better using TLS, but this requires extensive work.  I'd favor Noise to simplify three probable future modifications, like:

- adding a post-quantum ephemeral handshake, 
- adding a forward secure ratchet, and
- running the key exchange in trusted hardware ike SGX.

I'd favor some module-LWE key exchange like Kyber for the post-quantum ephemeral handshake, mostly becuse they fit this role well, but also partially because isogenies-based key exchanges are preferable at the mix packet format layer.  I think post-quantum authentication sounds premature, but some forward secure ratchet designs help somewhat here.  We might add Tor pluggable transport support to libp2p eventually.

We've concerns about all nodes in one strata reaching all nodes in the next strata, due to each packet taking a different route, but we're unsure if any real problem exists.  Also, both a stratified topology with restricted routes, and QUIC, should improve this somewhat. 


## 2. Mixnet

Any layers critical to operating the mixnet itself.  We include incentive extensions here because they integrate tightly with mixnet components.

Roughly, these mixnet layers use only libp2p below, and some blockchain, while the interface layers above interact with them through the mix client and relay PKI. 


### 2a. Relay PKI 

Uses libp2p at least for nodes starting up, and maybe mix relays (Robert's scheme).
Used by mix packet format (creation) and relay operation.  Also likely used by message storage-and-forward layer for selecting storage nodes.  

Initially, we simply register all nodes on some blockchain, but registration should be free so we require some mechanism to register an "intention" and have that intention validated. 

Research:

- Off-chain registration, pseudo-gossip, etc.
- Explain using VRFs to canonically sort relays into strata or strata with restricted routes (Jeff & Fatemeh).
- Index-based encryption PKI compression MPC (Jeff & Fatemeh).  Issues:  Actually explain any analyse how it work!  Is BLS12-381 going to get constant-time pairings? (TODO: zcash goals link)  Can/should this be combined with CSIDH but with slower key updates?  How fast should keys be updated anyways?

#### Relay capacity estimation game

Relay PKI extension, but makes additional usage of libp2p and/or mix relays.  Impacts inflationary incentives built into cover traffic heavily.

If we require capacity estimation early then centralised authorities could play this role initially, like in Tor.  We should analyse using "involuntarily bonded wealth" for this too, under the inflationary cover traffic driven incentive scheme.  In other words, if incentive payouts begin bonded for sampling, but automatically unbond with some decay function, then they represent recent bandwidth contributions, which might suffice.

Research:

- Explore other VRF-based sampling games.
- Approach for message storage.  Ideas worth understanding:
  - How far can capacity claims be trusted?
  - Are VRF games even sensible for ephemeral storage?
  - Are recommendations damaging to privacy? 
  - Reputation schemes?
  - Involuntarily bonded wealth schemes?


### 2b. Mixing

#### Mix relay

Uses libp2p heavily and uses relay PKI to locate next hop.  Implements/calls mix packet format processing.
Invoked for sending by relay cover traffic, but otherwise runs stand alone.

We enforce message delays by deterministically sampling them here.  

Future work:

- Optimize replay protection database
- Improve forward security protections for operators by running in SGX or similar.


#### Mix client

Uses libp2p heavily.  Implements/calls mix packet creation.
Invoked for sending by cover traffic, SURB distribution, and direct messaging.

We create and manage SURBs here including tracking the SURB unwinding database. 


#### Mix packet format (library)

Uses relay PKI in packet creation.
Used as packet processing by relay operation for processing packets.  Used as packet creation by SURB distribution, message storage-and-forward, and and maybe direct messaging

Future research:

- Write modern security proof using Rogaway's oracle silencing trick.
- Adapt security proof to support SURB logs.
- Experiment with post-quantum onion encryption using CSIDH. 


#### Payment channel incentives (library) (Robert's design)

Extension to mix packet format. 

Eee: ???

Jeff's notes:  At Web3, we consider this slightly heavy cryptographically, although not as heavy as other desirable features, like CSIDH.  It's largest problem is dis-incentivising cover traffic, but it sounds useful for supporting a secondary large packet size however because we might not want any cover traffic for a larger packet size anyways.  And such a secondary larger packet size helps us create an economy. 

Research:

- Do security proof (Stephanie Roos, Jeff, etc.).  If found insecure for privacy, then correct by pushing economic security onto SGX assumptions.
- Tooling overlap with other payment channels work?
- Compatibile with CSIDH and/or index-based encryption PKI compression MPC?


### 2c. Cover traffic

Uses relay PKI and mix relay/client.
Afaik used by nothing, so potential orthogonal to higher layers

We take inspiration from Loopix by adopting roughly its cover traffic categories and sampling delays from Poisson/Exponential distributions.

Research:

- Ania Piotrowska's simulation based security work (done/on-going)
- Any non-simulation based security work (future)
- Any work on cover for less strict message sending times or densities (future)

#### Inflationary incentives (Jeff's design)

Extension to cover traffic.  Influences capacity estimation heavily built into relay PKI.

See incentives.md

Research:

- Write paper for feedback


## 3. Mix interfaces

Semi-required layers that make the mix network usable.


### 3a. [MLS](https://messaginglayersecurity.rocks/)

Uses mix client.
Used by message queue.

All messages passing crossover points, aka using SURBs, require a higher level encryption that protect them from the crossover point, which include message storage nodes.  Axolotl suffices fine for simple one-to-one messages, but adopting MLS should better support multi-device and small group messaging. 

Research:

- Ensure MLS supports serverless operation cleanly.
  - Federation maybe works okay now?
  - Inria interested but concurrency seems hard
- Add post-quantum key exchanges, likely module-LWE ala Kyber.

Upcoming events:

- Hackfest in Berlin on 16 May https://mailarchive.ietf.org/arch/msg/mls/j86jCFQULbztCjUKzCrMMA76j0k


### 3b. Message queue

Uses MLS and mix client.  Invoked for sending by cover traffic component probably.
Used by MLS and ...

An initial design should be rather simple, but overall this component looks quite complex, and should be subdivided.

Enqueues messages per recipient, including distinct speed and reliability quality-of-service paramaters.  Erasure codes (RaptorQ) multiple messages into one real mix message according to quality-of-service paramater.  Provides most useful messages when a real message slot is advertised by the cover traffic component.  Larger messages are required to be lower priority.

We must decided if the data sync layer above takes ultimately responsible for reliability, and merely aborts unsent retries from the queue, or if and when the message queue itself takes responsible for reliability.

As an example, if Alice sends Bob a small message marked as very important, then the queue shall resend it until Bob acknowledges, using exponential backoff but likely with a high initial backoff.  Into these repeated messages any other lower priority messages gets packed, including:

 - Fresh direct reply SURBs
 - Any other higher priority message Alice sends to 
 - Acknowledgements
 - Erasure coded chunks of any large messages Alice sent to Bob
 - Anything lower priority that another component like MLS wants shared


### 3c. SURB distribution

All use mix client for SURB creation, but provide SURBs to the message queue.

#### Direct reply SURBs

Invoked by message queue.  

Provides fresh direct reply SURBs upon request by message queue.  .

#### Message storage SURB stocking

Invoked by either mix client or storage client.

Provides the user's message storage servers with fresh SURBs.


### 3d. Message storage-and-forwarding

Jeff calls these aggregation points in earlier write ups https://github.com/burdges/lake/blob/master/Xolotl/papers/Xolotl-3-mailboxes.tex

We need a mailbox type identifier that permits us to iterate on the design.

#### Authentication libraries

Used by message storage-and-forwarding layer.

We recommend implementing VLR group signatures for most applications.  Any signature scheme suffices for initial deployment though.  And zero-knowledge blockchain payments or bonds work too.  We thus highly recommend modularity here.

In future, we might like more "democratic" options than pure VLR group signatures.  As an example, VLR group signatures could possibly support threshold issuing and revocation, and even threshold "rekeying" the master.  We shall seek collaboration with Bryan Ford's DEDIS group at EPFL for these.

#### Server

Registers as a mix net service with the mix relay.  Sends messages using SURBs via the mix client.  Implements message storage SURB stocking probably.

Tracks messages in "mailboxes" and batches of SURBs associated to mailboxes.  Avoid it knowing which batch corresponds to which user or device.  Instead batches are independent, but new batches specify old messages they no longer require, or preferably delete old messages.  

If possible, messages are identified by ciphertext hash, with exclusions or deletions identified perhaps by bloom filter.

#### Client

Uses mix client directly, or maybe uses message queue.

Registers with message storage-and-forwarding services.  Keeps them stoked with SURBs.


### 3e. Contacts and/or rooms

Contacts scheme shared by message storage-and-forwarding client, MLS, and the message queue, as well as the data sync layer above. 


## 4. Data sync

Uses message queue heavily.

Interfaces applications developers actually like and understand.  A priori, I'd handle reliability here since the data sync layer might utilize space more efficiently, due to understanding some relationships between messages parts to be combined by the message queue.

### 4a. Lower layer demo using pijul and/or git

As a demo, we might adapt pijul and/or git for sending pushes through the message storage layer.  Adapting git for non-manual usage tends not to go far, but pijul might prove more widely usable.

### 4b. .

Non-mixnet layers 
required layers that 

   - Status' analysis: https://github.com/status-im/bigbrother-specs/blob/master/data_sync/p2p-data-sync-comparison.md
   - Compacting message and reinserting DAG
   - Appears largely orthogonal to lower layers, except users run data sync between themsevles, not with message stores, because sync should compact messages to best exploit the fixed size mixnet packets.  Relationship with MLS needs clarification.


###


We consider layers 1-5 to be def`enses against powerful internet adversaries, but we'd like layer 6+ to operate over more local networks, including mesh networks.

