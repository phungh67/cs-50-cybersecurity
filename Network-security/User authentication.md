
>[! Summary]
>The authentication process over insecure networks, how the credentials can be steal and how to secure this step?

# Forewords

There is a bunch of common passwords that are widely used by many users throughout the internet. Then, a typical attacker can use these well-known words/phrases to conduct something calls "dictionary" attack - an offensive action based on common knowledge, only scan for common patterns to gain access to personal vaults, accounts,...

# 1. Passwords

A string or an entity that will provide user accessibility to a typical system if successfully input to the authentication mechanism. Commonly, these "data" should be never stored in plain-text or keeping the same value for a too long time.

Passwords are often stored in encrypted or hashed form (both methods are applied, storing hashed result in encrypted form). Because:
- If simply store these things in plain-text, a hashed dictionaries can be use in reversed. Or the most classical way: brute force.
- Moreover, a pre-calculated lookup table can be shared between adversaries.
- A simple approach can be added to overcome the weaknesses: salted bit, to ensure that two same strings will be hashed into different results. (but keep in mind that this method requires 1 look up table per salt - for checking and verifying purpose)
- [[Rainbow table]] is also used in attack to reduce attacking time.

# 2. LDAP

A protocol, Light-weight Directory Access Protocol - simply. a protocol for database access (and yes, used for directory listing - information about users: password, roles, organization, mail address, phone number. Format: typically x500). Secure with TLS. Microsoft supports many other methods: Kerberos, Radius along with LDAP.

## 2.1. Example and common operations.

As mentioned before, each entry contains attributes and a unique "id" calls `dn`- distinguished name.

```Bash
# common operations are (but not limited to)
# get, add, delete, modify, seach, compare, extend
dn: cn=Jonh Smith, dc=company, dc=com # this iss the dn of entry/record
sn: Smith # an attribute, sn stands for surname
givenName: Jonh
userPassword: fjjf3n¤jfk1
mail: jonh.smith@company.com
telephoneNumber: +1 555 123 4567

# with cn is the common name of the object, the primary name
# and dc is the domain component, to determine where the object in 
# the tree of organization
```

For example, we have this operation
```Bash
GET E-Mail.Brown.Faculty.Businesss.Waikiki
```

The command will travel from the biggest one University of Waikiki (Organization - O with Common Name: Waikiki) to Business (Organization Unit - OU) then a child of it: Business. Within that sub unit, it looks up for entity with a common name: Brown, then fetch an attribute names: email.

# 3. Standards of authenticating into a system

Must not send the login information in plain text over a network, even if that network was secured with the transfer layer secure protocol (encrypted in-transit packets). Even with encrypted value, it still can be relayed by the man-in-the-middle attackers.

First method

## CHAP: Challenge Handshake Authentication Protocol

The client sends an authenticate request message to the server.
Server replies with a challenge (a nonce - unique in timestamp).
Then the client will send a secret and challenge message in hashed form for the server. For the server to be able to verify it, it must have the secret in clear text on its side.

## ISO 9798 Entity authentication

ISO has standardized some authentication methods. That is, 9798-2 stands for asymmetric algorithms with 4 is for three-pass mutual authentication with symmetric key

### 9798-2-4: 2 - asymmetric and 4 - three pass mutual authentication

- Client A sends $R_a || Text_1$ for authentication purpose.
- The server sends back $Text_3 || E_k(R_b || R_a || ID_a || Text_2)$ (K is common key)
- Since A and B agreed on the common key, A must be able to retrieve the value $R_b$ then reply with a new text, include both $R_a$ - its secret and $R_b$ the secret from server.
- The response shows that both sides are alive and belong to the same session.

### 9798-4-3: 4 - hashed and 3 - two pass mutual authentication

- Client sends a timestamp $TN_a || Text_a || hash_k(TN_a || ID_b || {Text}_n)$ that TN is the timestamp or sequence number - never reused, with n is some optional thing. The client states that it can hashed the nonce with provided key (shared key).
- Then the server sends back $TN_b || Text_n || hash_k(TN_b || ID_a || {Text}_n)$

### One-time passwords: S/Key algorithm

Give a chain of hash $Hash_n(x)=Hash(Hash(...Hash(x)))=Hash(Hash_ {n-1}(x))$ user knows x while the server has result of hash-n. Then when asked, user side computes the hash-n and sends back to server for authenticating. But server only stores last result.

## Radius authentication

Client sends a request that contains:
- A random data in 16-octet form.
- ID to authenticate (to know which entity is going to access this data).
- Username and password - which is XOR with hashed value (in MD5 format) of a shared secret between server and client and the said random data.
- A message authenticator - forming with MD5 hash of packet content (message, request) and shared secret to prevent the modification of request in transit.

On the other hand, server sends back:
- Access granted or rejected message with the ID of requester and a message authenticator.
- The authenticator also contains the packet, but also a random nonce sent by client earlier and the shared secret.