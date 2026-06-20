---
created: 2026-04-03T00:26
updated: 2026-06-18T12:38
---
# Cryptography Lecture 4 Explained

Cleaned formatting version based on Lec_4.pdf.

---

# 1) Big Picture of This Lecture

This lecture connects several important ideas in modern cryptography:

1. **PRF security**
2. **PRP security**
3. **Modes of encryption** like **ECB, CBC, CTR**
4. **CCA security** and why CPA is not always enough
5. **Integrity** and **Message Authentication Codes (MACs)**
6. Common attacks like:
   - ECB pattern leakage
   - ciphertext modification attacks
   - replay attacks
   - padding oracle attacks
   - bad MAC constructions for long messages

---

# 2) PRF. Pseudorandom Function

## Simple idea
A **PRF** is a keyed function that behaves, to an attacker, like a truly random function.

So if the attacker queries the function on inputs of their choice, they should **not be able to tell** whether:
- they are talking to a real function `F_k`, or
- they are talking to a truly random function `R`

This is the main idea shown in the **PRF security definition and game** in the lecture.

## Formal form
A keyed function:

```text
F: {0,1}^n x {0,1}^n -> {0,1}^n
```

means:
- first input = secret key `k`
- second input = message or block `x`
- output = `n`-bit value

---

## PRF Security Intuition
The attacker is allowed to ask for outputs on many inputs.

If the attacker still cannot distinguish the real oracle from a random oracle with probability noticeably better than `1/2`, then the function is secure.

### Exam Rule
**If a function "looks random" to every efficient attacker, then it is a PRF.**

### What to remember in an exam
Whenever you see PRF security, think of this question:

> "Can the attacker distinguish `F_k(.)` from a truly random function?"

If the answer is **no**, then it is secure.

---

## Extra Example
Suppose:

```text
F_k(x) = AES_k(x)
```

If AES is modeled as a strong secure block cipher, then it is often treated like a PRF on fixed-size blocks.

---

## How to solve PRF questions
### If the question asks: "Is this a PRF?"
Check these points:
1. Is there a **secret key**?
2. For the same key and same input, is the output deterministic?
3. Does it avoid obvious structure?
4. Can an attacker distinguish it from random with a simple test?

If you can build a simple distinguisher, then it is **not** a secure PRF.

---

# 3) PRG vs PRF

The lecture states that it is easy to construct a **PRG from a PRF**, while constructing a PRF from a PRG is possible but trickier.

## Simple intuition
- **PRG**: stretch short randomness into longer pseudorandom output
- **PRF**: answer many input queries in a way that looks random

### Memory rule
- PRG = stretch randomness
- PRF = answer function queries pseudorandomly

---

# 4) PRP. Pseudorandom Permutation

## Simple idea
A **PRP** is like a PRF, but with one extra property:

> It must be **invertible**.

The lecture defines a pseudorandom permutation as a keyed function that is invertible and still looks random without the secret key.

## Why invertibility matters
In block encryption, we need to:
- encrypt using `F_k(x)`
- decrypt using `F_k^{-1}(x)`

So for many block ciphers, we want a **PRP**, not just any PRF.

---

## PRF vs PRP

| Concept | PRF | PRP |
|---|---|---|
| Looks random | Yes | Yes |
| Invertible | Not required | Required |
| Used in block ciphers | Sometimes | Very common |

### Exam Rule
**Every PRP behaves like a structured PRF, but not every PRF is a PRP because a PRF may not be invertible.**

---

# 5) Random Functions vs Random Permutations

The lecture compares:
- random functions
- random permutations

A random function may map multiple inputs to the same output.
A random permutation is a one-to-one rearrangement of all possible inputs.

## Exam shortcut
- Function: collisions are allowed
- Permutation: collisions are not allowed

---

# 6) Modes of Encryption

This is one of the most important parts of the lecture.

When we have a block cipher that works on only one block, we need a **mode of operation** to encrypt long messages.

The lecture discusses:
- **ECB**
- **CBC**
- **CTR**

---

# 7) ECB Mode. Electronic Codebook

The lecture defines ECB as encrypting each block independently:

```
Enc_k(m1,..., mt) = (F_k(m1),..., F_k(mt))
```

This means each plaintext block is encrypted separately using the same key.

## Main problem
ECB is **deterministic**.

So:
- same plaintext block -> same ciphertext block
- patterns remain visible

This is exactly why the lecture shows the famous **encrypted penguin** image. You can still recognize the penguin after ECB encryption, which means the encryption is leaking structure.

---

## The Penguin Principle
If you can still see the penguin after encryption, something is very wrong with the scheme.

### Exam Rule
**ECB is not CPA-secure because encryption is deterministic and leaks repeated patterns.**

---

## Extra Example
Suppose a message is split into blocks:

```text
m = [HELLO][HELLO][BYE!!]
```

In ECB:

```text
c = [F_k(HELLO)][F_k(HELLO)][F_k(BYE!!)]
```

The first two ciphertext blocks are identical.

So an attacker learns that the first two plaintext blocks were also identical.

---

## How to solve ECB exam questions
If asked whether ECB is secure, mention:
1. It is deterministic.
2. Equal plaintext blocks produce equal ciphertext blocks.
3. Therefore it leaks patterns.
4. So it is **not CPA-secure**.

---

# 8) CBC Mode. Cipher Block Chaining

CBC improves security by chaining blocks together.

Instead of encrypting each block directly, each plaintext block is first XORed with the previous ciphertext block, starting from an **IV**.

## Encryption idea
```text
c1 = F_k(m1 XOR IV)
c2 = F_k(m2 XOR c1)
c3 = F_k(m3 XOR c2)
```

This structure is shown in the lecture's CBC diagram.

## Why CBC helps
Even if two plaintext blocks are the same, their inputs to the block cipher may differ because of chaining.

So repeated plaintext does **not automatically** create repeated ciphertext.

### Exam Rule
**CBC with a proper random IV can achieve CPA security if the underlying block cipher behaves like a secure PRP.**

---

## Extra Example
Suppose:
- `m1 = m2 = SAME`
- but `IV!= c1`

Then:
- first input = `SAME XOR IV`
- second input = `SAME XOR c1`

These are usually different.

So the ciphertext blocks are different.

---

## Common exam trap
If the IV is reused badly or chosen predictably in the wrong setting, security can fail.

So always mention the IV.

### How to solve CBC questions
When asked how CBC works:
1. Start with IV.
2. XOR plaintext block with previous ciphertext block.
3. Encrypt the result.
4. Repeat.

When asked why it is better than ECB:
- because it hides patterns by mixing each block with previous ciphertext.

---

# 9) CTR Mode. Counter Mode

CTR mode turns a block cipher into something that behaves like a stream cipher.

The lecture shows CTR as encrypting counters and XORing them with plaintext blocks.

## Encryption idea
```text
keystream1 = F_k(ctr)
keystream2 = F_k(ctr + 1)
keystream3 = F_k(ctr + 2)

c1 = m1 XOR keystream1
c2 = m2 XOR keystream2
c3 = m3 XOR keystream3
```

## Why CTR is nice
- encryption can be parallelized
- decryption can be parallelized
- very efficient

### Exam Rule
**CTR mode is CPA-secure if the underlying primitive is a secure PRF or PRP and counters are not reused with the same key.**

---

## Dangerous mistake in CTR
If the same counter sequence is reused with the same key for two messages:

```text
c1 = m1 XOR s
c2 = m2 XOR s
```

then:

```text
c1 XOR c2 = m1 XOR m2
```

That leaks information.

### Extra Example
If one plaintext is known, the other can be recovered.

So never reuse the same keystream.

---

## How to solve CTR questions
Check:
1. Is there a fresh counter or nonce?
2. Are counters unique under the same key?
3. Is encryption done by XOR with a generated keystream?

If yes, it is likely CTR-style and can be secure.

---

# 10) Chosen Ciphertext Attacks. CCA

The lecture explains that **CPA security is not enough** when the attacker can also obtain decryptions of ciphertexts of their choice.

## Intuition
In CCA:
- attacker can ask for encryptions
- attacker can also ask for decryptions
- except they cannot directly ask to decrypt the exact challenge ciphertext

This is much stronger than CPA.

---

## Why CCA matters
Real systems sometimes reveal partial information when decrypting:
- "invalid padding"
- "bad format"
- "wrong message"
- different behavior after decryption

That is enough to attack many schemes.

### Exam Rule
**CCA security is stronger than CPA security.**

The lecture explicitly states that CCA security is strictly stronger than CPA security.

---

# 11) CPA does not imply CCA

The lecture gives an example where a scheme is CPA-secure but not CCA-secure because ciphertexts are malleable.

## Key idea
If the attacker can modify ciphertext in a controlled way and then query the decryption oracle, they may learn the hidden bit or plaintext.

### Memory rule
- CPA only protects secrecy against chosen plaintexts
- CCA also protects secrecy against chosen ciphertext interaction

---

# 12) Malleability

A scheme is **malleable** if an attacker can change ciphertext and cause a predictable change in the plaintext.

The lecture shows an example using XOR-based encryption where changing the ciphertext by XORing with `y` changes the decrypted plaintext from `m` to `m XOR y`.

## Important idea
If:

```text
c = Enc_k(m)
```

and attacker builds:

```text
c' = c XOR y
```

then decryption may become:

```text
m' = m XOR y
```

This is very dangerous.

---

## Extra Example
Suppose plaintext contains:

```text
amount = 050
```

An attacker flips some bits in ciphertext and changes plaintext after decryption into:

```text
amount = 950
```

The attacker may not know the whole message, but may still control meaningful parts.

### Exam Rule
**Malleability is one major reason why CPA-secure encryption may fail to be CCA-secure.**

---

# 13) Padding Oracle Attack

The lecture presents a real-world style attack on block ciphers using padding validity checks.

## Setup
Many block ciphers require message length to be a multiple of the block size.

So padding is added.

For example, with PKCS-style padding:
- `01` means 1 byte padding
- `02 02` means 2 bytes padding
- `03 03 03` means 3 bytes padding

If a server tells the attacker whether padding is valid or invalid, that leaks information.

---

## Why this is dangerous
The attacker modifies ciphertext blocks and watches the server response:
- valid padding
- invalid padding

That response acts like a decryption hint.

Over many tries, the attacker can recover plaintext byte by byte.

The lecture notes attack complexity like:
- up to `L` tries to learn number of padding bytes
- up to `256` tries per plaintext byte in a byte-oriented setting.

---

## Exam Rule
**Padding oracle attacks show why error messages and decryption behavior must not reveal useful information.**

---

## How to solve padding oracle questions
When asked why it works, mention:
1. Block cipher mode uses padding.
2. Receiver leaks whether padding is valid.
3. Attacker changes ciphertext.
4. Valid or invalid response reveals plaintext information.

---

# 14) What Does Secure Information Mean?

The lecture reminds us that secure communication needs more than secrecy. It includes:
- **Confidentiality**. only intended receiver can read
- **Integrity / Authenticity**. message really came from claimed sender and was not altered

### Exam Rule
- Encryption alone may give confidentiality.
- It does **not automatically** give integrity.

---

# 15) Message Authentication Codes. MACs

This is the second major part of the lecture.

## Goal of a MAC
A MAC is used to protect **integrity and authenticity**, not secrecy.

The lecture says:
- CPA-secure encryption focuses on secrecy
- MACs focus on integrity
- CCA-secure encryption requires both secrecy and integrity ideas together

---

## MAC Syntax
A MAC has three algorithms:

1. `Gen` for key generation
2. `Mac_k(m)` for tag generation
3. `Vrfy_k(m, t)` for verification

The lecture gives the correctness condition:

```text
Vrfy_k(m, Mac_k(m)) = 1
```

for valid message-tag pairs.

---

## Informal Security Goal
The attacker should **not be able to forge** a valid tag for a new message.

That is the heart of MAC security.

### Exam Rule
**MAC security is about forgery resistance.**

---

# 16) MAC Forgery Game

The lecture defines the MAC attack game roughly like this:
- attacker asks for tags on messages of their choice
- eventually attacker outputs a **new** message `m` with a tag `t`
- if verification accepts and `m` was not asked before, attacker wins

### Exam Rule
The forged message must be **new**.

This is very important.

If the attacker only reuses a previously seen message-tag pair, that is not a new-message forgery.

---

# 17) Replay Attacks

The lecture points out an important limitation:

> MACs alone do not stop replay attacks.

An attacker may simply resend the same valid message and tag later.

## Example
Alice sends:

```text
"Pay Bob $1000"
```

with a valid MAC.

Attacker cannot modify it safely.
But attacker may replay it 10 times.

That is **not** a break of basic MAC unforgeability.

---

## Common defenses
The lecture mentions defenses like:
- sequence numbers
- timestamps
- session IDs

### Exam Rule
**Replay protection usually needs state or freshness, not just a MAC.**

---

# 18) Strong MAC for Fixed-Length Messages

The lecture gives a simple secure fixed-length construction using a secure PRF:

```text
Mac_k(m) = F_k(m)
```

and verification checks whether:

```text
t == F_k(m)
```

This works for fixed-length messages if `F` is a secure PRF.

---

## Extra Example
Suppose:

```text
F_k("HELLO") = 101101
```

Then the tag for `HELLO` is `101101`.

Receiver recomputes `F_k("HELLO")`.
If it matches the tag, accept. Otherwise reject.

---

## How to solve fixed-length MAC questions
If asked to design a MAC for fixed-length messages:
1. Assume secure PRF.
2. Set tag = PRF output on message.
3. Verify by recomputing the same PRF.

---

# 19) Bad MAC Designs for Arbitrary-Length Messages

The lecture spends a lot of time showing **failed attempts** for long messages.

This is very exam-important.

---

## Failed Attempt 1. Tag each block independently
If message is split into blocks and tag is just:

```text
Mac_k(m) = (t1,..., td)
where ti = Mac'_k(mi)
```

then block reordering becomes possible.

### Why it fails
Attacker can reorder blocks and reorder tags the same way.

So a new valid message-tag pair may be created.

### Exam Rule
**Independent block tagging fails because of block-reordering attacks.**

---

## Extra Example
Original message blocks:

```text
m1 = "Transfer"
m2 = "$100"
m3 = "to Bob"
```

Attacker reorders to:

```text
m1 = "$100"
m2 = "Transfer"
m3 = "to Bob"
```

and reorders tags too.

If verification is per-block only, the forged message may still verify.

---

## Failed Attempt 2. Include block index
Now tag each block as:

```text
ti = Mac'_k(i || mi)
```

This fixes reordering.

But the lecture says truncation is still possible.

### Why it fails
Attacker may drop the last block and its tag.
The remaining prefix may still verify.

### Exam Rule
**Adding block position prevents reordering, but not truncation.**

---

## Extra Example
Original message:

```text
m1 = "I don't like you."
m2 = "I LOVE you!"
```

If attacker removes the second block, the truncated message may still be accepted if length is not protected.

---

## Failed Attempt 3. Include index and total length
Now tag each block as:

```text
ti = Mac'_k(i || l || mi)
```

This fixes reordering and truncation.

But the lecture shows another issue:
**mix-and-match attack**.

### Why it fails
If two messages have same number of blocks and same length, attacker may combine blocks and tags from different messages.

### Exam Rule
**Even index + length may fail because blocks from different messages can be mixed together.**

---

## Extra Example
Message A:

```text
m1 = "Dear Alice"
m2 = "You are wonderful"
```

Message B:

```text
m1 = "Dear Alice"
m2 = "You are evil"
```

Attacker may combine the first block-tag pair from one message with later block-tag pairs from another compatible message.

---

# 20) Secure MAC for Arbitrary-Length Messages

The lecture finally gives a non-failed approach.

## Construction idea
Choose a random nonce `r`, then authenticate each block using:

```text
ti = Mac'_k(r || l || i || mi)
```

and output:

```text
(r, t1,..., td)
```

The random nonce ties all tags to the same specific message instance.

### Why it works better
- prevents reordering
- prevents truncation
- prevents mix-and-match across different messages because `r` changes

### Exam Rule
**Fresh randomness can bind all blocks together and stop combining attacks.**

---

## How to solve arbitrary-length MAC questions
If the exam asks why naive constructions fail, test these attacks in order:
1. Reordering
2. Truncation
3. Mix-and-match from different messages
4. Replay

If the construction includes:
- block index `i`
- total length `l`
- fresh random nonce `r`

then it is much stronger.

---

# 21) Error-Correcting Codes are Not a Security Tool

The lecture briefly explains that error-correcting codes are for handling random transmission errors, not malicious tampering.

## Exam Rule
**Error-correcting codes help with noise. MACs help with malicious modification.**

Do not confuse them.

---

# 22) Most Important Compare-and-Contrast Table

| Concept | Goal | Main Weakness / Note |
|---|---|---|
| ECB | Encrypt blocks independently | leaks patterns, not CPA-secure |
| CBC | Hides patterns via chaining | needs proper IV handling |
| CTR | Uses generated keystream | nonce/counter reuse is disastrous |
| CPA security | secrecy under chosen plaintext | does not handle decryption oracle |
| CCA security | secrecy under chosen ciphertext | stronger than CPA |
| MAC | integrity and authenticity | no secrecy, no replay protection by itself |
| Error-correcting code | fix random transmission errors | not protection against attackers |

---

# 23) Common Exam Questions and How to Answer

## Q1. Why is ECB insecure?
### Answer outline
- deterministic
- repeated plaintext blocks give repeated ciphertext blocks
- patterns remain visible
- therefore not CPA-secure

---

## Q2. Why is CBC better than ECB?
### Answer outline
- uses IV and chaining
- each block depends on previous ciphertext
- repeated plaintext blocks do not necessarily produce repeated ciphertext blocks

---

## Q3. Why is CTR secure only with fresh counters?
### Answer outline
- same counter + same key gives same keystream
- keystream reuse leaks `m1 XOR m2`
- therefore counters or nonces must not repeat under same key

---

## Q4. Why does CPA security not imply CCA security?
### Answer outline
- CPA attacker gets encryption oracle only
- CCA attacker also gets decryption oracle
- malleable ciphertext may be modified and tested through decryption oracle
- so stronger attacks become possible

---

## Q5. What is the security goal of a MAC?
### Answer outline
- prevent forgery of valid tag for a new message
- receiver verifies authenticity and integrity

---

## Q6. Why do MACs not stop replay attacks?
### Answer outline
- replay uses the exact same valid message-tag pair
- no new forgery is needed
- need timestamps, sequence numbers, or session IDs

---

## Q7. Why do naive long-message MAC constructions fail?
### Answer outline
Test in order:
1. Can attacker reorder blocks?
2. Can attacker truncate message?
3. Can attacker mix blocks from multiple valid messages?

If yes to any of these, the construction is insecure.

---

# 24) Fast Revision Sheet

## Must memorize
- PRF = indistinguishable from random function
- PRP = PRF + invertible
- ECB leaks patterns
- CBC and CTR can achieve CPA security with correct conditions
- CCA is stronger than CPA
- malleability is dangerous
- padding oracle attacks exploit error leakage
- MAC gives integrity/authenticity, not secrecy
- MAC alone does not stop replay
- fixed-length MAC from PRF is simple
- arbitrary-length MAC needs careful design

---

# 25) Final Summary

This lecture teaches one huge lesson:

> **Secrecy alone is not enough.**

A secure system must think about:
- secrecy
- integrity
- attacker interaction
- ciphertext modification
- side-channel style leaks like padding errors
- safe composition for long messages

So modern cryptography does not just ask:

> "Can the attacker read the message?"

It also asks:

> "Can the attacker modify it, replay it, test decryptions, or learn anything from system behavior?"

That is why:
- ECB fails
- CPA is not the end goal
- CCA matters
- MACs matter
- careful construction matters

---

# 26) Very Short One-Line Memory Aids

- **PRF**: looks random
- **PRP**: looks random + invertible
- **ECB**: same block, same ciphertext, bad
- **CBC**: chain blocks
- **CTR**: encrypt counters, XOR keystream
- **CCA**: attacker can decrypt chosen ciphertexts
- **MAC**: stop forgeries
- **Replay**: same valid message sent again
- **Padding oracle**: error message leaks plaintext info
