# Payment Scheme for Incentivizing Correct Mixing

### Aim:

Incentivize mixes to provide bandwidth for mixing without complicating the unlinkability security proof or requiring that users pay.

### Solution:

We shall assume a design roughly similar to Loopix [1], at least in cover traffic handling.  We want to add a payment scheme on top of Loopix to incentivize mixes to offer bandwidth, without requiring that users possess any particularly desirable resource.  We shall describe the solution arrived by a “process of elimination” explored in https://github.com/w3f/messaging/blob/master/paying/probabalistic.md

As in Loopix, our mixes send “loop” cover traffic messages that traverse each strata and return back to their sender.  As described in [1, 4.2.1], these loops provides defense against active attacks like n-1 attacks. The concept of loop cover traffic for mixes was first introduced by George Danezis in RGB mix networks [8]. In RGB mix networks, a mix sends anonymous cover traffic messages (called “heartbeat”) through the system back to itself “in order to establish its state of connectivity with the rest of the network” [8].

We have a limited number of mixes produce cover traffic using verifiable random functions.  After all messages are routed through the mixnet, we hold a lottery using collaborative randomness, in which cover traffic packets by several of these mixes’ win, and those mixes may reveal the packet.  We ensure the routes taken by these packets are hard to bias because the winning pool of mixes is limited, and they produce their packets using VRFs.  It follows that paying the mixes along the packet’s route should distribute payments fairly according to the route selection rules. 

We require that mixes provide a proof, from a commitment to their input/output, that they forwarded the winning cover traffic paket message.  If they do so, they are rewarded.  In the end, the more messages a mix is relaying the more chances it has to be rewarded, again subject to the route selection rules for loop cover packets.

In terms of anonymity, we impact the defenses against active attacks provided by the loop cover traffic, as well as protections afforded by total traffic.  We’d expect these active defenses might be fully utilized before the lottery, but warn that if mixes sample traffic differently using say Brahms then this represents a leakage.  We expect minimal reduction in total traffic.  We shall investigate using cover traffic originating from users as well, but this requires further analysis, and limiting eligible users remains tricky.


### Details:

We assume some fraction of mixes posses a threshold level of “stake” in some resource, probably a cyber-coin aka crypto-currency.  We require only that the number of sufficiently staked nodes be known, not that all mixes are staked, nor do we need the ability to slash for miss behavior.  All staked mixes commit to a verifiable random function (VRF) key $V = v G$.  If these nodes do not have a global view of the network then they must commit to their view of the network as well, but a global view sounds much simpler here``.  We might use BLS with proof-of-possession to aggregate these VRF proofs, but VEd25519 or NSEC5 work fine if only batching is required.  We suspect batching suffices because very few packets shall win the lottery.

We have a collaborative randomness beacon that sets epochs and releases a shared random value $r_i$ in each epoch $i$.  In each epoch, each mix $V$ with sufficient stake computes:
 - an epoch value $e_{V,i} := VRF_v(r_i)$, and
 - packet seeds $p_{V,i,j} := VRF_v(r_i ++ j)$.

Any mixes with enough stake should preferably send a cover packets created with a CSPRNG seeded by $p_{V,i,j}$ for each $j < j_\max$, and the global or committed network view.  We have a hard limit $j_\max$ on $j$ so the mix cannot “play the lottery” too many times.  An Erlang distribution with parameters lambda and $k := j_\max$ gives the sum of $j_\max$ exponential distributions with rate $\lambda$, so the Erlang CDF can tell us the probability a mix sends more than $j_\max$ messages in an epoch.  If we keep this probability low then mixes can rarely profit by sending extra cover messages, and increasing epoch length helps significantly here since the Erlang variance is $k/\lambda^2$.

As in Loopix, we send all traffic according to exponential distributions, so if we want all packet times to be correct then we can also sample the packet send time for $p_{V,i,j}$ uniformly from its own CSPRNG.  I suspect merely sending anytime within an epoch suffices because nodes should not benefit financially from altering the timing.  

_Problem_:  Do we need proofs that packets were sent at correct times?

Right now, I expect all nodes should commit to their replay protection database when an epoch ends.  We then in epoch $i+1$ run a lottery by seeding another CSPRNG with $e_{V,i+1}$ to test (a) if mix $V$ won epoch i, and (b) for which j they won.  If $V$ wins then $V$ optionally reveals $e_{V,i+1}$ and $p_{V,i,j}$, as well as proofs that its sampled its network view honestly in building the packet for $p_{V,i,j}$, which are trivial in the case of a global network view.  It publishes this winning notice and attempts to notify nodes named in the route taken by packet $p_{V,i,j}$.  Any nodes named in the route taken by of packet $p_{V,i,j}$ then publish the proof that $p_{V,i,j}$ appeared in their replay protection database.  If all nodes in the route $p_{V,i,j}$ took reveal their proofs, then they all get paid, along with $V$, likely nobody gets paid if any node equivocates.

We believe these replay protection database commitments prevent bribery attacks where staked mixes do not send cover traffic but merely commit to VRF keys and then bribe other mixes to claim they forwarded the winning packet.  There are good odds such protections may be overkill, except we might reward forwarding mixes far more than $V$ itself, which makes the bribery attack simple.  We do not reward any nodes if any nodes equivocate for the same reason.  We can limit replay protection database sizes too, but doing so requires overly reliable bandwidth measurements.

_Problem_:  Do we need replay protection database commitments?

We must rotate routing keys, but we envision routing keys living much longer than epochs, so actually doing this with the real replay protection database sound tricky.  If epochs are short, then we can simply have duplicate replay protection databases.  We envision replay protection databases being cuckoo filters, so that some cryptographic materials about each packet can be retained.  We leave this as an implementation detail.

We have described the protocol using rigid epochs, but relaxing these should make the protocol easier to build and debug.  If we do want replay protection commitments then we should choose a winning packet in epoch $i$ based on randomness in $i+2$ or later, thus giving time for sharing replay protection commitments.  We actually expect to use VDFs for the random beacon, in which case $i+2$ becomes much later.

We could easily give each packet the opportunity to win across a series of several epochs as well.  In principle, we could hold a lottery with each block added to a blockchain, so possibly only a few second.  We need epochal randomness $r_i$ to be longer lived however, but doing this appears essential elsewhere in polkadot too.


_Problem_:  Avoid epochs being too rigid and simplify the protocol


### Related work: 

**Validity Lab’s HOPR** [3] aims at ensuring correct routing by adding payment to the header of the messages that only can be redeemed if the next intended hop has received the message.  

We foresee several problem with their system:  All mixnet packet formats without security proofs were eventually broken, while their payment scheme dramatically complicates any security proof for the packet format.  All clients and mixes are disincentivized to send cover traffic, likely breaking anonymity in practice.  If client routing is responsive, then an adversary has incentives to carry out eclipse attacks.  We believe clients require a zero-knowledge payment channel like BOLT to their first hop, built on a zero-knowledge chain like ZCash, so that opening a payment channel does not deanonymize them.  Also, we do not foresee users possessing any tokens with which to pay.


**Lira** [2] aims at incentivizing Tor nodes to provide more bandwidth. The user adds coins to the message headers for each chosen Tor node. The question is whether these coins can be traced back to the users. There is a strong incentive to drop messages once they receive it (particularly the guard nodes), since this would make the user to spend more, no? 

**Proof-of-Work as Anonymous Micropayment: Rewarding a Tor Relay** [4]: Tor clients do not pay Tor relays with electronic cash directly but submit proof of work shares which the relays can re-submit to a cryptocurrency mining pool. Relays credit users who submit shares with tickets that can later be used to purchase improved service.

**From Onions to Shallots: Rewarding Tor Relays with TEARS** [5]: “TEARS audits relays and rewards them with anonymous coins called Shallots, proportionally to bandwidth contributed…” “...using protocols from distributed digital cryptocurrency systems like Bitcoin.”
https://apps.dtic.mil/dtic/tr/fulltext/u2/a623492.pdf

**A TorPath to TorCoin** [6]: “We propose an incentive scheme for Tor relying on two novel concepts. We introduce TorCoin, an ‘altcoin’ that uses the Bitcoin protocol to re- ward relays for contributing bandwidth. Relays ‘mine’ TorCoins, then sell them for cash on any existing altcoin exchange. “

**XPay**: Practical anonymous payments for Tor routing and other networked services: micropayment scheme for Tor communication and a trusted party (bank) [7]https://www.freehaven.net/anonbib/cache/wpes09-xpay.pdf




### References:
[1] The Loopix Anonymity System. Ania M. Piotrowska, Jamie Hayes, Tariq Elahi, Sebastian Meiser, George Danezis, USENIX Security 2018. 
[2] LIRA: Lightweight Incentivized Routing for Anonymity. Rob Jansen, Aaron Johnson, and Paul Syverson, 20th Symposium on Network and Distributed System Security (NDSS) 2013.
[3] HOPR mixing. Validity Labs, https://github.com/validitylabs/messagingProtocol.
[4] Proof-of-Work as Anonymous Micropayment: Rewarding a Tor Relay. Alex Biryukov and Ivan Pustogarov, Financial Cryptography and Data Security (FC) 2015.
[5] From Onions to Shallots: Rewarding Tor Relays with TEARS. Rob Jansen, Andrew Miller, Paul Syverson, and Bryan Ford, 7th Workshop on Hot Topics in Privacy Enhancing Technologies (HotPETs) 2014.
[6] A TorPath to TorCoin:Proof-of-Bandwidth Altcoins for Compensating Relays. Mainak Ghosh, Miles Richardson, Bryan Ford, Rob Jansen, 7th Workshop on Hot Topics in Privacy Enhancing Technologies (HotPETs) 2014.
[7] XPay: Practical anonymous payments for Tor routing and other networked services. Yao Chen , Radu Sion , Bogdan Carbunar, Workshop on Privacy in the Electronic Society (WPES) 2009.
[8] Heartbeat Traffic to Counter (n-1) Attacks. George Danezis and Len Sassaman, Workshop on Privacy in the Electronic Society (WPES) 2003.