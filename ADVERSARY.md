# Adversary model
As for every meaningful cryptographic construction, there should be a proper adversary model to define against which issues the protocol achieves protection. A formal adversary definition also allows to prove security properties formally.

## Informal adversary model
Since the protocol will be structured in several layers, the assumptions that are made are structured in the same way.

#### Traffic analysis
- The adversary may observe and analyse traffic among arbitrary participants of the network, both clients and relays.
- An adversary may compromise a numerous but limited relays and users for use in both passive and active attacks.

#### Cryptographic assumptions
- At present, the adversary has no access to quantum computers, meaning classical complexity-theoretic hardness assumptions apply for authentication and the public key infrastructure.  Also 16 byte MAC tags suffice in Sphinx.
- We cannot foresee an future adversary's access to quantum computers, so we should push for post-quantum protection on the forward-secure ratchet layer and the network link layer in libp2p.  At least one implementation should accomodate experiments with post-quantum key exchanges designed for mix networks, like CSIDH, which includes supporting an incentives layer secure against future quantum computers.

#### DoS / SPAM
- As we must accomodate cover traffic, we cannot necessarily limit the adversary's capacity to send messages financially.  We also artificially limit bandwidth for clients so SPAM represents a significant threat.  We should instead require authentication for one user to message another, but exceptions might exists, possibly using zero-knowledge payments to recipients.

#### Incentives
- We should ideally assume nodes act rationally over the very timeframes to exploit the incentives layer.
- We assume some relays but not all have locked stake.

## Formal Adversary - coming soon
