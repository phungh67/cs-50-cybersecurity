
#security #data

The controversial definitions between "security" and "privacy":
- privacy is a way of controlling information (sensitive) to be spread or to be used incorrectly.
- security is protecting something from unauthorized access.

It's more easy to image that: security is to build a fortress to keep all the precious things but privacy is to ensure that things should only go to the right place and received by the right person (or people). This different ways of definition come with several different methods to ensure these conditions.

# The foundation protection: Access Control List (ACL)

This is the oldest and most widely deployed mechanism in the computer security. At its core, access control answers only to the "privileged entities". 
It restricts unauthorized access to data by ensuring:
A typical subject can only access to a predefined set of objects with restricted privileges (read, write, execute, delete,...).

There are several models
- ABAC
- RBAC
Limitations:
- Confidentiality problem: cannot control how a privileged user do what actions to the dataaset.
- Integrity problem: cannot preserve the truth or correctness of the data source.

# Cryptography - protecting data in the transit.

This can ensure that only authorized and correct data is used (by using hashes or checksum). But it cannot guarantee the integrity after decoding the data.

# Information-Flow control: beyond access

This technique ensures the flow of data propagate within the system (or even the network).

# The Confinement problem

## Labels and Security Lattices:

Addition labels to ensure that only appropriate level can read the data, all lower levels cannot read even the data is delivered to them.
This leads to the Noninterference guarantees: If a projection  that take both high and low inputs to project from a higher level of security to lower level, the output must not be affected by the high result.

$$ [ \forall h_1, h_2 \in H, \; \forall l \in L, \quad \pi_{Low}(P(h_1, l)) = \pi_{Low}(P(h_2, l)) ] $$
where $( \pi_{Low} )$ is the projection of the program’s output onto **low-security outputs**.

With both h1 and h2 are the high input, with the same low input, the projection must be the same no matter how different between h1 and h2. Because with this guarantee, attacker only learns information about low-level data.
