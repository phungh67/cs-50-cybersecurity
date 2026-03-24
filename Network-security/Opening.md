
>[!Question]
>If we want to secure communication  between two systems, encryption is an important tool

Eavesdropping on a Dialog - a very common strategy for attacking, that led to information tampering (if the attacker has sufficient tools).
But sadly, encryption aims to confidentially, not integrity. The attacker can have the information, but cannot understand or extracting any meaningful information at all, but they can mess up the message.
Hashed can be used, that is, to guarantee that the message was not messed up during the sending process.

But that's not all. The messages can be replayed, reordered and also deleted.

So, in the summaries:
- Encryption is for confidentially.
- Fingerprint is for integrity.
- Sequence number is for the consistency (TCP is not trustworthy at all!).

But as mentioned, packets from old sessions can still be replayed. Because these reinforcements are only ensure that you are communicating with messages from a "person" but the person is currently talking with us could be other as well.

>[!Consider]
>We need a session concept, that is, guarantee the freshness and prevents insertion of old messages. And how to encrypt the message? If there is only one password, broken means every messages from the beginning were exposed, 100% not-so-secure at all.

