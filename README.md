# A Decentralised Privacy-Preserving Communication Protocol

> Messaging for Web3.

## Motivation

We need communications protocols that protect metadata, partially as an ethical end goal and partially for business reasons, but also as a fundamental building block of decentralised applications.  

We cannot control all information asymmetries, especially the metadata observed by large infrastructure providers.  Yet, information asymmetry or symmetry are an extremely important assumption in economics models.  We thus consider increasing our control over information asymmetry to be an essential building block, which makes privacy a crucial tool, including metadata protection. 

We have witnessed adoption of centralised messaging solutions being influenced by users' perceptions of privacy, at least at a personal level.  We thus believe providing real metadata protections should help establish a stable user base, which may help solidify or stabilise other services provided by the protocol, including associated financial services.

We also consider privacy to be a fundamental human right.  Article 12 of the Universal Declaration of Human Rights names "privacy" and "correspondence" for protection, which logically covers metadata.  




> No one shall be subjected to arbitrary interference with his privacy, family, home or correspondence, nor to attacks upon his honour and reputation. Everyone has the right to the protection of the law against such interference or attacks.


We also believe that user anonimity and stronger privacy protections aid adoption. 

If we lack metadata protections, then infrastructure providers will act as adversaries who collect and abuse user data in a myriad of ways, such as by adjusting prices, employing strategic voting, front running, etc.  

### Whisper


In the current decentralised application landscape, we are seeing projects
struggle to achieve mainstream adoption. We believe that the underlying
protocols do not have sufficient capability to enable the necessary level of
adoption. Part of the problem is that these applications often require the
exchange of transient messages; however, it does not make sense to transmit
these messages via a blockchain. Gavin Wood realised this problem in the early
days of Ethereum, and suggested that DApps would require a decentralised
messaging protocol that would provide this capability. This protocol is called
[Whisper](https://github.com/ethereum/wiki/wiki/Whisper).

Unfortunately, the evolution and adoption of Whisper has been stunted, which is
despite the rapid advancement of applications. The lack of development means
that Whisper is not able to provide the scaling that we need, nor is it feasible
for projects to create their own bespoke messaging protocol. What we really need
is a new protocol to handle transient messaging at scale. We believe this is a
necessary cornerstone for enabling the mainstream adoption of DApps.

## Project

We would like to gather a number of projects together to align and support this protocol:
- Researchers
   - Academia
   - Web 3 projects
- Protocol implementers
   - Core dev teams from Web 3 space
- Application builders
  - User messaging application
  - State channels
  - Latency-agnostic streaming protocols
  - Other applications requiring transient messaging
  
**The goal is to end up with at least one viable implementation, spec and a theoretical
analysis of the protocol properties.**

The idea is to start with gathering requirements and aligning on goals, the Web3
Foundation (W3F) has discussed with a number of projects to better understand
what the needs are. W3F has also started to do an initial exploration of
potential components to be used in the protocol.

### Plan (to be evolved)

1. Refine this document to reflect the motivation, requirements for the
project and initial mapping of the space until initial contributors are happy with it.
2. W3F to organise a workshop with all the relevant parties
3. Come up with clear work packages to be done by all contributors.
4. Readjust the plan together with the project contributors.
5. Achieve the goal.

### Project contributors (in alphabetical order)
- [Status](https://status.im)
- [Validity Labs](https://validitylabs.org)
- [Web3 Foundation](https://web3.foundation)


To contribute your project will need to commit some time to help specifying the motivation, requirements and mapping of the solution space. Following that contributors will be invited to participate in a joint workshop.

## Role of the protocol

| Layer | Purpose | Examples |
| ----- | ------- | -------- |
| Application | Application logic | Chat app |
| Storage / Sync | Sync data, make messages persistent | |
| -> **Protocol**, **DHT** <- | Scalable, decentralised metadata protection | |
| P2P | Overlay routing, NAT traversal | [libp2p](https://libp2p.io), [WebRTC](https://webrtc.org) |
| Network | Underlay routing | [TCP / IP](https://en.wikipedia.org/wiki/Internet_protocol_suite) |

## Adversary model
For the adversary model, see [a detailed description](./ADVERSARY.md)

## Protocol requirements
For a more detailed description, see [a detailed description](./REQUIREMENTS.md) of the below listed requirements.

Metadata protection:

**1. Sender Anonymity** (who sent a message?)

**2. Receiver Anonymity** (who read a message?)

**3. Sender-Receiver Unlinkability** (who is talking to whom?)

Convenience, Usability:

**4. Reasonable Latency** (<5s, to allow for IM [XXX])

**5. Reasonable Bandwidth** (not specified, mobile data plan in undeveloped countries)

**6. Adaptable Anonymity** (adjustable resource consumption)

Decentralization:

**7. Scalable** (up to, say, ~1M active nodes)

**8. No Specialized Services** (pure p2p)

Incentives to achieve mass adoption:

**9. Incentivisation for relayers** (not necessarily economical)


### Things that are explicitly out of scope
- Trust Establishment - provenance of long term keys to some known identity
- Conversational Security - authentication, confidentiality, integrity, perfect
  forward secrecy, accountability.

Additionally, see below for other things that may be out of scope at this layer.

## Questions

### What about Incentives?
The Protocol will be used in conjunction with Ethereum and similar technologies.
There's also a strong need for incentive-compatible designs. This means it is
useful to consider incentives and payment mechanisms as the protocol layer.
However, an ideal protocol suite should be layered and have a clear separation
of concern. This means there's a simple design with minimal dependencies. As
inspiration,
see
[Bittorrent economics paper (pdf)](http://users.eecs.northwestern.edu/~hq/papers/communities.pdf),
which is a separate protocol layer that people can choose to use or not to get
better quality of service (request and choking).

### What about Message Reliability?
The protocol should do Best Effort Delivery. Reliable Delivery can be provided
on top, similar to TCP/IP (or BSP/BTP for Briar), to accommodate things like:

- Guaranteed Message Delivery
- Message Ordering
- And possibly: Asynchronous Messaging (some protocols deal with it at this
  layer, so this may or may not be desirable)

Depending on the specifics of the reliability mechanism, throughput etc, this
may have consequences for the above Reasonable Latency requirement. The
Reasonable Latency requirement outlined above is for End to End messaging.

### How to deal with Network Spam?
- One approach is to use a Friend-to-Friend (F2F) network. This is what Briar
  and Secure Scuttlebutt (SSB) does.
- For open DHT-based, another approach is to rely on proof of work like Whisper.
  This isn't very practical for mobile / resource restricted devices, and
  appears to have limited usability.
- More approaches are likely possible, such as traditional rate limiting, basic
  peer reputation, payments, etc.
- Global network attacks more relevant here than a specific node. What does this
  imply for a DHT?

### How to deal with Asynchronous Messaging?
- One approach is to punt this problem to data sync layer.
- Another example is Briar requiring two entities to both be online
- Not clear that it is a necessary component of AC layer.
- One idea is to use Aggregation Points (Xolotl, lake mixnet) as providers
  similar to Loopix, but presumably with less HA guarantees.

## Initial work

###  Network
We would like to develop a protocol that leverages the best research and
protocols from the past in order to create something for the future of
decentralised applications. We want a “roadmap without potholes” for providing
stronger privacy assurances than Tor for both senders and receivers, while also
scaling well and providing short-term message storage for offline users. In
other words, we accept that a project this large requires a piecemeal approach
but we shall understand and avoid design dead ends that prevent either scaling
or rigorous analysis of anonymity properties.

We think mix networks occupy a sweet spot in which nodes run extremely efficiently but rigorous analysis remains possible, if challenging.  There are designs like Tor with more efficient designs, but they cannot provide rigorous anonymity properties.  There are also numerous academic schemes designed to support rigorous analysis, but at extreme sacrifices in efficiency.  We choose a Sphinx-like packet format because it’s efficient, adaptable enough, and has good security proofs.  

We leave actually analyzing our scheme for future work in collaboration with academics.  We shall however use Poison mixing and cover traffic strategy similar to Loopix, which appears analyzable although academic work to do so remains ongoing.  We concede that mix networks impose latency on users, but any faster design would definitely not admit rigorous analysis of anonymity properties, and could not credibly claim to be stronger than Tor.  

We shall tweak Sphinx slightly to accommodate both receiver anonymity and
short-term message storage simultaneously, which may require updating its
security proofs eventually. We consider short-term message storage essential to
user experience and advise against doing it via a second layer protocol.

We foresee the public key infrastructure (PKI) being the ultimate scaling
bottleneck for all existing anonymity scheme designs. We could avoid this with
gossip protocols but these enable epistemic attacks. We largely leave this to
future work, but suggest investigating a verifiable gossip based protocol. We
shall therefore use an insecure gossip based protocol initially with the hope
that it meshes best with later designs. If we used a more secure design inspired
by the Tor consensus then we might assumptions elsewhere that limit scalability
later.

We know rewards for node operators remains a contentious question in the
anonymity community with seemingly unforeseeable consequences. We nevertheless
think rewards represents our best hope for a network large enough to challenge
today’s centralized providers that operate on surveillance capitalism. We do not
imagine rewards obliviate the need to steward relay operator culture, possibly
quite the opposite.

### Messaging types
We’re focusing on one-to-one messaging for now.  We actually do require messaging layer crypto, even after all the mix net layers, so expect an Axolotl-like ratchet for this.  

We can adapt our short-term message storage plans for small group messaging, but
not with exactly the same privacy assurances. We leave designing this to future
work.

We think one-to-mass messaging should be done by using the mix network to send
to a broadcast protocol like Whisper v1 or perhaps a blockchain.

### Payment
We’re designing an accounting scheme to prevent abuse and reward nodes, without
damaging users’ anonymity. We’re currently working on several designs based on
fundamentally different methodologies, primarily payment channels, blind
signatures, and secret shopper, so as to more fairly evaluate them. We’re
keeping these designs as agnostic as possible to questions like if the users
actually pay anything ever.

### Implementation
In order to leverage existing work done in the space we would like to leverage
libp2p for networking and make sure that at least one implementation is fully
runnable in the browser leveraging Javascript and Wasm.

## Copyright

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
