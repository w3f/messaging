# Protocol requirements

The purpose of an anonymous communication (AC) protocol is to ensure metadata
isn't leaked when messages are communicated between peers. It deals with how
messages are transported, and not what is in them.

An overarching principle is to keep things as simple as possible, and try to
factor out subproblems to separate layers as much as possible. When in doubt,
leave it out.

Metadata that we want to protect are:

**1. Sender Anonymity** (who sent a message?)

**2. Receiver Anonymity** (who read a message?)

**3. Sender-Receiver Unlinkability** (who is talking to whom?)

Primary consideration in threat model is:

- Global Passive Adversary resistant (GPA, insight into whole network)

Some consideration should also be given to a global active attacker (GAA), in
additional to local/remote attacks. For example, it might be reasonable to
detect GAA, but not be fully GAA resistant. A more complete threat model with
respect to capabilities should be provided, see the 
[Briar](https://blog.grobox.de/2016/briar-next-step-of-the-crypto-messenger-evolution/)
[threat model](https://code.briarproject.org/briar/briar/wikis/threat-model)
for example.

Participation Anonymity is less important. This is most likely to be a factor in
terms of Censorship Resistance (deep packet inspection, traffic morphing), which
is important but currently not of primary concern for this specific protocol.
This may change depending on protocol layering.

The Anonymity Trilemma [XXX] states that there's a fundamental trade-off between
Strong Anonymity on one hand and Low Latency / Low Bandwidth Overhead on the
other. Additionally there's a forth dimension, User Distribution, whereby an
increased number of users in the system increases the anonymity set.

This design consideration, in addition to general user adoption concerns, has
two implications:

**4. Reasonable Latency** (<5s, to allow for IM [XXX])

**5. Reasonable Bandwidth** (not specified, mobile data plan in undeveloped countries)

Since The User of the protocol could equally be someone with a limited data plan
as someone publishing sensitive information under a nom de plume, it is
desirable that the protocol accommodates the following property:

**6. Adaptable Anonymity** (adjustable resource consumption)

One could imagine a message being sent/received based on three parameters:
chosen anonymity set, latency and bandwidth. The feasability of this approach is
untested, and may differ for sender, receiver, and unlinkability. As an example,
PSS provides a form of sliding Receiver Anonymity scale using partial
addressing.

This might be a misguided notion,
see
[Signal's design philosophy, point 1 and 3](https://github.com/signalapp/Signal-Android/blob/master/CONTRIBUTING.md#development-ideology).

Additionally, the protocol should be:

**7. Scalable** (up to, say, ~1M active nodes)

Resource consumption should grow gracefully with the number of users. I.e. the
guarantees around latency, bandwidth etc should be provided even up to ~1M
nodes. Counterexamples: naive DC-nets and Whisper, which both exhibit quadratic
scaling behavior. Another counter example are mixnetworks with latency on the
order of minutes as you get up to to a sizable network size.

There may be more specific throughput requirements, but this underspecified for
now.

Finally, this should be a peer-to-peer decentralized protocol:

**8. No Specialized Services** (pure p2p)

Any node should be able to do any job in the system. A specific node may choose
to only operate with a subset of capabilities (say, for resource consumption
reasons), but this is up to that node. Example: light node operation, taking
advantage of Desktop/server different performance profile compared to mobile
(intermittent connectivity, bandwidth).
