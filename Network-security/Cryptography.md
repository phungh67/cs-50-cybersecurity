
Can check the previous lecture in [[Cybersecurity/cs-50-cybersecurity/Computer security/Cryptography|Cryptography - computer security]]

But from the point of view of Network Security, there are several considerations to take in account.

# Symmetric and Asymmetric key encryption

>[! Definition]
>Symmetric key encryption - use only one key for both encryption and decryption process. While asymmetric results in a pair of keys. One will be used in encryption and the other is for decryption. User can choose which key would be published (typical public key).


## Feistel Cipher structure 

It's not really a kind of cryptography, but more like a frame work to develop the theory of modem cryptography.

Typically, when a plain text was fed to the system, it will be split into $L$ and $R$, then mixed with key and function:
- $L_{i+1}=R_i$ 
- $R_{i+1}=L_{i} \oplus F(R_i, K_i)$   - which means that the right hand-side will be put in the function with round key - derivable from the main key.
- There are multiple rounds (at least 16).
- Last round - last swap - then merge.

## Symmetric keys

There are two notable candidates:
- DES - broken since the key only 56 bits length and the S-box is a little mystery - hence does not satisfy transparent principle in design. Also, cracked less than hour with today's computing power. A triple DES was used as a method to further improve the security, but the trade-off was speed.
- AES came with permutation and substitutions. And most important: supports larger key size: 128, 192 and 256 bits. Support in most modern application

## Block and stream

Block - split the text into chunks (with same size, padding if necessary) then begin to encrypt. Because if the key is for example, 128 bits, we have 4 chunks of 128 bits, it's good, but if the last chunk only 120 bits - need padding

Stream, on other hand, encrypt bit by bit

But most important, the code was not designed to handle multiple blocks, so we have several modes:
### ECB - Electronic Code-Block mode

Block + key -> cipher text. Limitation: same block results in same output.

### CBC - Cipher Block Chaining mode

An initial vector + key to produce cipher text from first block, then use the first block to mix with next block and encrypt. Can be decrypted in parallel but not encrypt

### CRT - Counter mode

Go with counter and an initial vector (a nonce - random number + time), encrypt the counter + vector with the key to produce the keystream, then XOR with the plain text.

## Cryptographic Hashed

Normally, hash take a data, hashed them, to make a "fixed-length" output. Then, how to make it more convenient in information exchange?

$HMAC_k(text)=hash(K \oplus opad || hash(K \oplus ipad || text))$

The above is about Hash - MAC (message authentication). The text will be appended by the inner pad (to match with the length of the key) - then xored with the key, hashed again, then added outer pad again before xored with the key, and finally, hashed one last time.

# Diffie-Hellman key exchange

Key for 2 parties that do not know each other to agree on communication over an insecure channel.

They must agree on two common values: $g$ and $p$.

Party A will take a secret $a$, perform $g^a \mod p$ then sends to B. B will do the same, take secret $b$ and sends $g^b \mod p$

Then two sides will begin to compute the session key - a shared secret key (with same key-generation algorithm)

$K = B^a = (g^b)^a \mod p$ (on B's side) and $K = A^b = (g^a)^b \mod p$ (on A's side). So, same key, but different secret to exchange.

# Summary

- Asymmetric key and cryptography for authentication (certificates, or SSH hehe).
- Diffile-Hellman for session key agreement.
- Symmetric ciphers for bulk or stream operation.
- Keyed hashed to detect data modification - protect the integrity.