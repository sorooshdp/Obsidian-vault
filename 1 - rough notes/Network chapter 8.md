# Comprehensive Study Guide: Computer Networking — Chapter 8 (Security in Computer Networks)

## 1. Fundamental Principles of Network Security

Network security encompasses the protocols, mathematical algorithms, and operational architectures designed to protect communication across inherently untrusted physical and logical media. To systematically analyze network security, computer scientists define four core security properties that any secure system must provide:

* **Confidentiality:** The requirement that only the intended sender and receiver can understand the contents of a transmitted message. An eavesdropper who intercepts an encrypted message (ciphertext) must be computationally incapable of recovering the original plaintext.
* **Message Integrity:** The assurance that a received message has not been altered, corrupted, tampered with, or forged in transit—whether by accidental channel noise or malicious adversary intervention.
* **Endpoint Authentication:** The process of verifying the identity of a communicating entity (a host, client, server, user, or router) before establishing a session or granting access to resources.
* **Operational Security and Availability:** The mechanisms (such as firewalls, intrusion detection systems, and access controls) that ensure network resources remain accessible to authorized users while defending internal infrastructure against malware, unauthorized intrusion, scanning, and Denial-of-Service (DoS) attacks.

### 1.1 The Threat Model and Adversary Capabilities

In network protocol design, security mechanisms are modeled against an active adversary historically referred to as Trudy (the intruder), who sits on the communication path between sender Alice and receiver Bob. An adversary may possess a wide range of capabilities:

1. Eavesdropping (Packet Sniffing): Passively intercepting and reading frames or datagrams passing over shared physical media (such as wireless radio links or broadcast Ethernet).
2. Message Insertion, Modification, and Deletion: Actively injecting bogus packets into an active stream, altering specific header fields or payload bytes of in-flight packets, or dropping selected packets to cause disruption.
3. Impersonation (Spoofing): Generating IP datagrams or link-layer frames containing fabricated source addresses to impersonate a trusted entity.
4. Hijacking: Forcibly taking over an active, established transport connection by spoofing valid sequence and acknowledgment numbers.
5. Replay Attacks: Recording legitimate encrypted or authenticated messages and retransmitting them later to execute unauthorized transactions or bypass authentication.

## 2. Principles of Cryptography

Cryptography provides the foundational mathematical techniques for achieving confidentiality and message authentication. Cryptographic systems transform plaintext message $m$ into ciphertext $c$ using an encryption algorithm E and a key K, written as $c = E_K(m)$. The receiver uses a decryption algorithm D and key K' to recover the original message: $m = D_{K'}(c) = D_{K'}(E_K(m))$.

### 2.1 Symmetric Key Cryptography

In symmetric key cryptography, Alice and Bob share a single, identical secret key $K_S$. The encryption key and decryption key are identical ($K = K' = K_S$), and this key must remain strictly secret between the communicating parties.

```text
                  +-------------------+
                  | Secret Key (K_S)  |
                  +---------+---------+
                            |
  Plaintext (m) ---> [ Encryption E ] ---> Ciphertext (c)
                                                 |
                                         (Untrusted Medium)
                                                 |
  Plaintext (m) <--- [ Decryption D ] <--- Ciphertext (c)
                            ^
                            |
                  +---------+---------+
                  | Secret Key (K_S)  |
                  +-------------------+
```

#### Historical Ciphers

* Caesar Cipher: A substitution cipher that shifts each letter in the alphabet by a fixed integer key k. For $k = 3$, $a \to d$, $b \to e$, $\dots$, $z \to c$. The Caesar cipher has only 25 possible keys in the English alphabet, rendering it trivial to break via exhaustive brute-force search.
* Monoalphabetic Cipher: Replaces each letter of the alphabet with a unique assigned substitute letter based on a 26-letter permutation key. While this yields $26! \approx 4 \times 10^{26}$ possible keys, it is easily broken using statistical frequency analysis, matching the observed frequency of ciphertext characters against standard language letter distributions (e.g., 'e', 't', 'a' appearing most frequently in English).
* Polyalphabetic Ciphers (Vigenère): Uses a repeating keyword to determine variable Caesar shifts across consecutive positions in the plaintext, smoothing out single-letter frequency distributions.

#### Block Ciphers and Operation Modes

Modern symmetric cryptography processes data in fixed-size blocks (e.g., 64 bits for DES, 128 bits for AES). An n-bit block cipher maps an n-bit input block to an n-bit output block using a secret key.

* Data Encryption Standard (DES): Encrypts 64-bit blocks using a 56-bit key through 16 rounds of Feistel network substitutions and permutations. Due to its short 56-bit key length ($2^{56} \approx 7.2 \times 10^{16}$ keys), DES was rendered vulnerable to brute-force decryption. Triple-DES (3DES) extends key length by applying DES three times with different keys.
* Advanced Encryption Standard (AES): The current standard symmetric cipher, processing 128-bit blocks using key lengths of 128, 192, or 256 bits through repeated rounds of byte substitution, row shifting, column mixing, and round-key addition.

#### Cipher Block Chaining (CBC) Mode

When encrypting long messages consisting of multiple blocks $m(1), m(2), \dots, m(k)$, encrypting each block independently—known as Electronic Codebook (ECB) mode—is insecure because identical plaintext blocks always produce identical ciphertext blocks, revealing structural patterns.

To solve this, Cipher Block Chaining (CBC) introduces an Initialization Vector (IV)—a randomly generated n-bit number—and XORs each plaintext block with the preceding ciphertext block before encryption:

$$\begin{aligned} c(1) &= E_{K_S}(m(1) \oplus \text{IV}) \ c(i) &= E_{K_S}(m(i) \oplus c(i-1)) \quad \text{for } i \ge 2 \end{aligned}$$

```text
Block 1:  m(1)  ------> (+) ------> [ Encrypt E_{K_S} ] ------> c(1)
                         ^                                        |
                        IV                                        v
Block 2:  m(2)  ------> (+) ------> [ Encrypt E_{K_S} ] ------> c(2)
                         ^                                        |
                        c(1) -------------------------------------+
```

Decryption operates inversely:

$$\begin{aligned} m(1) &= D_{K_S}(c(1)) \oplus \text{IV} \ m(i) &= D_{K_S}(c(i)) \oplus c(i-1) \quad \text{for } i \ge 2 \end{aligned}$$

Because $D_{K_S}(c(i)) = m(i) \oplus c(i-1)$, XORing the decrypted output with $c(i-1)$ yields $(m(i) \oplus c(i-1)) \oplus c(i-1) = m(i)$. The sender transmits the unencrypted IV alongside the ciphertext blocks so the receiver can initialize decryption.

### 2.2 Public-Key Cryptography & The RSA Algorithm

Public-key cryptography (asymmetric key cryptography) uses two distinct, mathematically paired keys for each entity:

* A Public Key ($K_B^+$): Known to the entire world, used by anyone to encrypt messages intended for Bob.
* A Private Key ($K_B^-$): Kept strictly secret by Bob, known only to Bob, used to decrypt ciphertext encrypted with his public key.

The fundamental identity governing public-key encryption is: $D_{K_B^-}\left( E_{K_B^+}(m) \right) = m$

Furthermore, for key pairs created under algorithms like RSA, the operations are commutative: $E_{K_B^+}\left( D_{K_B^-}(m) \right) = D_{K_B^-}\left( E_{K_B^+}(m) \right) = m$

```text
Alice: Plaintext (m) ---> [ Encrypt with K_B^+ ] ---> Ciphertext c = (m^e mod n)
                                                                 |
                                                         (Public Internet)
                                                                 |
Bob:   Plaintext (m) <--- [ Decrypt with K_B^- ] <--- Ciphertext c
```

#### The RSA Algorithm: Key Generation

The RSA algorithm (named after Rivest, Shamir, and Adleman) relies on the mathematical difficulty of factoring the product of two large prime numbers. The key generation process follows these steps:

1. Select Primes: Choose two extremely large prime numbers p and q (typically 1024 bits or larger each).
2. Compute Modulus: Calculate $n = p \cdot q$. The integer n is the modulus for both encryption and decryption.
3. Compute Euler's Totient: Calculate $z = (p - 1)(q - 1)$.
4. Select Public Exponent: Choose an integer $e < n$ such that e and z have no common factors other than 1 (i.e., $\gcd(e, z) = 1$).
5. Compute Private Exponent: Calculate an integer d such that $e \cdot d \equiv 1 \pmod z$. This means $(e \cdot d - 1)$ is an exact integer multiple of z.
6. Publish Keys:
    * The Public Key is the pair $(n, e)$.
    * The Private Key is the pair $(n, d)$.

#### RSA Encryption and Decryption

To send a message represented as an integer m ($0 \le m < n$) to Bob:

* Encryption by Alice: Alice computes ciphertext c: $c = m^e \bmod n$
* Decryption by Bob: Bob computes plaintext m: $m = c^d \bmod n$

#### Mathematical Proof of RSA

RSA relies on Euler's Generalization of Fermat's Little Theorem, which states that if $\gcd(m, n) = 1$, then: $m^{(p-1)(q-1)} \equiv 1 \pmod n \implies m^z \equiv 1 \pmod n$

Since $e \cdot d \equiv 1 \pmod z$, we can write $e \cdot d = k \cdot z + 1$ for some integer k. Substituting this into the decryption formula yields: $c^d \bmod n = (m^e)^d \bmod n = m^{ed} \bmod n = m^{kz + 1} \bmod n = (m^z)^k \cdot m \bmod n = (1)^k \cdot m \bmod n = m \bmod n$

#### Computational Efficiency & Session Keys

Because RSA involves modular exponentiation of multi-hundred-digit numbers, it is orders of magnitude slower than symmetric block ciphers like AES. Consequently, real-world protocols use hybrid cryptosystems:

1. Alice generates a temporary, random symmetric session key $K_S$.
2. Alice encrypts the actual payload data using $K_S$ via fast symmetric encryption ($c = E_{K_S}(m)$).
3. Alice encrypts the session key $K_S$ using Bob's public key via RSA ($c_{\text{key}} = (K_S)^e \bmod n$).
4. Alice sends both $c_{\text{key}}$ and c to Bob. Bob decrypts $c_{\text{key}}$ using his private key $K_B^-$ to recover $K_S$, then decrypts c using $K_S$.

## 3. Message Integrity, Digital Signatures, and PKI

Confidentiality prevents unauthorized reading of data, but it does not prevent an adversary from tampering with or replacing messages. Message integrity guarantees that data received is identical to data sent.

### 3.1 Cryptographic Hash Functions

A cryptographic hash function $H(m)$ takes an arbitrary-length block of data m as input and produces a fixed-length bit string (hash value or message digest) as output. To be cryptographically secure, H must satisfy two fundamental properties:

1. One-Way (Pre-image Resistance): Given a digest h, it is computationally infeasible to find any message m such that $H(m) = h$.
2. Collision Resistance: It is computationally infeasible to find any two distinct messages x and y ($x \neq y$) such that $H(x) = H(y)$.

Widely used hash algorithms include:

* MD5: Produces a 128-bit digest. Collision attacks against MD5 are now practical, making it deprecated for security applications.
* SHA-1: Produces a 160-bit digest; also deprecated due to cryptographic weaknesses.
* SHA-256 / SHA-3: Produces 256-bit (or 512-bit) digests; currently standard for network security protocols.

### 3.2 Message Authentication Codes (MAC)

To verify message integrity and sender identity without using heavy public-key operations, communicating parties who share a secret key s use a Message Authentication Code (MAC).

1. Alice concatenates her message m with the shared secret key s, forming $(m + s)$.
2. Alice calculates the hash digest of this concatenated string: $H(m + s)$. This hash value is the MAC.
3. Alice attaches the MAC to the plaintext message m and transmits $(m, H(m + s))$ to Bob.
4. Bob receives $(m, h')$, appends his copy of shared secret s, computes $H(m + s)$, and compares it with h'. If they match, Bob confirms the message was not modified and was created by someone possessing secret key s.

```text
Alice: [ Message (m) + Secret (s) ] ---> [ Hash H ] ---> MAC = H(m + s)
       Send: (m, MAC) over Internet
Bob:   Receive (m, MAC) ---> Compute H(m + s) ---> Compare with MAC
```

Crucially, the secret key s is never transmitted over the wire; it is only held by the endpoints.

### 3.3 Digital Signatures

A Digital Signature is a cryptographic mechanism that provides message integrity, origin authentication, and non-repudiation (the sender cannot later deny generating the signature).

#### Signing and Verification Sequence

1. Signing: Bob creates a digital signature for message m by passing m into a hash function $H(m)$, then encrypting the resulting digest with his private key $K_B^-$: $\text{Signature } \sigma = D_{K_B^-}(H(m))$
2. Transmission: Bob sends the package $(m, \sigma)$ to Alice.
3. Verification: Alice receives $(m, \sigma)$ and verifies it by:
    * Decrypting $\sigma$ using Bob's public key $K_B^+$: $h_1 = E_{K_B^+}(\sigma) = E_{K_B^+}(D_{K_B^-}(H(m))) = H(m)$.
    * Computing the hash of the received message: $h_2 = H(m)$.
    * Checking if $h_1 == h_2$. If equal, the signature is valid.

```text
Bob (Signer):    m ---> [ Hash H ] ---> H(m) ---> [ Encrypt with K_B^- ] ---> Signature σ
Alice (Verifier): σ ---> [ Decrypt with K_B^+ ] ---> H(m)_received
                 m ---> [ Hash H ] -------------> H(m)_calculated  (Compare)
```

Because only Bob possesses $K_B^-$, no one else could have generated a valid signature $\sigma$ that decrypts successfully using $K_B^+$.

### 3.4 Public Key Infrastructure (PKI) and Certification Authorities

A fundamental vulnerability in public-key systems is the Man-in-the-Middle (MitM) attack during public key distribution. If Trudy intercepts Alice's request for Bob's public key, Trudy can return her own public key $K_T^+$ instead of $K_B^+$. Alice will then encrypt session keys using $K_T^+$, allowing Trudy to decrypt, read, modify, and re-encrypt traffic destined for Bob.

To solve this problem, networks deploy a Public Key Infrastructure (PKI) based on trusted Certification Authorities (CAs).

```text
                  +--------------------------------+
                  | Certification Authority (CA)   |
                  | Private Key: K_CA^-            |
                  +---------------+----------------+
                                  |
                                  | Certificate = Sign_{K_CA^-}(Identity, K_B^+)
                                  v
+-------+   Fetches Certificate    +-------+
| Alice | -----------------------> |  Bob  |
+-------+                          +-------+
    |
    +---> Decrypts Certificate using trusted K_CA^+
          Extracts and verifies Bob's authentic K_B^+
```

#### Certification Authority Workflow

1. Binding: A Certification Authority validates the real-world identity of an entity (e.g., website domain www.bank.com owned by Bob).
2. Certificate Issuance: The CA creates a digital certificate containing Bob's identity details and Bob's public key $K_B^+$. The CA signs this entire package using the CA's private key $K_{\text{CA}}^-$: $\text{Certificate} = \left[ ( \text{Identity}, K_B^+ ), D_{K_{\text{CA}}^-}(H(\text{Identity}, K_B^+)) \right]$
3. Verification: When Alice connects to Bob, Bob presents his CA-signed certificate. Alice decrypts the signature using the CA's publicly known public key $K_{\text{CA}}^+$ (pre-installed in her operating system or browser trust store). If valid, Alice obtains Bob's authentic public key $K_B^+$.

#### X.509 Certificate Fields

Standard digital certificates conform to the X.509 format, containing:

* Subject Name: Distinguished name of the entity owning the public key.
* Subject Public Key: The actual public key value and algorithm identifier.
* Issuer Name: The CA that issued and signed the certificate.
* Validity Period: Start date and expiration date.
* Serial Number: Unique identification number assigned by the CA.
* CA Digital Signature: Signature produced under $K_{\text{CA}}^-$.

## 4. Endpoint Authentication Protocols

Endpoint authentication is the protocol mechanism by which one entity proves its identity to another over a network.

### 4.1 Evolution of Authentication Protocols (ap1.0 to ap5.0)

#### Protocol ap1.0 ("I am Alice")

* Mechanism: Alice sends a text message stating "I am Alice".
* Flaw: Completely insecure. Trudy can send "I am Alice" and impersonate Alice.

#### Protocol ap2.0 (IP Source Address Authentication)

* Mechanism: Alice sends "I am Alice" in an IP datagram originating from Alice's IP address.
* Flaw: Vulnerable to IP Spoofing. Trudy can construct raw IP packets containing Alice's source IP address.

#### Protocol ap3.0 (Secret Password)

* Mechanism: Alice transmits a secret password shared between Alice and Bob.
* Flaw: Vulnerable to Eavesdropping (Playback Attack). Trudy sniffs the password in plaintext and re-uses it later.

#### Protocol ap3.1 (Encrypted Secret Password)

* Mechanism: Alice encrypts her secret password using a shared symmetric key $K_{A-B}$.
* Flaw: Still vulnerable to Replay Attacks. Trudy does not need to decrypt the password; she simply sniffs the encrypted ciphertext payload and replays the exact encrypted string to Bob later to authenticate.

### 4.2 Nonce-Based Authentication (ap4.0 and ap5.0)

To prevent replay attacks, authentication protocols introduce a Nonce—a random number used only once per authentication session.

#### Protocol ap4.0 (Symmetric-Key Nonce Authentication)

1. Alice sends "I am Alice" to Bob.
2. Bob generates a unique, unpredictable random nonce R and sends R to Alice.
3. Alice encrypts the nonce using the shared symmetric key $K_{A-B}$ and returns $K_{A-B}(R)$ to Bob.
4. Bob decrypts the response using $K_{A-B}$. If the recovered value equals R, Bob authenticates Alice.

```text
Alice                                              Bob
  | -------------- "I am Alice" --------------->    |
  | <---------------- Nonce R ------------------    |
  | -------- Compute K_{A-B}(R) --------------->    | (Decrypt & verify R)
```

Trudy cannot replay $K_{A-B}(R)$ in a future session because Bob will generate a new, different nonce R' for every fresh authentication attempt.

#### Protocol ap5.0 (Public-Key Nonce Authentication)

Instead of a shared symmetric key, ap5.0 uses public-key cryptography and digital signatures:

1. Alice sends "I am Alice" to Bob.
2. Bob sends a random nonce R to Alice.
3. Alice signs the nonce using her private key: $\sigma = D_{K_A^-}(R)$, and sends $\sigma$ to Bob.
4. Bob fetches Alice's verified public key $K_A^+$ and computes $E_{K_A^+}(\sigma)$. If $E_{K_A^+}(\sigma) == R$, Bob authenticates Alice.

## 5. Application-Layer Security: Secure E-Mail (PGP)

Email systems designed around SMTP pass messages in plain text across intermediate mail transfer agents (MTAs). To secure email, security services must be integrated at the application layer. Pretty Good Privacy (PGP) is a widely deployed application-layer security protocol for email.

### 5.1 Achieving Comprehensive E-Mail Security

A fully secure email protocol must provide confidentiality, sender authentication, message integrity, and non-repudiation. PGP achieves all four goals by combining symmetric encryption, public-key encryption, cryptographic hashes, and digital signatures into a unified processing pipeline.

```text
ALICE (Sender):
Plaintext Message (m)
   |
   +---> [ Hash H ] ---> H(m) ---> [ Encrypt w/ K_A^- ] ---> Digital Signature σ
   |                                                                |
   +----------------------- Concatenate (+) <-----------------------+
                                  |
                           Signed Message (m + σ)
                                  |
                      [ Encrypt w/ Session Key K_S ]
                                  |
                          Encrypted Payload
                                  |
   +----------------------- Concatenate (+) <--- [ Encrypt K_S w/ K_B^+ ]
   |                                                     (Encrypted Key)
   v
Transmitted Package: [ E_{K_B^+}(K_S) ] + [ E_{K_S}( m + D_{K_A^-}(H(m)) ) ]
```

### 5.2 Step-by-Step PGP Operations

#### Sending Phase (Alice)

1. Digital Signature Generation:
    * Alice computes the digest of message m: $h = H(m)$.
    * Alice signs h using her private key $K_A^-$: $\sigma = D_{K_A^-}(h)$.
    * Alice appends $\sigma$ to m, creating signed payload $(m, \sigma)$.
2. Symmetric Encryption:
    * Alice generates a one-time random symmetric session key $K_S$.
    * Alice encrypts the signed payload $(m, \sigma)$ using $K_S$ with AES/DES: $c_{\text{data}} = E_{K_S}(m, \sigma)$.
3. Session Key Encryption:
    * Alice encrypts the session key $K_S$ using Bob's public key $K_B^+$ via RSA: $c_{\text{key}} = E_{K_B^+}(K_S) = (K_S)^e \bmod n$.
4. Package Assembly:
    * Alice concatenates $c_{\text{key}}$ and $c_{\text{data}}$ into a single binary stream, base64-encodes it into ASCII characters, and sends it as an email payload.

#### Receiving Phase (Bob)

1. Session Key Recovery: Bob uses his private key $K_B^-$ to decrypt $c_{\text{key}}$ and recover $K_S$: $K_S = D_{K_B^-}(c_{\text{key}})$
2. Payload Decryption: Bob uses $K_S$ to decrypt $c_{\text{data}}$, yielding cleartext message m and signature $\sigma$.
3. Integrity & Authentication Verification:
    * Bob decrypts signature $\sigma$ using Alice's public key $K_A^+$, obtaining digest $h_1 = E_{K_A^+}(\sigma)$.
    * Bob computes hash $h_2 = H(m)$ over the decrypted message text.
    * If $h_1 == h_2$, Bob proves that Alice authored the message (authentication) and that the message was not modified in transit (integrity).

## 6. Transport-Layer Security: TLS / SSL

Transport Layer Security (TLS)—and its predecessor Secure Sockets Layer (SSL)—provides security services for TCP connections. Positioned technically at the application layer above TCP (providing a socket API identical to standard TCP), TLS secures applications such as HTTPS (HTTP over TLS), secure IMAP, and SSH.

```text
+---------------------------------------------------+
|               Application (HTTP)                  |
+---------------------------------------------------+
|            Transport Layer Security (TLS)         |  <-- Secures Socket Interface
+---------------------------------------------------+
|            Transmission Control Protocol (TCP)    |
+---------------------------------------------------+
|            Internet Protocol (IP)                 |
+---------------------------------------------------+
```

### 6.1 The Three Phases of TLS

A TLS session between a client (Bob) and a server (Alice) consists of three phases: Handshake, Key Derivation, and Data Transfer.

```text
Client (Bob)                                          Server (Alice)
     |                                                      |
     | ------------------ 1. ClientHello -----------------> |
     | <----------------- 2. ServerHello ------------------ |
     | <----------------- 3. Certificate ------------------ | (Contains K_A^+)
     |                                                      |
     | -- 4. Pre-Master Secret (EMS = E_{K_A^+}(PMS)) ---> | (Encrypted w/ K_A^+)
     |                                                      |
     +======== BOTH DERIVE MASTER SECRET & KEYS ===========+
     |                                                      |
     | <---------------- 5. Finished (MAC) ----------------> |
     |                                                      |
     +=============== SECURE DATA TRANSFER =================+
     | <============== TLS Record Frames =================> |
```

#### Phase 1: Handshake

1. ClientHello: Bob sends a ClientHello message specifying supported TLS version numbers, supported cryptographic cipher suites (e.g., RSA with AES-256-CBC and SHA-256), and a client random string $R_C$.
2. ServerHello: Alice responds with chosen cipher suite parameters, a server random string $R_S$, and her CA-signed X.509 Digital Certificate containing her public key $K_A^+$.
3. Pre-Master Secret Generation: Bob verifies Alice's certificate against his trusted CA store. Bob then generates a 48-byte random Pre-Master Secret (PMS), encrypts it using Alice's public key $K_A^+$, and transmits $E_{K_A^+}(\text{PMS})$ to Alice.
4. Master Secret Calculation: Alice decrypts the message using her private key $K_A^-$ to recover PMS. Both endpoints now independently compute identical Master Secrets (MS) using a pseudo-random function (PRF): $\text{MS} = \text{PRF}(\text{PMS}, R_C, R_S)$

#### Phase 2: Key Derivation

To prevent key-reuse vulnerabilities (e.g., using the same key for encryption and MAC, or for sending and receiving), the Master Secret is expanded into four distinct symmetric keys and two Initialization Vectors:

1. $E_B$: Client-to-Server Encryption Key (Bob $\to$ Alice data confidentiality).
2. $M_B$: Client-to-Server MAC Key (Bob $\to$ Alice message integrity).
3. $E_A$: Server-to-Client Encryption Key (Alice $\to$ Bob data confidentiality).
4. $M_A$: Server-to-Client MAC Key (Alice $\to$ Bob message integrity).

#### Phase 3: Data Transfer (TLS Record Protocol)

Data passed down from the application layer is processed by the TLS Record Layer:

1. Fragmentation: The stream is broken into manageable chunks (up to 16 KB).
2. Sequence Numbering: TLS maintains a implicit 64-bit sequence number N for each direction.
3. MAC Calculation: A MAC is computed over the sequence number, header, and data: $\text{MAC} = H(M_B, N + \text{header} + \text{data})$
4. Encapsulation & Encryption: The data payload and MAC are appended together and encrypted using symmetric key $E_B$.
5. Header Addition: A 5-byte unencrypted TLS Record Header (Type, Version, Length) is prepended, and the resulting record is passed to TCP for transmission.

```text
Application Data Payload
   |
   v
[ Fragment ] ---> [ Append MAC: H(M_B, N + Header + Fragment) ]
                        |
                        v
            [ Encrypt via Symmetric Key E_B ]
                        |
                        v
   [ TLS Record Header (Type, Version, Length) ] + [ Encrypted (Fragment + MAC) ]
```

#### Defending Against Connection Hijacking and Deletion Attacks

* Sequence Numbers in MAC: Including sequence number N inside the MAC calculation prevents an active attacker from reordering or inserting bogus TCP segments into the stream, as MAC verification will fail.
* Closure Alerts: To prevent an attacker from prematurely terminating a session by injecting spoofed TCP FIN packets (a Truncation Attack), TLS requires explicit encrypted close_notify alert records before closing a socket.

## 7. Network-Layer Security: IPsec and Virtual Private Networks

While TLS secures individual transport connections, IPsec operates at the Network Layer (Layer 3), securing all IP datagrams traveling between two network entities (hosts or routers) regardless of transport or application protocols.

### 7.1 Virtual Private Networks (VPNs) and IPsec Deployments

Institutions use IPsec to construct Virtual Private Networks (VPNs) over the public Internet. Instead of leasing expensive dedicated physical lines between branch offices, organizations send encrypted IPsec traffic across public ISPs.

```text
Branch Office Network                           Headquarters Network
[ Host H1 ]                                        [ Host H2 ]
    |                                                   |
 (Native IP)                                       (Native IP)
    v                                                   v
[ IPsec Gateway R1 ] <=== Encrypted IPsec Tunnel ===> [ IPsec Gateway R2 ]
                           (Public Internet)
```

### 7.2 IPsec Modes: Transport vs. Tunnel Mode

IPsec supports two operational modes:

#### Transport Mode

* Scope: Secures host-to-host communications directly between two endpoints.
* Structure: IPsec headers (AH or ESP) are inserted directly between the original IP header and the transport layer (TCP/UDP) header. The original IP header source and destination fields remain unencrypted and visible.

#### Tunnel Mode

* Scope: Secures gateway-to-gateway or host-to-gateway communications across intermediate networks (standard VPN deployment).
* Structure: The entire original IPv4 datagram (including original IP header and payload) is encrypted and encapsulated inside a brand-new IPsec datagram with a New IP Header addressing the VPN gateways (R1 and R2).

```text
Original IPv4 Datagram:
[ Orig IP Header (H1 -> H2) ] [ TCP Header ] [ Payload Data ]

Tunnel Mode ESP Datagram:
[ New IP Header (R1 -> R2) ] [ ESP Header ] [ Orig IP Header ] [ TCP ] [ Payload ] [ ESP Trailer ] [ ESP Auth ]
|<--- Unencrypted ----------->|<--------------------- Encrypted -------------------->|<--- Authenticated --->|
```

### 7.3 Security Associations (SAs)

A Security Association (SA) is a unidirectional, one-way logical connection established between two IPsec entities that defines the specific security algorithms, keys, and parameters governing traffic for that connection.

If router R1 and router R2 communicate securely in both directions, they must establish two distinct SAs (one for $R_1 \to R_2$, and one for $R_2 \to R_1$).

An SA is uniquely identified by a 3-tuple:

1. Security Parameter Index (SPI): A 32-bit integer carried explicitly in the IPsec header.
2. Destination IP Address: Address of the receiving entity.
3. Security Protocol Identifier: Specifies whether the SA uses AH or ESP.

#### IPsec Databases

Endpoints maintain two core database structures:

* Security Association Database (SAD): Stores the active parameters for all established SAs (SPI, encryption keys, MAC keys, algorithms, sequence number counters).
* Security Policy Database (SPD): Stores administrative rules defining how incoming and outgoing datagrams should be handled (e.g., whether to apply IPsec, bypass/pass cleartext, or discard/drop packets based on source/destination IP, ports, and protocols).

### 7.4 The IPsec Protocols: AH and ESP

IPsec defines two distinct header protocols:

* Authentication Header (AH): Provides source authentication and data integrity, but no confidentiality (payload is unencrypted). AH inserts an AH header containing an SPI and MAC digest. Because AH calculates its MAC over parts of the IP header (including IP addresses), it is incompatible with Network Address Translation (NAT).
* Encapsulating Security Payload (ESP): Provides confidentiality, source authentication, and data integrity. ESP encrypts the payload and appends an ESP trailer and ESP MAC authentication field. ESP is the dominant protocol in real-world VPNs.

### 7.5 Internet Key Exchange (IKE) Protocol

While network administrators can manually configure static SAs for simple 2-router VPNs, large enterprise VPNs with thousands of hosts use the Internet Key Exchange (IKE) protocol (RFC 5996) to automate key negotiation, authentication, and SA creation.

IKE operates in two phases:

* IKE Phase 1: Establishes an initial secure, authenticated channel between endpoints called the IKE SA (using Diffie-Hellman key exchange and pre-shared keys or digital certificates).
* IKE Phase 2: Uses the secure IKE SA channel to negotiate parameters and establish the actual operational unidirectional IPsec SAs used for data transfer.

## 8. Link-Layer and Wireless Security

Wireless networks transmit data over unguided radio spectrum, allowing any receiver positioned within range to sniff or inject link-layer frames. Consequently, link-layer security is essential for wireless infrastructure.

### 8.1 IEEE 802.11 (WiFi) Security Evolution

#### Wired Equivalent Privacy (WEP) Flaws

The original 802.11 security standard, WEP, suffered from severe architectural flaws:

* Used the RC4 stream cipher with short 24-bit Initialization Vectors (IVs).
* Re-use of 24-bit IVs allowed attackers to collect packets over short durations, recover the keystream, and extract the secret key using publicly available automated tools (e.g., Aircrack-ng).
* Used an unkeyed Cyclic Redundancy Check (CRC-32) for integrity, allowing attackers to modify frame payloads without detection.

#### WPA and WPA2/WPA3 (IEEE 802.11i)

To replace WEP, the WiFi Alliance introduced WPA and WPA2 (IEEE 802.11i):

* WPA2 mandates the use of AES in Counter Mode with CBC-MAC (CCMP) for symmetric encryption and message integrity.
* Mutual Authentication: Ensures the mobile station authenticates the Access Point (AP) and the AP authenticates the mobile station.

```text
Mobile Device (M)                 Access Point (AP)             Authentication Server (AS)
     |                                   |                                   |
     |<========== IEEE 802.11 Link =====>|<========= Wired Network =========>|
     |                                   |                                   |
     | <---------------------- EAP / EAPoL / RADIUS -----------------------> |
     |                             (Mutual Authentication)                   |
     |                                   |                                   |
     +<--- 4-Way Handshake (WPA2) ------>+                                   |
     |    (Derive Pairwise Keys)         |                                   |
```

#### The WPA2 4-Way Handshake

1. Pre-Shared Key / Master Key: The station and Authentication Server (AS) share a Pairwise Master Key (PMK).
2. Handshake: The station and AP exchange random nonces (ANonce and SNonce) over a 4-way message sequence.
3. Key Derivation: Both sides combine PMK, nonces, and MAC addresses to derive a temporary symmetric Pairwise Transient Key (PTK) used to encrypt unicast data frames over the air link.

#### EAP and RADIUS Infrastructure

In enterprise WiFi networks, authentication is delegated to a centralized RADIUS server using the Extensible Authentication Protocol (EAP):

* Frames over the wireless link encapsulate EAP inside EAP over LAN (EAPoL) (IEEE 802.1X).
* The AP relays these EAP messages to the RADIUS server encapsulated inside UDP/IP datagrams.

### 8.2 4G/5G Cellular Security

Cellular networks implement mutual authentication and key management between user equipment (UE/USIM) and the core network using Authentication and Key Agreement (AKA):

* Symmetric Secret: A 128-bit or 256-bit secret key K is permanently stored in the hardware Universal Subscriber Identity Module (USIM) smartcard and the core network Home Subscriber Server (HSS) / Authentication Server Function (AUSF).
* Challenge-Response: The core network generates a random challenge RAND and computes an authentication vector (AUTN, RES, $K_{\text{SEAF}}$).
* Mutual Verification: The USIM validates AUTN to verify the core network, then computes response RES to prove its identity to the network. Cryptographic keys derived from K encrypt user data over the air interface.

## 9. Operational Security: Firewalls and Intrusion Detection Systems

Operational security devices monitor, filter, analyze, and control traffic crossing organizational network boundaries.

```text
Public Internet -------> [ Packet Filter / Firewall ] -------> Internal Network
                                     |
                                     v
                       [ Intrusion Detection System ]
                        (Deep Packet Inspection / Log)
```

### 9.1 Firewalls

A Firewall is an operational security device positioned at the perimeter connecting an organization's internal network to the public Internet. It inspects incoming and outgoing IP datagrams, making forward/drop decisions based on administrative rules.

#### 1. Stateless Packet Filters

Inspects individual packets in isolation based exclusively on fields in their Layer 3 and Layer 4 headers:

* Source and Destination IP Addresses.
* Protocol Type (TCP, UDP, ICMP).
* Source and Destination Port Numbers.
* TCP Flag Bits (SYN, ACK, FIN, RST).
* ICMP Message Types.

Example Stateless Access Control List (ACL):

| Action | Source IP | Dest IP | Protocol | Source Port | Dest Port | Flag Bit | Meaning |
| --- | --- | --- | --- | --- | --- | --- | --- |
| ALLOW | External | 130.207.1.5 | TCP | > 1023 | 80 | Any | Allow external web traffic to internal web server |
| ALLOW | 130.207.0.0/16 | External | TCP | > 1023 | 80 | Any | Allow internal clients to send HTTP requests outside |
| ALLOW | External | 130.207.0.0/16 | TCP | 80 | > 1023 | ACK | Allow returning external HTTP traffic to internal clients |
| DENY | Any | Any | Any | Any | Any | Any | Default Deny All |

#### 2. Stateful Packet Filters

Stateless filters cannot track state; for instance, an attacker could send bogus packets with the ACK bit set to bypass a rule allowing returning TCP traffic.

A Stateful Packet Filter solves this by tracking the status of active TCP connections in a dynamic Connection Table:

* When an internal host initiates an outbound TCP connection (SYN), the firewall creates a record in its state table tracking (Source IP, Dest IP, Source Port, Dest Port).
* Inbound packets with ACK bits set are permitted only if an active entry exists in the connection table. When the connection closes (FIN/RST), the entry is deleted.

#### 3. Application Gateways (Proxies)

Application-level proxies operate at Layer 7. Instead of allowing direct IP-level routing between internal hosts and the Internet, an application gateway terminates TCP connections from internal clients, performs application-specific security checks (e.g., verifying SQL commands or HTTP headers), and opens a separate TCP connection to the external destination.

### 9.2 Intrusion Detection Systems (IDS) and Intrusion Prevention Systems (IPS)

While firewalls inspect packet headers, modern security threats (such as buffer overflow exploits, SQL injections, and malware payloads) are embedded deep inside application-layer data payloads.

* Intrusion Detection System (IDS): Inspects packet headers and payload contents (Deep Packet Inspection [DPI]). When suspicious traffic patterns are detected, the IDS raises real-world alerts for network administrators.
* Intrusion Prevention System (IPS): An inline device that inspects packets and automatically drops malicious datagrams in real time before they reach their destination.

#### IDS/IPS Detection Methodologies

1. Signature-Based Detection: Compares packet payloads against a database of known attack signatures (e.g., specific byte sequences associated with a known Snort rule or buffer overflow string). Vulnerable to zero-day attacks whose signatures are not yet known.
2. Anomaly-Based Detection: Establishes a baseline profile of normal network traffic (e.g., average volume, protocol distribution, connection rates). Traffic exhibiting significant statistical deviations from the baseline is flagged as potential attack activity.

## 10. Worked Numerical & Algorithmic Trace Examples

### 10.1 Worked Numerical Example: RSA Key Generation and Encryption

#### Problem Statement

Walk through the RSA algorithm for key generation, message encryption, and decryption using small prime numbers $p = 5$ and $q = 11$. Encrypt the numeric plaintext message $m = 8$.

#### Step-by-Step Solution

##### Step 1: Key Generation

1. Compute Modulus n: $n = p \cdot q = 5 \cdot 11 = 55$
2. Compute Euler's Totient z: $z = (p - 1)(q - 1) = (5 - 1)(11 - 1) = 4 \cdot 10 = 40$
3. Select Public Exponent e: Choose $e = 3$, checking that $\gcd(e, z) = \gcd(3, 40) = 1$.
4. Calculate Private Exponent d: Find d such that $e \cdot d \equiv 1 \pmod z \implies 3d \equiv 1 \pmod{40}$. Testing multiples of 40: $40 \cdot 1 + 1 = 41$ (not divisible by 3); $40 \cdot 2 + 1 = 81$ (divisible by 3). $3d = 81 \implies d = 27$
5. Resulting Keys:
    * Public Key: $(n=55, e=3)$
    * Private Key: $(n=55, d=27)$

##### Step 2: Encryption

Given plaintext $m = 8$: $c = m^e \bmod n = 8^3 \bmod 55 = 512 \bmod 55$ Dividing 512 by 55: $55 \times 9 = 495$. $512 - 495 = 17 \implies c = 17$ The resulting ciphertext is $c = 17$.

##### Step 3: Decryption Verification

To decrypt $c = 17$: $m = c^d \bmod n = 17^{27} \bmod 55$

Using modular exponentiation via repeated squaring:

$$\begin{aligned} 17^1 &\equiv 17 \pmod{55} \ 17^2 &= 289 \equiv 14 \pmod{55} \ 17^4 &= 14^2 = 196 \equiv 31 \pmod{55} \ 17^8 &= 31^2 = 961 \equiv 26 \pmod{55} \ 17^{16} &= 26^2 = 676 \equiv 16 \pmod{55} \end{aligned}$$

Writing exponent 27 in binary (27 = 16 + 8 + 2 + 1):

$$\begin{aligned} 17^{27} \bmod 55 &= (17^{16} \cdot 17^8 \cdot 17^2 \cdot 17^1) \bmod 55 \ &= (16 \cdot 26 \cdot 14 \cdot 17) \bmod 55 \end{aligned}$$

Step-by-step modular multiplication:

1. $(16 \cdot 26) \bmod 55 = 416 \bmod 55 = 31$
2. $(31 \cdot 14) \bmod 55 = 434 \bmod 55 = 49$
3. $(49 \cdot 17) \bmod 55 = 833 \bmod 55 = 8$

The recovered plaintext is $m = 8$, confirming correctness.

### 10.2 Algorithmic Trace: Cipher Block Chaining (CBC) Encryption & Decryption

#### Problem Statement

Trace 3-block Cipher Block Chaining (CBC) encryption and decryption using 4-bit blocks.

* Key function $E_{K_S}(x) = (x + 5) \bmod 16$
* Inverse key function $D_{K_S}(y) = (y - 5) \bmod 16$
* Initialization Vector $\text{IV} = 1010_2 = 10_{10}$
* Plaintext blocks: $m(1) = 0011_2$ ($3_{10}$), $m(2) = 1100_2$ ($12_{10}$), $m(3) = 0011_2$ ($3_{10}$)

#### Encryption Trace

$$\begin{aligned} c(1) &= E_{K_S}(m(1) \oplus \text{IV}) = E_{K_S}(3 \oplus 10) = E_{K_S}(9) = (9 + 5) \bmod 16 = 14_{10} = 1110_2 \ c(2) &= E_{K_S}(m(2) \oplus c(1)) = E_{K_S}(12 \oplus 14) = E_{K_S}(2) = (2 + 5) \bmod 16 = 7_{10} = 0111_2 \ c(3) &= E_{K_S}(m(3) \oplus c(2)) = E_{K_S}(3 \oplus 7) = E_{K_S}(4) = (4 + 5) \bmod 16 = 9_{10} = 1001_2 \end{aligned}$$

Ciphertext generated: $c(1)=1110_2$, $c(2)=0111_2$, $c(3)=1001_2$. Notice that identical plaintext blocks $m(1)$ and $m(3)$ produced different ciphertexts $c(1)$ and $c(3)$.

#### Decryption Trace

$$\begin{aligned} m(1) &= D_{K_S}(c(1)) \oplus \text{IV} = D_{K_S}(14) \oplus 10 = ((14 - 5) \bmod 16) \oplus 10 = 9 \oplus 10 = 3_{10} = 0011_2 \ m(2) &= D_{K_S}(c(2)) \oplus c(1) = D_{K_S}(7) \oplus 14 = ((7 - 5) \bmod 16) \oplus 14 = 2 \oplus 14 = 12_{10} = 1100_2 \ m(3) &= D_{K_S}(c(3)) \oplus c(2) = D_{K_S}(9) \oplus 7 = ((9 - 5) \bmod 16) \oplus 7 = 4 \oplus 7 = 3_{10} = 0011_2 \end{aligned}$$

All original plaintext blocks are perfectly recovered.

## 11. Practice Examination

### Questions

#### Question 1

Define the difference between message confidentiality and message integrity. Can a protocol provide message confidentiality without message integrity? Explain with an example.

#### Question 2

Consider an 8-bit block cipher. a. How many distinct input blocks does this cipher have? b. How many possible unique mappings (permutations) are there from input blocks to output blocks?

#### Question 3

Explain why Electronic Codebook (ECB) mode is insecure for encrypting multi-block messages, and demonstrate how Cipher Block Chaining (CBC) resolves this vulnerability.

#### Question 4

In RSA key generation, suppose $p = 7$ and $q = 13$. a. Calculate n and z. b. If public exponent $e = 5$, find the private exponent d such that $e \cdot d \equiv 1 \pmod z$. c. Encrypt plaintext message $m = 4$.

#### Question 5

Explain the difference between a Message Authentication Code (MAC) and a Digital Signature in terms of: a. Underlying cryptographic operations used. b. Support for non-repudiation.

#### Question 6

Explain how a Man-in-the-Middle (MitM) attack can compromise unauthenticated public key distribution, and how X.509 Digital Certificates signed by CAs mitigate this attack.

#### Question 7

In authentication protocol ap4.0, why does Bob send a nonce R to Alice instead of just having Alice encrypt a static secret password? Describe a scenario where omitting the nonce leads to security failure.

#### Question 8

Explain how Pretty Good Privacy (PGP) provides confidentiality, sender authentication, and message integrity for an email message m. Name the key used at each step.

#### Question 9

Compare Transport Layer Security (TLS) and IPsec (Tunnel Mode) across the following dimensions: a. Protocol layer where security operates. b. Operational transparency to end-user applications. c. Endpoints that terminate the security association.

#### Question 10

Contrast a Stateful Packet Filter firewall with a Signature-based Intrusion Detection System (IDS). What specific class of attacks can an IDS catch that a stateful packet filter cannot?

### Complete Answer Key

#### Solution to Question 1

Confidentiality ensures that only the sender and receiver can read the plaintext message. Message integrity ensures the message was not modified in transit. Yes, a protocol can provide confidentiality without integrity: if an adversary modifies ciphertext bits during transmission, a receiver decrypting the corrupted ciphertext will obtain garbled plaintext. Without an integrity check (such as a MAC), the receiver cannot determine whether the garbled text resulted from network error or malicious alteration.

#### Solution to Question 2

a. An 8-bit block cipher has $2^8 = 256$ distinct input blocks. b. The number of possible unique mappings is the number of permutations of 256 blocks, which equals $256! \approx 8.57 \times 10^{506}$ possible keys.

#### Solution to Question 3

In ECB mode, each block is encrypted independently. Identical plaintext blocks yield identical ciphertext blocks, exposing patterns and structure (such as images or repeated headers). CBC mode prevents this by XORing each plaintext block with the preceding ciphertext block (or an IV for the first block) prior to encryption, ensuring identical plaintext blocks produce distinct ciphertexts.

#### Solution to Question 4

a. $n = p \cdot q = 7 \cdot 13 = 91$. $z = (p-1)(q-1) = 6 \cdot 12 = 72$. b. $e \cdot d \equiv 1 \pmod{72} \implies 5d \equiv 1 \pmod{72}$. Testing multiples of 72: $72 \cdot 2 + 1 = 145$. $145 / 5 = 29$. Thus, $d = 29$. c. $c = m^e \bmod n = 4^5 \bmod 91 = 1024 \bmod 91$. Since $91 \times 11 = 1001$, $1024 - 1001 = 23$. Ciphertext $c = 23$.

#### Solution to Question 5

a. MACs use fast symmetric hash operations ($H(m + s)$) with a shared secret key. Digital signatures use asymmetric public-key operations ($D_{K^-}(H(m))$). b. MACs do not support non-repudiation because both Alice and Bob hold key s; either could have generated the MAC. Digital signatures provide non-repudiation because only the sender holds the private key used to sign.

#### Solution to Question 6

Without authentication, an attacker Trudy intercepts public key requests and sends her own public key $K_T^+$ to Alice while posing as Bob. Alice encrypts data using $K_T^+$, allowing Trudy to decrypt, read, modify, re-encrypt under Bob's real public key $K_B^+$, and forward to Bob undetected. CAs mitigate this by digitally signing a binding between Bob's identity and $K_B^+$. Alice verifies the CA's signature using a trusted $K_{\text{CA}}^+$, ensuring $K_B^+$ belongs to Bob.

#### Solution to Question 7

A nonce R ensures freshness. If Alice only encrypts a static password, Trudy can sniff the encrypted password and replay it in a future session to authenticate as Alice. The nonce changes every session, making replayed responses invalid.

#### Solution to Question 8

1. Digital Signature: Alice hashes m and encrypts the digest with her private key $K_A^-$ for authentication/integrity.
2. Data Confidentiality: Alice generates random session key $K_S$ and encrypts $(m, \text{signature})$ with $K_S$ via symmetric encryption.
3. Session Key Encryption: Alice encrypts $K_S$ with Bob's public key $K_B^+$ and appends it to the payload.

#### Solution to Question 9

a. Layer: TLS operates at the Transport/Application layer (Layer 4/7). IPsec Tunnel Mode operates at the Network layer (Layer 3). b. Transparency: TLS requires applications to be written to use TLS sockets (e.g., HTTPS vs HTTP). IPsec is completely transparent to applications; packets are intercepted and secured at Layer 3 automatically. c. Endpoints: TLS terminates at application endpoints (user process to server process). IPsec Tunnel Mode typically terminates at network security gateways/routers.

#### Solution to Question 10

A stateful packet filter inspects Layer 3/4 headers and connection state tables. A signature-based IDS performs Deep Packet Inspection (DPI) into Layer 7 payloads. An IDS can catch application-level attacks (e.g., SQL injection strings OR 1=1, HTTP buffer overflow payloads, or known malware signatures) hidden inside valid, established TCP connections that a firewall would allow.

## 12. Synthesis & Conceptual Discussion Prompts

1. Layering Security: Analyze the trade-offs of deploying security at the Application Layer (PGP), Transport Layer (TLS), and Network Layer (IPsec). Why has no single layer eliminated the need for security at the other layers?
2. Performance vs. Security: Public-key operations are computationally expensive. How do modern hybrid cryptosystems strike a balance between key distribution flexibility and high-throughput data processing?
3. Quantum Threats: Evaluate the impact of quantum computing (specifically Shor's Algorithm) on current asymmetric (RSA) versus symmetric (AES-256) cryptographic algorithms.
4. Firewall Evasion and Encrypted Traffic: As TLS encryption becomes universal across HTTP/2 and HTTP/3, discuss how traditional network-based firewalls and IDSs lose visibility into application payloads, and examine architectural solutions (such as TLS Interception Proxies and Endpoint Detection and Response [EDR]).
5. Zero Trust Architecture: Contrast perimeter-based firewall operational security with modern Zero Trust Network Architecture (ZTNA), evaluating why perimeter firewalls are insufficient for modern cloud environments.

## 13. Comprehensive Technical Glossary

* AES (Advanced Encryption Standard): A symmetric block cipher operating on 128-bit blocks using 128, 192, or 256-bit keys.
* AH (Authentication Header): An IPsec protocol providing data integrity and origin authentication without payload encryption.
* Asymmetric Cryptography: Cryptography utilizing a mathematically paired public key and private key.
* CA (Certification Authority): A trusted third-party entity that validates identities and issues digitally signed X.509 certificates.
* CBC (Cipher Block Chaining): A symmetric block cipher mode where each plaintext block is XORed with the previous ciphertext block before encryption.
* Ciphertext: The scrambled, encrypted output produced by a cryptographic algorithm.
* Confidentiality: Ensuring that transmitted data is readable only by intended recipients.
* DES (Data Encryption Standard): An older 64-bit symmetric block cipher using a 56-bit key, now legacy due to key-length vulnerabilities.
* Digital Signature: A cryptographic digest encrypted with a sender's private key, providing origin authentication, integrity, and non-repudiation.
* DPI (Deep Packet Inspection): Inspection of data payloads beyond Layer 3/4 headers by IDSs/IPSs.
* EAP (Extensible Authentication Protocol): An authentication framework used in IEEE 802.11i networks.
* ESP (Encapsulating Security Payload): An IPsec protocol providing payload confidentiality, integrity, and origin authentication.
* Firewall: A network security device that filters traffic crossing network perimeters based on IP headers and security rules.
* HMAC (Hash-based Message Authentication Code): A specific construction for calculating a MAC involving a cryptographic hash function and a secret key.
* IDS / IPS: Operational devices that monitor traffic for malicious activity and alert (IDS) or drop (IPS) matching packets.
* IKE (Internet Key Exchange): The protocol used to negotiate security parameters and establish SAs in IPsec.
* IPsec: A suite of protocols providing network-layer security services (confidentiality, integrity, authentication) for IP datagrams.
* MAC (Message Authentication Code): A secret-key-based hash tag appended to a message to verify integrity and authenticity.
* MitM (Man-in-the-Middle) Attack: An attack where an adversary intercepts and potentially alters communication between two unsuspecting endpoints.
* Nonce: A random, non-repeating number used in authentication protocols to prevent replay attacks.
* PGP (Pretty Good Privacy): An application-layer security protocol used for securing email messages.
* PKI (Public Key Infrastructure): The framework of CAs, digital certificates, and repositories supporting public key distribution.
* Plaintext: The original, unencrypted readable message or data.
* Replay Attack: An attack where an adversary captures a valid transmission and retransmits it later to execute unauthorized actions.
* RSA: A public-key cryptosystem based on the mathematical difficulty of factoring large composite integers.
* SA (Security Association): A unidirectional logical connection in IPsec defining encryption algorithms and keys.
* SAD (Security Association Database): An IPsec database storing active SA parameters.
* SPD (Security Policy Database): An IPsec database storing filtering and security enforcement rules.
* SPI (Security Parameter Index): A 32-bit integer in an IPsec header identifying the specific SA for a packet.
* Symmetric Cryptography: Cryptography where sender and receiver share the same secret key for encryption and decryption.
* TLS (Transport Layer Security): A protocol providing security (confidentiality, integrity, authentication) over TCP connections.
* X.509: The standard international specification for digital certificates.