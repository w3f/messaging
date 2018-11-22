# Motivation
In the current decentralised application landscape, we are seeing projects struggle to achieve mainstream adoption. We believe that the underlying protocols do not have sufficient capability to enable the necessary level of adoption. Part of the problem is that these applications often require the exchange of transient messages; however, it does not make sense to transmit these messages via a blockchain. Gavin Wood realised this problem in the early days of Ethereum, and suggested that DApps would require a decentralised messaging protocol that would provide this capability. This protocol is called Whisper.

Unfortunately, the evolution and adoption of Whisper has been stunted, which is despite the rapid advancement of applications. The lack of development means that Whisper is not able to provide the scaling that we need, nor is it feasible for projects to create their own bespoke messaging protocol. What we really need is a new protocol to handle transient messaging at scale. We believe this is a necessary cornerstone for enabling the mainstream adoption of DApps.

# Project

We would like to gather a number of projects together to align and support this protocol:
- Researchers
  - Academia
  - Web 3 projects
- Protocol implementers
  - Core dev teams from Web 3 space
- Application builders
  - User messaging application
  - State channels
  - Streaming protocols
  - Other applications requiring transient messaging

The idea is to start with gathering requirements and aligning on goals, the Web3 Foundation (W3F) has discussed with a number of projects to better understand what the needs are. W3F has also started to do an initial exploration of potential components to be used in the protocol.

**The goal is to end up with at least one viable implementation (based on requirements), specification and a theoretical analysis of its properties.**

## Plan (to be evolved)

1. Gather motivation in the Motivation section of this document.
2. Gather goals and initial promising directions in the Initial ideas section of this document.
3. Submit issues regarding the topics that still need to be worked on.
4. Once the initial project contributors are happy with the document W3F will organise a workshop to discuss the project with all relevant parties.
5. Clear work packages will be outlined and participating organisations will take responsibility to push them forward, these will include research and implementations.
6. Project contributors will adjust the plan as needed.
7. Goal is achieved.

# Initial ideas
## Network
We would like to develop a protocol that leverages the best research and protocols from the past in order to create something for the future of decentralised applications. We want a “roadmap without potholes” for providing stronger privacy assurances than Tor for both senders and receivers, while also scaling well and providing short-term message storage for offline users.  In other words, we accept that a project this large requires a piecemeal approach but we shall understand and avoid design dead ends that prevent either scaling or rigorous analysis of anonymity properties. 

We think mix networks occupy a sweet spot in which nodes run extremely efficiently but rigorous analysis remains possible, if challenging.  There are designs like Tor with more efficient designs, but they cannot provide rigorous anonymity properties.  There are also numerous academic schemes designed to support rigorous analysis, but at extreme sacrifices in efficiency.  We choose a Sphinx-like packet format because it’s efficient, adaptable enough, and has good security proofs.  

We leave actually analyzing our scheme for future work in collaboration with academics.  We shall however use Poison mixing and cover traffic strategy similar to Loopix, which appears analyzable although academic work to do so remains ongoing.  We concede that mix networks impose latency on users, but any faster design would definitely not admit rigorous analysis of anonymity properties, and could not credibly claim to be stronger than Tor.  

We shall tweak Sphinx slightly to accommodate both receiver anonymity and short-term message storage simultaneously, which may require updating its security proofs eventually.  We consider short-term message storage essential to user experience and advise against doing it via a second layer protocol. 

We foresee the public key infrastructure (PKI) being the ultimate scaling bottleneck for all existing anonymity scheme designs.  We could avoid this with gossip protocols but these enable epistemic attacks.  We largely leave this to future work, but suggest investigating a verifiable gossip based protocol.  We shall therefore use an insecure gossip based protocol initially with the hope that it meshes best with later designs.  If we used a more secure design inspired by the Tor consensus then we might assumptions elsewhere that limit scalability later.

We know rewards for node operators remains a contentious question in the anonymity community with seemingly unforeseeable consequences.  We nevertheless think rewards represents our best hope for a network large enough to challenge today’s centralized providers that operate on surveillance capitalism.  We do not imagine rewards obliviate the need to steward relay operator culture, possibly quite the opposite.
## Messaging types
We’re focusing on one-to-one messaging for now.  We actually do require messaging layer crypto, even after all the mix net layers, so expect an Axolotl-like ratchet for this.  

We can adapt our short-term message storage plans for small group messaging, but not with exactly the same privacy assurances.  We leave designing this to future work.

We think one-to-mass messaging should be done by using the mix network to send to a broadcast protocol like Whisper v1 or perhaps a blockchain.
## Payment
We’re designing an accounting scheme to prevent abuse and reward nodes, without damaging users’ anonymity.  We’re currently working on several designs based on fundamentally different methodologies, primarily payment channels, blind signatures, and secret shopper, so as to more fairly evaluate them.  We’re keeping these designs as agnostic as possible to questions like if the users actually pay anything ever.
Implementation
In order to leverage existing work done in the space we would like to leverage libp2p for networking and make sure that at least one implementation is fully runnable in the browser leveraging Javascript and Wasm.

# Project contributors
- Web3 Foundation
- Status
- Validity Labs
