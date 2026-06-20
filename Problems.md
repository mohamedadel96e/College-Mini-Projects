---
created: 2026-06-20T11:06
updated: 2026-06-20T11:06
---

### 1. Diffie-Hellman Key Exchange (From Lecture 6)

The Diffie-Hellman protocol allows two parties (Alice and Bob) to establish a shared secret over an insecure channel.

**The Equations:**

* **Public Parameter Generation:** * Alice computes her public key: $A = g^a \bmod p$
* Bob computes his public key: $B = g^b \bmod p$


* **Shared Secret Computation:**
* Alice computes the key using Bob's public key: $k = B^a \bmod p$
* Bob computes the key using Alice's public key: $k = A^b \bmod p$
* Mathematically, both result in: $k = g^{ab} \bmod p$



**Big Example: Establishing a Shared Secret**

1. **Setup:** Alice and Bob publicly agree on a prime modulus $p = 23$ and a generator (base) $g = 5$.
2. **Private Keys:** * Alice randomly chooses a private key $a = 6$.
* Bob randomly chooses a private key $b = 15$.


3. **Public Keys (Exchange):**
* Alice computes her public key: $A = 5^6 \bmod 23 = 8$. She sends $A=8$ to Bob.
* Bob computes his public key: $B = 5^{15} \bmod 23 = 19$. He sends $B=19$ to Alice.


4. **Computing the Shared Secret:**
* Alice receives $B=19$ and computes: $k = 19^6 \bmod 23 = 2$.
* Bob receives $A=8$ and computes: $k = 8^{15} \bmod 23 = 2$.
*Result:* Both Alice and Bob have successfully established the shared secret key $k = 2$.



---

### 2. ElGamal Encryption (From Lecture 7)

ElGamal is an asymmetric encryption algorithm based on the Diffie-Hellman key exchange.

**The Equations:**

* **Key Generation:**
* Public Key $y$: $y = g^x \bmod p$ (where $x$ is the private key)


* **Encryption (producing ciphertext components $c_1, c_2$ for message $m$):**
* $c_1 = g^k \bmod p$ (where $k$ is a random integer chosen by the sender)
* $c_2 = m \cdot y^k \bmod p$


* **Decryption (recovering message $m$):**
* Step 1 (Find shared secret): $s = c_1^x \bmod p$
* Step 2 (Find inverse): Find $s^{-1}$ such that $(s \cdot s^{-1}) \equiv 1 \bmod p$
* Step 3 (Recover plaintext): $m = c_2 \cdot s^{-1} \bmod p$



**Big Example: Encrypting and Decrypting a Message**

1. **Key Generation:** * Alice chooses prime $p = 23$, generator $g = 5$, and private key $x = 6$.
* She computes her public key $y = 5^6 \bmod 23 = 8$.
* Her public key is $(p=23, g=5, y=8)$.


2. **Encryption:** * Bob wants to send the message $m = 7$ to Alice. He chooses a random integer $k = 3$.
* He computes $c_1 = 5^3 \bmod 23 = 10$.
* He computes $c_2 = 7 \cdot 8^3 \bmod 23 = 21$.
* Bob sends the ciphertext $(10, 21)$ to Alice.


3. **Decryption:**
* Alice receives $(c_1=10, c_2=21)$. She uses her private key $x=6$ to compute $s = 10^6 \bmod 23 = 9$.
* She finds the modular inverse of 9 modulo 23. By testing, $9 \times 18 = 162 \equiv 1 \bmod 23$, so $s^{-1} = 18$.
* She recovers the message: $m = 21 \cdot 18 \bmod 23 = 7$.
*Result:* Alice successfully decrypted Bob's message to get $m=7$.



---

### 3. RSA Encryption (From Lecture 8)

RSA is the most widely used asymmetric algorithm, relying on the difficulty of factoring large prime numbers.

**The Equations:**

* **Key Generation:**
* Modulus: $N = p \cdot q$ (where $p$ and $q$ are large primes)
* Euler's Totient: $\phi(N) = (p-1)(q-1)$
* Private Key Exponent $d$: $e \cdot d \equiv 1 \bmod \phi(N)$ (where $e$ is the public exponent)


* **Encryption:** * $c = m^e \bmod N$
* **Decryption:** * $m = c^d \bmod N$

**Big Example: RSA Key Gen, Encryption, and Decryption**

1. **Key Generation:**
* Choose primes $p = 17$ and $q = 11$.
* Compute $N = 17 \times 11 = 187$.
* Compute $\phi(N) = (16) \times (10) = 160$.
* Choose $e = 7$ (since the greatest common divisor of 7 and 160 is 1).
* Find $d$ such that $7 \cdot d \equiv 1 \bmod 160$. Using the Extended Euclidean Algorithm, $d = 23$ (because $7 \times 23 = 161 = 1 \times 160 + 1$).
* Public Key: $(N=187, e=7)$. Private Key: $(d=23)$.


2. **Encryption:**
* Alice wants to encrypt the message $m = 88$ using Bob's public key.
* $c = 88^7 \bmod 187 = 11$.
* Alice sends $c = 11$ to Bob.


3. **Decryption:**
* Bob receives $c = 11$ and uses his private key $d = 23$.
* $m = 11^{23} \bmod 187 = 88$.
*Result:* Bob successfully recovered the original message 88.



---

### 4. RSA Digital Signatures (From Lectures 8 & 9)

Digital signatures provide integrity, authentication, and non-repudiation by reversing the RSA encryption/decryption role.

**The Equations:**

* **Plain RSA Signature (Insecure by itself):**
* Sign: $\sigma = m^d \bmod N$ (Signing using the sender's private key)
* Verify: $m \stackrel{?}{=} \sigma^e \bmod N$ (Verifying using the sender's public key)


* **Hash-and-Sign Paradigm (Secure):**
* Sign: $\sigma = H(m)^d \bmod N$ (Hash the message first, then sign)
* Verify: $H(m) \stackrel{?}{=} \sigma^e \bmod N$ (Compare the hash of the message with the decrypted signature)



**Big Example: The Hash-and-Sign Paradigm**

1. **Setup:** Alice has the RSA keys generated in the previous example: Public $(N=187, e=7)$, Private $(d=23)$.
2. **Signing:**
* Alice has a large document $m$ she wants to sign.
* First, she passes $m$ through a cryptographic hash function $H$, resulting in a condensed hash value. Let's assume $H(m) = 15$.
* Alice signs the hash: $\sigma = 15^{23} \bmod 187 = 9$.
* Alice sends the document $m$ and the signature $\sigma = 9$ to Bob.


3. **Verification:**
* Bob receives the document $m$ and the signature $\sigma = 9$.
* Bob hashes the document himself using the same agreed-upon hash function to get $H(m) = 15$.
* Bob verifies the signature using Alice's public key ($e=7, N=187$): $\text{Recovered Hash} = 9^7 \bmod 187 = 15$.
*Result:* Since the Recovered Hash (15) matches the $H(m)$ Bob calculated (15), the signature is valid. Alice definitely sent it, and the document was not altered.



---

### 5. Authenticated Encryption: Encrypt-then-Authenticate (From Lecture 5)

When combining Encryption (for privacy) and Message Authentication Codes (MACs, for integrity), the order of operations matters. "Encrypt-then-Authenticate" is the only universally secure paradigm.

**The Equations:**

* **Encryption Step:** $c = Enc'_{K_E}(m)$ (Encrypt the message $m$ using the encryption key $K_E$)
* **Authentication Step:** $t = Mac'_{K_M}(c)$ (Calculate the MAC tag $t$ on the *ciphertext* $c$ using a separate, independent MAC key $K_M$)
* **Final Output:** $Enc_K(m) = \langle c, t \rangle$

**Big Example: Secure Data Transmission**

1. **Setup:** Alice and Bob share two *independent* secret keys to uphold the Independent Key Principle:
* Encryption Key: $K_E = \text{"101010"}$
* MAC Key: $K_M = \text{"111100"}$


2. **Processing by Alice (Sender):**
* Alice wants to send the message $m = \text{"Secret Attack Plan"}$.
* *Step 1 (Encrypt):* She encrypts the message using AES and key $K_E$: $c = Enc'_{\text{101010}}(\text{"Secret Attack Plan"}) = \text{"X9F2B"}$.
* *Step 2 (Authenticate):* She computes a MAC on the *ciphertext* using key $K_M$: $t = Mac'_{\text{111100}}(\text{"X9F2B"}) = \text{"TAG44"}$.
* She sends the pair over the network: $\langle \text{"X9F2B"}, \text{"TAG44"} \rangle$.


3. **Processing by Bob (Receiver):**
* Bob receives $\langle c=\text{"X9F2B"}, t=\text{"TAG44"} \rangle$.
* *Step 1 (Verify):* Before doing any decryption, Bob recalculates the MAC on the received ciphertext $c$ using his copy of $K_M$. He computes $Mac'_{\text{111100}}(\text{"X9F2B"})$ and gets $\text{"TAG44"}$. Because it matches the $t$ he received, he knows the ciphertext was not tampered with.
* *Step 2 (Decrypt):* Since the ciphertext is authentic, Bob confidently decrypts it using $K_E$: $m = Dec'_{\text{101010}}(\text{"X9F2B"}) = \text{"Secret Attack Plan"}$.
*Result:* The communication achieved both perfect privacy and integrity, safe from Chosen-Ciphertext Attacks (CCA).