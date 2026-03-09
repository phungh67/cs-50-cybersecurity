
# 1. Purpose of using Cryptography

In a scenario that 2 people want to communication with each other through a communication channel. But surely, these two want that if there is a change that some adversary tries to listening in-between (i.e tries to steal information by performing some eaveasdrop), that person cannot understand any piece of stolen packets at all.

We have CIA property: Confidentially, Integrity and Availability, which are:
- One desired destination (people, entity) can know that information, and anyone others than that should not be able to intercept it.
- Only authorized users are able to change the content of message.
- Once a message was sent through the communication link, it should be highly available that if a link was down, that message will still be delivered as nothing happen.

We also should consider that "trust and certification" problem, that the message must be sent from exact user that we are communicating with and the key (used to encrypt) the message is surely belong to that person as well.

# 2. Two distinct way of Cryptography

The basic one aims to protect the data in-transit, at-rest,...
The other is to create cryptic algorithm to protect that data (strength, complexity, ...)

But in the context of security course, **Cryptography** is a method to protect information during execution..

Nevertheless, with the increase of electronic devices, everyone needs cryptography to protect the data from being exploited.

>[!One time pad]
>A short-lived cipher text, totally secure, because only used to transport the key to receipent. We will need a random string, K (number) messages, and the `XOR` binary operator to create the encrypted message.

# 3. Type of (common) Cryptography 
## 3.1 Traditional symmetrical model

A common model with sender, receiver and the cryptanalyst (the one who tries to break the encryption system).

The idea is pretty simple, a key used in the encryption process and if the receiver want to decrypt the message, he/she must have the same key used in the encryption process.

This leads to a question: how to send the key through a secure channel? Because if not, of course the cryptanalyst will be able to steal that key and uses it to read all the messages?

Questions about the property of the communication channel? Need confidential/ authenticated channel for this purpose, only desired one can read and people in this communication link must know who sends these messages.

But **Why not use this channel for messages also** ?
- Limited bandwidth.
- Key << actual message (key can be 256 bits) but message can be KBs, MiB or even GiB.
- Key exchanging happened first, than can be used in the future (the channel only).

**Theoretical** vs **Practical**

In this system, we should consider about key management system and how the cryptanalyst can be "feasible" to crack this system.
We can assume that they have the cipher text (which looks meaningless) but also have some words, phrases of plaintext that are corresponding with some cipher text (so they can figure out the mechanism somehow). Because it is not necessary to know all the plaintext, they just needs several critical phrases to expose the secret behind.

**Key length** - can make the attacking more challenge, because the brute force aims to exhaustive try all possible key, longer the key, longer the time taken to figure it out.

**Confusion** and **Diffusion**
- Diffusion - if changed a single symbol in the plain text, many symbols in the cipher text would be affected (waterfall effect).
- Confusion - cipher text should be depended both on plain text and key in a complicated way (to make sure that attackers should not be able to map cipher and plain to crack our system).
As a result, in cryptography system, we have permutation (which creates result from add, subtract or doing something with the text in a manner that respects the sequence). And substitution (replacing blocks according to some formula). 

**A quick comparison with Asymmetrical** 

We cannot compare the key length because the Asymmetrical relies on computational complexity, while the symmetrical uses the permutation, substitution (based on key length).

>[! What have been solve?]
>The listeners could not be able to decrypt the messages if does not know the key. In general, confidentiality is guaranteed, but how about the integrity?

## 3.2. Asymmetric system - public keys system

The idea is you will have 2 keys: public and private key. You want to spread the public key as much as possible, for everyone to be able to send secure messages to you. But only you can decrypt these messages by using the other key in the pair: private key.

Advantage: no secure channel needed to be established first. But of course, there must be some requirements for this?
- Authentication - a certain key should belong to a certain person, to avoid the "key-trick-replacement".
- One-way: encrypt should be easy, but the reverse should be nearly impossible (trapdoor one-way-function).

The solution: ***signature*** for the key (to verify identity of the "sender")

>[!Signature system]
>First, ensure that cannot go backward, using the $K_{priv}$ to signing, and for checking, using $K_{pub}$. But since everyone can have that public key, a combination between both encryption and signing should be used. Take the example of 2 people A and B, the owner (A) will use the private key (that is, only A has) to sign the message first, for integrity. Then used the B's public key to encrypt the message. Then, upon receiving, B will use the B's private key for decrypting the message, then use A's public key to check signature to know if the message was tampered or not. This is a feasible scenario, everybody has public keys, but for signing, only private key was used, to make sure that the signature only from correct person.




# 3. Famous problem: Symmetrical and Asymmetrical, what is better and why?

Symmetrical uses only 1 key but requires a secure channel which is quite slow, expensive and must existed before communication process. The key must be confidential, no one excepts authorized personnel can read. The cipher text and plain text are using the same key to do some encrypt/ decrypt stuff.

Asymmetrical uses 2 keys and the public key can be spread as much as possible. It also requires some authenticated channel for public key. The public key is used to encrypted the message while private key is used to decrypt it. We also have a mechanism called signature to ensure integrity (the message originated from correct user and was not tampered during the sending - because if it was, we know)

# 4. Digital certificate

A certification binds a public key to a specific user. It consists three things:
- Public key
- User's information
- Digital signature of the attester (authority that stamped this thing).

Components of this system:
- PKI public key infrastructure (store, management,...)
- Certification authority (the stamp).
- Authority organization (RA)

The most important thing is "revocation" - if the key is lost or got stolen, you (the author of the key) must be able to revoke it (make it invalid)


# 5. Information hiding

Covert channel allows an inside malicious process to send sensitive data to an outside receiver, using an existing baseline communication band.
Contrary, steganography presents the communication in clear sight but in a form that is not likely to be noticed (a mechanism, a technique to hide the information but not by using cipher or something, but to give the information in the form of something that is not likely to be noticed or recognized at the first sight).
With the cryptography, the content is concealed but the existence of the encrypted data is visible to all.

# 6. Data remanence and side channel attack

Remanence - the remaining data after deletion in a computer system. There are several examples for this:
- On the magstripe cards of floopy disks: write heads are not perfectly aligned on different system, with specialized tools, one can recover the data.
- Ferromagnetic media: weak form of data remain even when overwritten happened (that is, data is still there, in some form, maybe not fully recoverable but still can be spot and extracted even you filled the disk with 0 sector).
- CMOS-flip-flop circuits: data can remain for a period of time with the low temperature.
- Modern OS: when you delete something, they are just marked with deletion flag, can be recover by advanced means (only complete gone if overwritten was carried out).

>[!Side channel attack] 
>Is a type of attack based on the information gained through the physical implementation of an embedded system. And to do so, one must have the accessibility to the hardware system. This attack is also based on side-channel information (extra information) retrieved from the device (i.e the execution time of some statement is a little longer that expected, maybe that statement is wrong, because the rest were correct, so faster than the wrong one, because the system must evaluate to find out it was wrong).

A common counter for this is `Encrypted tunnel`: so everything inside this channel is secure and without credential, you cannot figure what exactly they are exchanging through this channel. But well, since this is side-channel attack, the adversaries use the information that seems meaningless, but in fact, it is not: by analyzing the packet length, packet direction and packet timing, they still can figure out which pages, which destinations that user was heading to - it called: **Peak-a-boo I still see you** (and with 68% percent of correct)


In additionally, we know that a "smart card" - with a tiny protected chip on it, contact-less (that you cannot take the contact time to steal data), but with side channel, it is still feasible to analyze something:
- The power consumption of a processor based on the instruction it has to execute (longer command takes more energy).
- Closely monitor the clock cycle (to analyze the power consumption).
- And when a sweet spot (desired instruction was isolated), we can closely monitor again to find out the data it processed (energy ~ instruction && amount of processed data).
It is possible because the tiny chip on smart card is designed to low-power-consumption, so it only takes "enough" power, no more, no less.

```C
bool check_password(char *password){
	for (int i = 0; i < password_length; i++){
		if (password[i] != stored_password[i]){
			return false;
		}
	}
	return true;
}
```

The above example is a famous password cracking method, by comparing the password with stored one (dictionaries, common phrases,...), if wrong, return immediately. Basing on timing information, we can guess the password. Because if true - takes longer than false (because the code iterates from 0 to max length).