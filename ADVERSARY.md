# Adversary model
As for every meaningful cryptographic construction, there should be a proper adversary model to define against which issues the protocol achieves protection. A formal adversary definition also allows to prove security properties formally.

## Informal adversary model
Since the protocol will be structured in several layers, the assumptions that are made are structured in the same way.

#### Network level
- The adversary may select arbitrary participants of the network and observe all of their outgoing and incoming messages.
- The adversary is unable to control all but its own nodes in the network.

#### Cryptography
- The adversary has no access quantum computers which means that classical complexity-theoretic hardness assumptions apply.
- The storage capacity of the adversary can be considered as unlimited, i. e. messages stay confidential only for a certain amount of time, e. g. 10 years.

#### Staking / SPAM protection
- The coin / time / computation resources of the adversary are limited. It costs the adversary a lot of effort / money / time to increase her attack power, so the more the adversary tries to spam the network, the more it becomes unprofitable her.

## Formal Adversary - coming soon
