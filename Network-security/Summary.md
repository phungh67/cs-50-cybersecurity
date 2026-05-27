## Steps by steps - between attackers and defenders

### Threat model: man-in-the-middle

For example, a communication link between 2 parties: A and B was intercepted by an adversary E (eavesdropper).

Between these 2 parties, many messages were sent and received.

### First brick:

If all the messages are in the clear text form, E can read, and even understand all the convesation with no difficulties at all.

The first mitigation: Cryptography.

**With encryption** in the communication message (just message, not the channel), we first achieve the **Confidentiality**.

In this scenario, attackers still can read the message but cannot understand, hence, only correct users can understand the message at all.

But, it leads to another type of attack: just messing up with the packets. It does not have to be correct, the attacker just need to insert something, tampered the message, so the on the receiver's side, the message becomes meaningless.

### Second step:

Question: how to prove that message was original?

We have fingerprint. A small piece of information, created by hash function (a function that always returns fixed-lenght data from input, can not be reversed). The message will be concatenated with a shared secret. Then hashed into a small fingerprint.

That little piece will be send along with the message, when the receive gets that message, a check will be performed, with same shared secret, with the message, compare with the fingerprint.


But, even with that, packets still are vulnerable to reorder, replay and delete attack.

Since the receiver does not know if the message was delievered correctly as it in the original sequence,...

### Third step:

Sequence number - an indicate that respects message's order. Wrong number means dropped/deleted.

But the attacker can just capture the whole pack, then try to win a race against legitmate user.

### Fourth step:

Authentication - a mechanism to prove that the sender is the correct user, message sent with freshness.

Some challenges will be provided, and if the client solved it successfully (by sending the response back, then the server will compare it will some trust base), the communication link is at least safe.

But nevertheless, many other concerns are still existed. Using which methods to protect the communication link, using which keys to encrypt the response, if two parties want to talk for the first time, which should be used...?

## What is network security?

Well, that is a very hard question, but let go with some types of network security solutions:

**Firewalls**: hardware of software systems that monitor and control incoming and outgoing network traffic based on predetermined security rules.

**Intrustion Detection and Prevention Systems**: monitor network traffic for suspicious activity and can take actions to block or mitigate threats.

**Virtual Private Networks**: create secure connections over public networks allowing remote users to access the networks safely.

**Network Access Control**: enforces security policies on devices trying to access the networks, ensuring only authorized and compliant devices can connect.

**Encryption**: protects data in transit to prevent unathorized access (confidentiality).

## Still, what is network security (but with threat models)

**Malware**: malicious software, including viruses, trojans, worms and ransomwares that can infect and spread via network.

**Phising**: social engineering attacks that designed to trick users into providing sensitive information alters communications between two parties (faked emails, faked websites,...).

**Man-in-the-middle**: cyberattacks - intercept the communication line between two parties. Since it sits in the middle, altering the messages is a common action.

**Distributed Denial of Service** just simply overwhelms the network with a bunch of traffic (very enormous volume) that caused denial of access for legitimate users.

**Unpatched Vulnerabilities** trigger hidden bugs, flaw logic that are not patched yet to gain unathorized access to network (or simply disrupt it).

## Mitigation strategies

**Segmentation**: Dividing the network into segments, isolated them with each others, only allow the communication if needed, as least paths as possible to limit the spread of attacks and protect sensitive data.

**Firewall and IDS**: filter the traffic, detect the abnormally ones and of course, monitor them - a pipeline to control and update the rules table.

**Security assessment**: regular audit, pentest to identity the vulnerabilities and mitigate the weakness frequently.

**Up-to-date system - trained users**: cannot trust the users, since they are (mostly) bad administrator of their own system. So, up to date software + good training.

>[! Define]
>So, after all, **Network Security** is to learn about all the security solutions - understand what are these tools, how to use them correctly. Also know about the threat models - potentially adversaries to the systems and finally, how to deal with them - mitigate and prevent.

