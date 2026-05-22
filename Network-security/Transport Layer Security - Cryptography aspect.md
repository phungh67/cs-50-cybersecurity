`TLS/SSL` for short - check [[Transport layer security]] and [[Cybersecurity/cs-50-cybersecurity/Network-security/Cryptography|Cryptography]] first.


# Properties of security protocol

**Confidential** - Ensure that only specific receivers can read the data - done with symmetric cryptography.
**Integrity** - Ensure that the sent data was not tampered - done with hashed - signature,...
**Authenticity** - The data was not inserted, reordered, delayed (different with integrity - mess your data, in this case, just swap the sequence of all pieces, to trigger some hidden bug,...) - done with timestamp, nonce,...
**Mutual authentication** - using certification (to avoid fraud)
**Perfect forward secrecy** - if older keys were revealed, future key must not be guessed.

# Perfect forward secrecy

>[! Scenario]
>In a communication channel, encryption is vital, since it will protect the whole information flow. A master key was used, of course, but if only a single key was used, sometimes, it can be broken - cracked. So to enhance the protection, a session key - delivered from master key was used - for each message - so that the attacker cannot retrieve the whole conversation.

With a perfect forward secrecy:
- If a master key was compromised, all session keys derived from that key should be still working fine (util they are expired of course).
- If derived keys were compromised, the master key should still work.

Key should not be depend on a shared secret of any material used for authentication. For example the Diffle-Hellman exchange: even the two parties A and B has own secret but they both rely on the shared - public information g and p. Which resulted in a session key with $g^{ab} \mod p$ with a and b came from A and B (secret they picked)

# `SSL/TLS` 

- Designed to protect all types of `TCP` connections.
- Using port 443 instead of traditional 80 (for HTTP - HTTPS).
- `TLS` extensions can be included in the first Hello message.
- Client and server: client authenticate server when connection (to make sure you are communicating with correct server instead of fraud).


# The `TLS` record protocol

An application data will be split into pieces (that did not exceed a length of  16kByte at most).
Then maybe these pieces would be compressed (but in newer version, `v1.3`, this step was dropped).
Add Message Authenticated Control - which is a hash with the write key, sequence number, protocol, version, length and message all appended together.
Then the message + hashed information will be encrypted
A header `SSL Record Header` will be added before sending.

# Architecture

## Change cipher spec protocol

One byte message.
Pending state will become current state - change the encryption to what have been decided earlier: algorithms, keys,...

## Alert protocol

Two bytes, first byte is the severity level, second is the alert code
There are 2 levels: warn and fatal - transmission must be stopped with fatal level.
Alert codes: 12 for `SSL` and as many as possible for `TLS`.

## Handshake protocol

Specify
- Authentication method.
- Data encryption algorithm.
- Data protection algorithm.
- Performs key exchange.
![[Pasted image 20260522191440.png]]

![[Pasted image 20260522191552.png]]

# Summary

`TLS` is a secure protocol. in which:
- Developed from `SSL` - which is now depreciated.
- Latest version is `1.3` with may bugs and suffers many attacks.

Security protocols have some common properties:
- Negotiate the algorithm and the ciphers to use.
- Always derive new key material.
- Only use private key for authentication - enforce perfect forward secrecy.
- Change keys regularly.

Random number and timestamp are used as strong ciphers essential (nonce - number at once).

Attackers will not try to break completely (very impossible), but:
- Heartbleed - send messages with incorrect headers - mess our communication link up.
- Logjam: Man-in-the-middle degrades ciphers to make it crackable.

