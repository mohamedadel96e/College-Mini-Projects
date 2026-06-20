---
created: 2026-06-20T11:06
updated: 2026-06-20T11:06
---

### **Important Definitions**

**1. Number Theory & Mathematical Concepts:**

- **Primitive Root:** A primitive root of a prime number $n$ is a number $a$ (where $a < n$) that gives a unique result for the equation $a^i \bmod n$ for every $i$ from 1 to $(n-1)$.
    
- **Discrete Logarithm:** For any integer $b$ and primitive root $a$ of a prime number $p$, the unique exponent $i$ such that $b = a^i \pmod p$ is referred to as the discrete logarithm of $b$.
    
- **Euler's Totient $\phi(n)$:** The number of positive integers less than $n$ that are relatively prime to $n$ (meaning their greatest common divisor is 1). If $n$ is prime, $\phi(n) = n - 1$.
    

**2. Cryptographic Principles & Mechanisms:**

- **Independent Key Principle:** The principle stating that "different instances of cryptographic primitives should always use independent keys".
    
- **KEM (Key Encapsulating Mechanism):** A mechanism used in hybrid encryption that takes a public key and outputs a ciphertext $c$ and a shared key $k$.
    
- **Collision-Resistant Hash Function:** A hash function where it is infeasible to find any pair of distinct inputs $x$ and $y$ such that $H(x) = H(y)$.
    
- **Digital Signature Scheme:** A scheme defined by three algorithms: `Gen` (outputs public/private keys), `Sign` (takes a private key and message to output a signature $\sigma$), and `Vrfy` (takes a public key, message, and signature to output 1 or 0).
    
- **Certificate Authority (CA):** A trusted third-party entity that issues digital certificates, validating the ownership of public keys used in communications like HTTPS.
    

**3. Blockchain:**

- **Blockchain:** A decentralized, distributed, and immutable digital ledger that records transactions across a network of computers.
    

### **Important Comparisons**

**1. Digital Signatures vs. MACs (Message Authentication Codes)**

This is a critical comparison highlighting why Digital Signatures are the public-key equivalent to MACs:

- **Public Verifiability:** Anyone can verify a digital signature. In contrast, only a holder of the secret key can verify a MAC tag.
    
- **Transferability:** You can forward a digital signature to someone else for them to verify.
    
- **Non-Repudiation:** A signer cannot easily deny issuing a signature. MACs cannot provide non-repudiation because without access to the secret key, a judge cannot verify the tag; and even if the key is correct, the receiver themselves could have generated the MAC tag.
    

**2. Secret Key Cryptography vs. Public Key Cryptography**

- **Key Used:** Secret key cryptography uses the same key for encryption and decryption. Public key cryptography uses different keys for encryption and decryption.
    
- **Speed:** Secret key cryptography is very fast, whereas public key cryptography is slow.
    
- **Key Agreement/Distribution:** Agreeing on the same key between sender and receiver is the biggest problem in secret key systems. Public key cryptography does not require prior key agreement.
    

**3. Proof of Work (PoW) vs. Proof of Stake (PoS)**

These are the two main consensus mechanisms used in Blockchain:

- **Proof of Work (e.g., Bitcoin):** Relies on computational work (CPU effort) to solve a puzzle attached to a block. It consumes large quantities of electricity.
    
- **Proof of Stake (e.g., Ethereum):** Instead of computational work, it chooses the validator randomly, but with a probability associated with their wealth or age (the "stake") in the network.
    

**4. KEM/DEM (Hybrid) vs. Pure PKE Schemes**

- **Message Length:** For very short messages, direct encryption using a Public Key Encryption (PKE) scheme can be the best choice. For anything longer, KEM/DEM (hybrid encryption) is more efficient.
    
- **Efficiency:** PKE consumes a lot of time and space due to computationally intensive mathematical operations and produces larger ciphertexts.