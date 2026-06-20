
### Question 11.14

**Question**
Consider a modified padded RSA encryption: messages have length exactly $||N||/2$. To encrypt, first compute $\hat{m} := 0\text{x}00 || r || 0\text{x}00 || m$ where $r$ is a uniform string. Compute ciphertext $c := [\hat{m}^e \bmod N]$. The receiver decrypts and returns an error if the format is not exactly $0\text{x}00$ followed by arbitrary bits, followed by $0\text{x}00$.
Show that this scheme is not CCA-secure. Why is it easier to construct an attack here than on PKCS#1 v1.5?

**Answer**
This scheme is not CCA-secure because the attacker can multiply the ciphertext by $2^e$ to shift the plaintext bits, bypassing the format check 50% of the time.

**Explanation**

* Because of RSA's mathematical properties, multiplying a ciphertext by $2^e$ effectively multiplies the underlying plaintext by 2.
* In binary, multiplying by 2 is just a left bit-shift.
* The original format is `0x00 || r1 ... || 0x00 || m`.
* When you shift this left by 1 bit, the first bit of the random string ($r_1$) gets pushed into the leading `0x00` block.
* If $r_1$ was `1`, the leading block is no longer `0x00`. The oracle sees the broken format and returns an **Error**.
* If $r_1$ was `0` (which happens 50% of the time), the leading block stays `0x00`. The format is still valid! The oracle decrypts it and gives you the shifted message.
* By seeing if an error occurs or reading the shifted message, the attacker easily determines which message was encrypted, breaking CCA security.
* **Why it fails on PKCS#1 v1.5:** That standard starts with `0x00 || 0x02`. If you shift that left (multiply by 2), it becomes `0x00 || 0x04`. The format is *always* ruined, the oracle *always* returns an error, and the attacker learns nothing.

**Steps to solve similar questions**

* Look at the exact padding format of the plaintext.
* Think about RSA's homomorphic property: what happens if you multiply the ciphertext by a small constant like $2^e$?
* Determine how that multiplication alters the plaintext format (e.g., bit shifting).
* Check if the decryption oracle will accept or reject the newly formatted plaintext, and what that reveals to the attacker.

---

### Question 11.15

**Question**
Consider an RSA encryption scheme where a user encrypts a message $m$ by computing $\hat{m} := H(m) || m$ and outputting the ciphertext $c := [\hat{m}^e \bmod N]$. The receiver verifies the correct form before outputting $m$. Prove or disprove that this scheme is CCA-secure if H is a random oracle.

**Answer**
Disprove. This scheme is not even CPA-secure, let alone CCA-secure.

**Explanation**

* Look closely at the encryption formula: $\hat{m} := H(m) || m$.
* There is absolutely no randomness (like a random string $r$ or $k$) used in this process.
* Because the hash function $H$ is deterministic, encrypting the exact same message will always produce the exact same ciphertext.
* Any deterministic public-key encryption scheme is insecure against Chosen-Plaintext Attacks (CPA).
* An attacker can just take the public key, manually encrypt the two possible guess messages themselves, and see which result perfectly matches the intercepted ciphertext.

**Steps to solve similar questions**

* Check the encryption formula for a random variable (e.g., $r$).
* If there is no random variable, the encryption is deterministic.
* State immediately that deterministic public-key encryption cannot be CPA-secure.

---

### Question 12.5

**Question**
In encoded RSA signatures, the signature is $\sigma := [enc(m)^d \bmod N]$.
(a) How is verification performed?
(b) Why does this prevent the "no-message attack"?
(c, d, e) Show that encoded RSA is insecure if $enc(m) = m \cdot \alpha$ (which happens if you just pad with zeros like $0...0 || m || 0...0$). Show a forgery attack for an arbitrary $e$.

**Answer**
(a) Verification is done by checking if $\sigma^e \equiv enc(m) \bmod N$.
(b) It prevents the attack because a random signature won't match the strict formatting rules of $enc(m)$.
(c, d, e) It is insecure because the padding acts as a mathematical multiplier, allowing attackers to forge signatures by multiplying/dividing valid signatures.

**Explanation**

* **Verification (a):** The verifier takes the signature, raises it to the public exponent $e$, and checks if the result exactly matches the publicly known encoded version of the message.
* **No-Message Attack (b):** In a no-message attack, an attacker picks a random signature $\sigma$ and hopes it works for *some* message. But $\sigma^e$ will just look like random garbage. Because $enc(m)$ forces the plaintext to have a very specific format (like starting with zeros), the random garbage will almost never match that format, so the attack fails.
* **The Forgery Attack (e):** If the encoding is just adding zeros, it mathematically equals multiplying the message by a constant $\alpha$.
* To forge a signature for the message `8`:
1. The attacker asks the oracle to sign message `4`, getting $\sigma_4$.
2. The attacker asks the oracle to sign message `2`, getting $\sigma_2$.
3. The attacker calculates a forged signature: $\sigma_{forged} = (\sigma_4^2 / \sigma_2) \bmod N$.


* Why it works: $(\sigma_4^2 / \sigma_2)^e = enc(4)^2 / enc(2) = (4\alpha)^2 / (2\alpha) = 16\alpha^2 / 2\alpha = 8\alpha = enc(8)$. The forged signature perfectly verifies for the message `8`!

**Steps to solve similar questions**

* Identify if the encoding/padding function can be represented as a simple mathematical operation (like multiplication by a constant).
* Use the homomorphic property of RSA ($\sigma_1 \cdot \sigma_2 = \sigma_{m1 \cdot m2}$).
* Build a combination of signed messages (using multiplication, division, or squaring) that algebraically resolves to the target message you want to forge.

---

### Question 12.9

**Question**
A strong one-time-secure signature scheme means that given a signature $\sigma'$ on message $m'$, it is infeasible to output a *new* valid signature $\sigma$ for that same message $m'$.
(a) Give a formal definition.
(c) Construct a strong one-time-secure signature scheme.

**Answer**
(a) The adversary wins if they output $(m, \sigma) \neq (m', \sigma')$ where $Vrfy_{pk}(m, \sigma) = 1$.
(c) Use a standard one-time signature, but sign the hash of the message $H(m)$ using a collision-resistant hash function.

**Explanation**

* **Formal Definition (a):** The attacker is given the public key and is allowed to ask the oracle to sign exactly *one* message, $m'$. The oracle returns $\sigma'$. The attacker wins if they can produce a valid pair $(m, \sigma)$ that is not perfectly identical to $(m', \sigma')$. (Meaning they either forged a signature for a new message, or found a second, different signature for the old message).
* **Construction (c):** To make a scheme "strong", we incorporate a collision-resistant hash function.
* We generate a hash key $s$ as part of the public key.
* Instead of signing the message directly, we sign $H^s(m)$.
* If an attacker tries to find a second, different signature $\sigma$ for the same message $m$, they are mathematically forced to find a collision in the hash function.
* Because we defined the hash function as collision-resistant, this is computationally impossible, proving the scheme is strong.

**Steps to solve similar questions**

* Understand the difference between standard security (cannot forge a *new message*) and strong security (cannot forge a *new signature* for an old message).
* When asked to build a strong scheme, almost always introduce a Collision-Resistant Hash Function into the signing process.
* Argue that breaking the strong security requirement would require finding a hash collision, which is impossible by definition.