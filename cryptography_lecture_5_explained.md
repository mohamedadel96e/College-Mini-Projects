# Cryptography Lecture 5 Explained. MACs, CBC-MAC, Authenticated Encryption, and Hash Functions

> Based on the uploaded lecture `Lec_5.pdf`. The notes below simplify the ideas, add exam-style examples, and include solving rules for common question types.

---

## 1) Big Picture of This Lecture

This lecture connects **three major ideas**:

1. **MACs** protect **integrity/authenticity**.  
   They help us detect tampering.
2. **Encryption** protects **confidentiality**.  
   It hides the message.
3. **Hash functions** produce a short fingerprint of data.  
   They are used in integrity checks, signatures, indexing, and many crypto constructions.

### Core theme
A secure system usually needs **both**:
- secrecy, and
- protection against modification.

> **Rule:** Encryption alone does not guarantee integrity.  
> **Rule:** A MAC alone does not hide the message.  
> **Rule:** Authenticated encryption is meant to give both.

---

## 2) MACs for Arbitrary-Length Messages. Quick Recall

The lecture starts by continuing the idea from the previous lecture: how to build a MAC for long messages from a MAC that only works on fixed-size blocks.

A secure construction in the lecture uses:
- a random value `r`
- the total message length `ℓ`
- the block index `i`
- the message block `m_i`

and computes tags like:

```text
t_i = Mac'_k(r || ℓ || i || m_i)
```

Then the output tag is roughly:

```text
(r, t_1, ..., t_d)
```

### Why this works better
Earlier failed ideas allowed attacks such as:
- block reordering
- truncation
- mix-and-match

This improved construction prevents those by making each block tag depend on:
- the random nonce `r`
- the total length `ℓ`
- the block position `i`
- the block itself `m_i`

> **Rule:** When authenticating long messages block by block, the tag must depend on the block content **and** its context, especially order and total length.

### Extra exam-style example
Suppose a message has 3 blocks:

```text
m = (m1, m2, m3)
```

If the MAC was only:

```text
Mac(m) = (Mac'(m1), Mac'(m2), Mac'(m3))
```

then an attacker could reorder the blocks and create:

```text
(m3, m1, m2)
```

with the reordered tags.

That is a valid forgery if verification only checks each block independently.

### How to solve questions of this type
When the exam gives a MAC construction for long messages, always ask:
1. Can I **reorder** blocks?
2. Can I **delete** the last block and still pass verification?
3. Can I **combine** blocks from different messages?
4. Does the tag include **position**, **length**, or **fresh randomness**?

---

## 3) CBC-MAC

CBC-MAC is a MAC construction based on a block cipher or PRF-like primitive.

The lecture shows CBC-MAC as a way to process message blocks sequentially and produce **one final tag**.

### Intuition
Think of CBC-MAC as follows:
- start from an initial value
- mix in the first block
- apply the block cipher
- mix in the second block
- apply the block cipher again
- continue until the last block
- the final output becomes the tag

### Important advantage
CBC-MAC gives a **short single tag**, unlike some earlier constructions that output one tag per block.

### Lecture point
The slide notes that CBC-MAC is efficient and sends only one authentication tag.

### Caveat from the lecture
If the message length is not handled carefully, padding issues can cause problems.

> **Rule:** CBC-MAC is not something you should use casually on arbitrary-length messages unless the construction is carefully specified.

### Extra exam example
Imagine two messages:

```text
m  = (m1, m2)
m' = (m1)
```

If a CBC-based MAC design ignores message length or uses an unsafe extension rule, an attacker may exploit structure between the two computations.

You do not need to memorize every attack detail unless your instructor covered it formally, but you should remember the high-level lesson:

> **Rule:** Message length matters in MAC design.

### How to solve CBC-MAC questions
If the exam asks whether a CBC-MAC-like construction is safe, check:
1. Does it work only for **fixed-length** messages, or arbitrary-length ones?
2. Is message length encoded or otherwise handled safely?
3. Is there one final tag, or are intermediate values exposed?

---

## 4) Why We Need Authenticated Encryption

The lecture says:
- **Encryption** hides the message from the attacker.
- **MACs** prevent an attacker from tampering with the message.

So naturally, we want **both**.

That gives us **authenticated encryption**.

### Intuition
If you only encrypt:
- attacker may not read the message,
- but may still modify it in a meaningful way.

If you only authenticate:
- attacker may not modify it undetected,
- but can still read it.

> **Rule:** Authenticated encryption = confidentiality + integrity.

---

## 5) Building Authenticated Encryption. Attempt 1

The lecture first tries:

```text
Enc_k(m) = (Enc'_k(m), Mac'_k(m))
```

where:
- `Enc'` is CPA-secure encryption
- `Mac'` is a secure MAC
- **the same key is used for both**

### Problem
This is unsafe because it violates the **independent key principle**.

The lecture explicitly highlights:

> **Different instances of cryptographic primitives should always use independent keys.**

### Why this is bad
If encryption and authentication both use the same key, information from one part may leak or interact with the other part in a dangerous way.

The slide shows a CPA attack that exploits reuse of the same secret key.

> **Rule:** Never reuse the same secret key for different cryptographic jobs unless the scheme is specifically designed for that.

### Extra exam example
Bad idea:

```text
k = one secret key
ciphertext = Enc_k(m)
tag = Mac_k(m)
```

Better idea:

```text
K = (K_E, K_M)
ciphertext = Enc_{K_E}(m)
tag = Mac_{K_M}(m)
```

### How to solve questions of this type
If a construction combines two primitives, immediately check:
1. Are the keys independent?
2. Is one primitive accidentally revealing information used by the other?
3. Did the question explicitly say `K = (K_E, K_M)` or did it reuse one key?

If the same key is reused across roles, be suspicious.

---

## 6) Independent Key Principle

The lecture gives a dedicated slide for this idea:

> **Different instances of cryptographic primitives should always use independent keys.**

This is one of the most important exam rules from this lecture.

### Easy interpretation
Do not do this:
- same key for encryption and MAC
- same key for two different protocol components
- same key for unrelated security goals

Do this instead:
- derive or assign separate keys for separate tasks

### Memory trick
Think of keys like passwords for different accounts.  
Using the same password everywhere is convenient, but one leak can break everything.

---

## 7) Building Authenticated Encryption. Attempt 2. Encrypt-and-Authenticate

The lecture then tries a more careful design with independent keys:

```text
Enc_K(m) = (Enc'_{K_E}(m), Mac'_{K_M}(m))
```

This is often called **Encrypt-and-Authenticate**.

### At first glance
This looks better because:
- encryption key and MAC key are different
- secrecy and integrity are both present in some form

### But it still fails in general
Why? Because the MAC is computed on the **plaintext**.

If the MAC is deterministic, then it may leak information about the message.

The lecture gives the exact idea:
- attacker chooses `m0, m1`
- gets a challenge ciphertext containing `Mac_{K_M}(m_b)`
- asks for encryption of `m0`
- compares tags
- if tags match, attacker learns which message was encrypted

So privacy is broken.

> **Rule:** A secure MAC does not promise to hide the message.

That is the key reason this attempt fails.

### Extra exam example
Suppose:

```text
m0 = "YES"
m1 = "NO"
```

If the ciphertext contains:

```text
tag = Mac_{K_M}(m_b)
```

and the attacker can obtain `Mac_{K_M}("YES")` from another query, then comparing tags may reveal whether `m_b = "YES"`.

### Exam conclusion
> **Rule:** Encrypt-and-Authenticate does **not** guarantee CPA security in general.

### How to solve these questions
Whenever a scheme outputs something like:

```text
( encrypted_message , MAC_of_plaintext )
```

ask:
1. Does the MAC reveal equality of plaintexts?
2. Can the attacker compare tags of chosen messages?
3. Does any part of the output directly depend on the plaintext in a deterministic way?

If yes, privacy may fail.

---

## 8) Building Authenticated Encryption. Attempt 3. Authenticate-then-Encrypt

The lecture then tries:

```text
t = Mac'_{K_M}(m)
Enc_K(m) = Enc'_{K_E}(m || t)
```

This is called **Authenticate-then-Encrypt**.

### Important lecture note
The slide says:
- used in SSL/TLS
- not generically secure
- easy to make implementation mistakes
- example: Lucky13-style issues on TLS

### Why it can fail
The main danger is that the receiver may:
1. decrypt first,
2. then check authentication.

If the system reveals any information during decryption, like:
- padding error
- format error
- different timing
- different error messages

then the attacker may learn information from those side effects.

This becomes a kind of **decryption oracle** or validation oracle.

### Lecture bad case
The lecture gives a bad example using an error-correcting code and shows that repeated queries can reveal information bit by bit.

The exact algebra is less important than the lesson:

> **Rule:** If a system decrypts before authenticating and leaks anything about the result, it may be attacked.

### Extra exam example
Suppose the receiver responds with:
- `"padding error"` if the decrypted data is malformed
- `"MAC error"` if padding is fine but tag is wrong

Then the attacker can modify ciphertexts and use the different responses to learn the plaintext gradually.

That is the same high-level idea behind padding-oracle style attacks.

### How to solve questions of this type
If the exam shows a scheme where the receiver decrypts first, ask:
1. Is the scheme giving the attacker any feedback after decryption?
2. Could different errors leak information?
3. Does authenticity get checked only **after** decryption?

If yes, the scheme may be vulnerable even if the formulas look reasonable.

---

## 9) Building Authenticated Encryption. Attempt 4. Encrypt-then-Authenticate

This is the successful construction from the lecture.

The scheme is:

```text
c = Enc'_{K_E}(m)
Enc_K(m) = (c, Mac'_{K_M}(c))
```

So we:
1. encrypt the message first
2. compute a MAC on the **ciphertext**
3. send both

The lecture says this one is secure, assuming:
- the encryption is CPA-secure
- the MAC is strongly secure
- keys are independent

### Why it works better
Now the receiver can:
1. check the MAC on the ciphertext first
2. only decrypt if the MAC is valid

This blocks many decryption-oracle style attacks.

> **Rule:** The clean, standard lesson is: **Encrypt-then-Authenticate** is the right composition.

### Extra exam example
Correct structure:

```text
c = Enc_{K_E}(m)
t = Mac_{K_M}(c)
send (c, t)
```

Verification:
1. verify `t` against `c`
2. if valid, decrypt `c`
3. otherwise reject immediately

### Exam comparison table

| Construction | General verdict |
|---|---|
| Same key for Enc and MAC | Unsafe |
| Encrypt-and-Authenticate | Not secure in general |
| Authenticate-then-Encrypt | Not secure in general |
| Encrypt-then-Authenticate | Secure under the right assumptions |

### Fast solving rule
If you see a composition question, check in this order:
1. Are there **independent keys**?
2. Is the MAC taken on the **plaintext** or on the **ciphertext**?
3. Does the receiver **authenticate before decrypting**?

Best answer pattern:
- MAC on ciphertext + verify before decrypting + independent keys = strong sign of correctness.

---

## 10) Hash Functions

The lecture then moves to **hash functions**.

### Informal idea
A hash function takes a message of arbitrary length and returns a short fixed-size output.

```text
h = H(M)
```

### What a hash is like
It is a **fingerprint** of the message.

But unlike encryption:
- it is public
- it is not meant to be reversible
- it compresses data

### Lecture point
The slide emphasizes:
- long input `x`
- short output `y`
- `H(x) = y`

> **Rule:** A hash function compresses input to a fixed-length output.

### Extra example
```text
H("hello") = a short digest
H("hello!") = a different digest
```

A tiny change in input should usually change the hash a lot.

---

## 11) Pigeonhole Principle and Hash Collisions

The lecture uses the **pigeonhole principle**:

> You cannot fit 10 pigeons into 9 pigeonholes.

### Crypto meaning
If a hash compresses a very large input space into a smaller output space, then:

> **Collisions must exist.**

That means there must be different inputs `x != y` such that:

```text
H(x) = H(y)
```

### Important nuance
Hash security does **not** mean collisions do not exist.  
They **must** exist if compression happens.

Hash security means:

> It should be computationally hard to **find** such collisions.

### Extra exam example
Suppose a toy hash outputs only one digit `0..9`.  
But there are infinitely many input strings.

Then many different messages must share the same output digit.

### How to solve questions of this type
If the exam asks:
- “Why must collisions exist?”

Answer using:
1. input space is larger than output space
2. function compresses many inputs into fewer outputs
3. by the pigeonhole principle, at least two inputs map to the same output

---

## 12) Classical Applications of Hash Functions

The lecture mentions hash tables and the idea that a “good” hash function should yield “few collisions.”

### Important distinction
Be careful here:
- In data structures, “good hash” often means practical distribution for fast lookup.
- In cryptography, we need stronger security properties, especially collision resistance and one-way behavior.

### Hash function properties in practice
A hash function:
- makes a fingerprint of a file/message/data
- condenses variable-length input to fixed-size output
- is typically public

---

## 13) Collision Resistance

The lecture’s intuition:

> It should be hard for a computationally bounded attacker to find **any pair** `x, x'` such that

```text
H(x) = H(x')
```

### Important correction of intuition
Since collisions must exist, the real goal is not “no collisions.”  
The goal is:

> **No efficient attacker should be able to find one.**

### Lecture warning
The lecture points out that if we let `H(x) = x`, then it would be perfectly collision-free, but it would not compress anything. So a real hash function needs compression.

> **Rule:** Collision resistance only makes sense when the hash output is shorter than the input domain in a meaningful way.

### Extra exam example
Toy bad hash:

```text
H(x) = last 4 bits of x
```

Then finding collisions is easy.
For example:

```text
x  = 00010011
x' = 10110011
```

Both end in `0011`, so they collide.

### How to solve collision-resistance questions
If the exam asks whether a hash is collision resistant:
1. Look for an easy way to construct two different messages with the same output.
2. If the output ignores a lot of input bits, it is probably easy to collide.
3. If the output is just a tiny checksum, collision resistance is usually weak.

---

## 14) Requirements for Hash Functions

The lecture lists these requirements:

1. It can be applied to messages of any size.  
2. It produces fixed-length output.  
3. It is easy to compute `H(M)`.  
4. Given `h`, it is infeasible to find `x` such that `H(x)=h`. This is the **one-way property**.  
5. Given `x`, it is infeasible to find `y != x` such that `H(y)=H(x)`. This is often called **weak collision resistance** or **second-preimage resistance** in many contexts.  
6. It is infeasible to find any pair `x, y` such that `H(y)=H(x)`. This is **strong collision resistance**.

### Easy understanding
There are three popular “hardness” ideas to separate:

#### A) Preimage resistance
Given hash output `h`, find a message `x` such that `H(x)=h`.

#### B) Second-preimage resistance
Given a message `x`, find another message `y != x` such that:

```text
H(y)=H(x)
```

#### C) Collision resistance
Find **any** two distinct messages `x != y` such that:

```text
H(x)=H(y)
```

### Extra exam example
Suppose the examiner gives:

```text
H(x) = x mod 100
```

Then:
- preimages are easy. For `h=37`, choose `x=37`.
- second preimages are easy. For `x=137`, choose `y=37`.
- collisions are easy. `25` and `125` collide.

So this is not cryptographically secure.

### How to solve these questions
When the question says “Given `h`, find `x`”, think **preimage**.  
When it says “Given `x`, find another `y`”, think **second preimage**.  
When it says “Find any two messages”, think **collision**.

---

## 15) Collision Experiment

The lecture formalizes collision resistance as a security game.

High-level idea:
1. A random setup or seed is generated.
2. The attacker outputs two messages `x1, x2`.
3. The attacker wins if:

```text
H^s(x1) = H^s(x2)
```

with `x1 != x2`.

The lecture notes that the seed is not a secret encryption key in the usual sense. It is just part of the hash family setup.

### What to remember for the exam
You usually do not need to memorize the full symbolic game unless your instructor expects it. What matters is the meaning:

> **Rule:** A hash family is collision resistant if no PPT attacker can find a collision except with negligible probability.

---

## 16) Hash Functions in Practice

The lecture lists common hash families.

### MD5
- Developed in 1991
- 128-bit output
- Collisions found in 2004
- **Should no longer be used**

### SHA-1
- Introduced in 1995
- 160-bit output
- Very common historically
- Collision found in 2017
- Migration to SHA-2 is standard

### SHA-2
- Introduced in 2001
- Versions with 224, 256, 384, and 512-bit outputs
- No major practical collision break like MD5/SHA-1

### SHA-3 / Keccak
- Result of a public competition
- Very different design from SHA-1/SHA-2
- Supports 224, 256, 384, and 512-bit outputs

> **Rule:** For modern practical use, SHA-2 and SHA-3 are the safe families to remember.  
> **Rule:** MD5 and SHA-1 are historically important but no longer appropriate for collision-sensitive security use.

### Exam note
If the exam asks “Which should no longer be used because of collisions?” the expected answers are often:
- MD5
- SHA-1

---

## 17) Hash-and-MAC

The last slide sketches a “Hash-and-MAC” idea.

### Basic idea
Instead of MACing the entire long message directly, one can:
1. compute a hash of the message
2. authenticate the hash using a MAC

So roughly:

```text
h = H(M)
t = Mac_k(h)
```

Receiver recomputes the hash and verifies the MAC.

### Why people do this
It can be faster and more practical for long messages.

### Important caution
This only works safely when the overall construction is carefully designed with secure components.

> **Rule:** Hashing before a MAC can be useful, but do not invent your own crypto design casually.

---

## 18) Most Important Rules From This Lecture

### Composition rules
> **Rule 1:** Encryption protects secrecy, MAC protects integrity.  
> **Rule 2:** Same key should not be reused for different cryptographic tasks.  
> **Rule 3:** Encrypt-and-Authenticate is not secure in general.  
> **Rule 4:** Authenticate-then-Encrypt is not secure in general.  
> **Rule 5:** Encrypt-then-Authenticate is the safe composition to remember.

### MAC rules
> **Rule 6:** Long-message MACs must bind each block to its position and total length.  
> **Rule 7:** CBC-MAC needs careful use, especially regarding message lengths.

### Hash rules
> **Rule 8:** Hash functions compress long inputs into fixed-length outputs.  
> **Rule 9:** Collisions must exist by the pigeonhole principle.  
> **Rule 10:** Security means collisions are hard to **find**, not impossible to exist.  
> **Rule 11:** MD5 and SHA-1 are not suitable for modern collision-sensitive security use.  
> **Rule 12:** SHA-2 and SHA-3 are the modern families to remember.

---

## 19) Exam-Oriented Solving Guide

## Type A. “Which authenticated-encryption composition is secure?”

### Step-by-step
1. Check whether encryption and MAC use **independent keys**.
2. Check whether the MAC is on the **plaintext** or **ciphertext**.
3. Check whether verification happens **before decryption**.

### Fast answer pattern
- MAC on plaintext -> suspicious.  
- Decrypt before authenticate -> suspicious.  
- MAC on ciphertext + verify first + independent keys -> likely correct.

---

## Type B. “Why is this MAC for long messages broken?”

### Test these attacks
- block reordering
- truncation
- mix-and-match from multiple messages
- missing length binding
- missing block index binding

### Fast answer pattern
If the tag on each block does not depend on its place in the message, there is probably a forgery attack.

---

## Type C. “Why must collisions exist?”

### Answer template
Because the hash maps a larger set of possible inputs to a smaller set of outputs. By the pigeonhole principle, at least two different inputs must map to the same output.

---

## Type D. “What hash property is being asked?”

### Decode the wording
- Given `h`, find `x` -> **preimage resistance**
- Given `x`, find `y != x` with same hash -> **second-preimage resistance**
- Find any `x, y` with same hash -> **collision resistance**

---

## Type E. “Is this practical hash secure?”

### Quick checks
- tiny output size -> likely insecure
- ignores many input bits -> likely insecure
- simple checksum/modulo trick -> likely insecure
- MD5 or SHA-1 in modern security context -> bad choice

---

## 20) Mini Review Questions

Try answering these yourself before checking the short answers.

### Q1
Why does Encrypt-and-Authenticate fail in general?

**Answer:** Because the MAC on the plaintext may leak information about the message, so privacy can fail.

### Q2
What is the main danger of Authenticate-then-Encrypt?

**Answer:** The receiver may decrypt first and leak information through errors, timing, or formatting checks.

### Q3
Which composition is the safe one to remember?

**Answer:** Encrypt-then-Authenticate.

### Q4
Do collisions exist for real hash functions?

**Answer:** Yes. They must exist if the function compresses inputs. The goal is that they are hard to find.

### Q5
What modern hash families should you remember as practical choices?

**Answer:** SHA-2 and SHA-3.

---

## 21) Final Summary

This lecture teaches a very important crypto design lesson:

- Secrecy and integrity are different goals.  
- Combining secure parts incorrectly can still produce an insecure system.  
- The order of encryption and authentication matters.  
- The best general lesson to remember is **Encrypt-then-Authenticate**.  
- Hash functions compress data and support many crypto tasks, but their security is about resisting efficient attacks such as collision finding.

If you remember only a few things from this lecture, remember these four:

1. **Independent keys matter.**  
2. **Encrypt-and-Authenticate is not enough in general.**  
3. **Authenticate-then-Encrypt can leak through decryption behavior.**  
4. **Encrypt-then-Authenticate is the correct paradigm to remember.**

