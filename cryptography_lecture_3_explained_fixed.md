---
created: 2026-04-02T23:58
updated: 2026-06-17T23:32
---
# Cryptography Lecture 3 Explained in a Simple, Exam-Focused Way

> Cleaned formatting version based on Lec_3.pdf.

---

# 1) Big Picture of This Lecture

In the previous lectures, classical ciphers were weak, and the one-time pad was perfectly secure. But the one-time pad has a serious practical problem:

- the key must be truly random
- the key must be as long as the message
- the key must never be reused

That works in theory, but it is hard in practice, especially if we want to send many messages. The lecture starts from this problem and shows how modern cryptography solves it using **computational assumptions**. The lecture then introduces:

- **EAV security**
- **pseudorandom generators (PRGs)**
- **stream ciphers**
- **multiple-message security**
- **chosen-plaintext attack security (CPA security)**

---

# 2) Perfect Secrecy and the One-Time Pad

## What the lecture is asking
The first slides ask whether it is practical to use a **one-time pad** for things like message broadcast or sending many messages. The answer is usually **no**, because key management becomes very hard. On page 1, the lecture reminds us that OTP is unbreakable only when these conditions hold: the key is truly random, at least as long as the message, never reused, and kept completely secret.

## Core idea
A one-time pad gives **information-theoretic security**, meaning security that does not depend on computational limits.

## But why is it impractical?
If Alice and Bob want to exchange many messages, then they need many fresh keys, one for each message. Pages 3 and 4 show this visually using different keys $K_1, K_2, K_3$ for different messages.

### Important rule
> **Rule:** In a one-time pad, **reusing the key is forbidden**.

### Why this rule matters
If the same OTP key is reused, security collapses.

### Exam-style statement
> **Exam point:** OTP is perfectly secret only under strict conditions. If any condition is violated, especially **key reuse**, perfect secrecy no longer holds.

### Additional simple example
Suppose:
- $m_1 = 101100$
- $m_2 = 110010$
- both are encrypted using the **same OTP key** $k$

Then:
- $c_1 = m_1 \oplus k$
- $c_2 = m_2 \oplus k$

If the attacker XORs them:
- $c_1 \oplus c_2 = m_1 \oplus m_2$

The key disappears, which leaks structure about the messages.

### How to solve this kind of exam question
If the question mentions OTP, immediately check:
1. Is the key truly random?
2. Is the key as long as the message?
3. Is the key reused?
4. Is the key secret?

If any answer is bad, say clearly: **OTP conditions are violated, so perfect secrecy is lost.**

---

# 3) EAV Security, Security Against an Eavesdropper

On page 5, the lecture introduces **EAV-secure encryption**. The idea is simple: an attacker sees a ciphertext and should not be able to tell which of two chosen messages was encrypted, except with probability only negligibly better than random guessing.

## The game intuition
1. Attacker chooses two messages $m_0$ and $m_1$
2. A random bit $b\in\{0,1\}$ is chosen
3. The challenger encrypts $m_b$
4. The attacker sees the ciphertext and guesses $b'$
5. If the attacker cannot do noticeably better than $1/2$, the scheme is secure

## Meaning of the formula
The slide says roughly:
$$
Pr[b'=b] \leq \frac{1}{2} + \mu(n)
$$
where $\mu(n)$ is negligible.

That means:
- $1/2$ = pure random guessing
- $\mu(n)$ = only a tiny extra advantage

## Concrete version
Page 6 gives the concrete version using $(t(n),\varepsilon(n))$-security. Here the attacker is limited to time $t(n)$, and the advantage must be at most $\varepsilon(n)$.

### Important rule
> **Rule:** EAV security means the ciphertext should not help an eavesdropper distinguish between two equally long challenge messages.

### Exam shortcut
If you see a distinguishing game with:
- two messages
- random bit $b$
- ciphertext of one of them
- attacker outputs $b'$

then you are in an **indistinguishability security game**.

### Additional example
Suppose attacker chooses:
- $m_0 =$ "attack"
- $m_1 =$ "retreat"

If after seeing the ciphertext the attacker can guess correctly with probability $0.99$, the scheme is **not secure**.

If the best attacker can do is $0.500001$, that may be secure if the extra amount is negligible.

### How to solve this type of question
When asked whether a scheme is EAV-secure:
1. Try to choose two messages that create a visible pattern in ciphertext
2. Show how an attacker can guess $b$
3. Compute success probability
4. Compare it to $1/2$
5. If advantage is noticeable, the scheme is insecure

---

# 4) Randomness and Why We Need PRGs

Page 8 asks what “random” and “uniform” mean, and reminds us that every n-bit string should appear with probability $2^{-n}$ in a uniform distribution.

## The problem
Real randomness is expensive. If we want to encrypt lots of data, we need lots of random bits.

## The big idea
Can we turn a **short random string** into a **long string that looks random**?

Page 10 answers:
- turn short random into long **truly random**: impossible
- turn short random into long **pseudorandom-looking** string: yes, by using a **PRG**

---

# 5) Pseudorandom Generator, PRG

Page 9 defines a PRG. A pseudorandom generator takes a short random seed and stretches it into a longer string that looks random to efficient attackers.

## Definition in simple words
A PRG is a function $G$ such that:
- input: short random seed $s \in \{0,1\}^n$
- output: longer string $G(s) \in \{0,1\}^{\ell(n)}$
- and $\ell(n) > n$

## Expansion factor
The value $\ell(n)$ is the output length.

### Important rule
> **Rule:** A PRG must **expand**. So the output length must be larger than the seed length.

### Example
If seed length is 128 bits and output length is 1024 bits, then the PRG expands 128 random bits into 1024 pseudorandom bits.

### Exam-style example
Input:
- $s \in \{0,1\}^{10}$

Output:
- $G(s) \in \{0,1\}^{100}$

Then the expansion factor is:
- $\ell(10)=100$

### How to solve this type of question
If the exam asks for expansion factor:
1. Find seed length $n$
2. Find output length
3. Write $\ell(n)$
4. Make sure $\ell(n) > n$

---

# 6) PRG Security as a Distinguishing Game

Pages 9, 11, and 12 define PRG security using another indistinguishability game. The attacker receives either:
- a truly random string $R$, or
- a PRG output $G(s)$

and tries to distinguish them.

## Intuition
A good PRG should fool every efficient attacker.

If the attacker cannot tell the difference between:
- real random bits, and
- PRG-generated bits,

then the PRG is secure.

### Important rule
> **Rule:** PRG security is not “the output is actually random”. It is “the output is **computationally indistinguishable** from random”.

### Very common confusion
- **False:** PRG output is truly random
- **Correct:** PRG output only **looks random to efficient attackers**

### Additional example
Imagine two boxes:
- Box A outputs real random 100-bit strings
- Box B outputs $G(s)$ for random 20-bit seeds

If no polynomial-time attacker can reliably tell which box they are looking at, then $G$ is a secure PRG.

### How to solve this type of question
If asked whether a function is a secure PRG:
1. Look for a pattern in the output
2. Design a distinguisher that checks that pattern
3. Compute probabilities in the real-random case and PRG case
4. Compare them
5. If gap is noticeable, PRG is insecure

---

# 7) A Bad PRG Example

Page 13 gives a very important bad example:
$$
G(s)=s\|1
$$
This means the generator just appends a final 1 bit to the seed. The slide notes that the output length is $n+1$, so the expansion factor is $\ell(n)=n+1$. It then builds a distinguisher that checks the last bit.

## Why it is insecure
If output is always ending in 1, then a distinguisher can do this:
- output 1 if last bit is 1
- output 0 otherwise

For PRG output, this succeeds with probability 1.
For true random $(n+1)$-bit strings, the last bit is 1 only with probability $1/2$.

So the advantage is:
$$
1 - \frac{1}{2} = \frac{1}{2}
$$
which is definitely not negligible.

### Important rule
> **Rule:** If the output has an obvious predictable pattern, then the function is not a secure PRG.

### Additional exam example
Suppose:
$$
G(s)=0\|s
$$
Every output starts with 0.

Distinguisher:
- if first bit is 0, guess “PRG output”
- else guess “truly random”

Success gap is again noticeable, so the PRG is insecure.

### How to solve this type of question
For any proposed PRG:
1. Check whether some bits are fixed or predictable
2. Check whether some relation always holds, like first bit = last bit
3. Build a distinguisher around that property
4. Compute probability under PRG and under uniform randomness
5. If difference is noticeable, say “not a secure PRG”

---

# 8) One-Time Pad + PRG

Page 14 combines the one-time pad idea with a PRG. Instead of using a message-length truly random key, we use a short seed $s$, expand it using $G(s)$, and XOR it with the message:
$$
Enc_s(m) = G(s) \oplus m
$$
$$
Dec_s(c) = G(s) \oplus c
$$
## Why decryption works
Because XORing twice with the same value cancels:
$$
G(s) \oplus (G(s) \oplus m)=m
$$
## Advantage
A short key can encrypt a longer message.

## Important subtle point
This is no longer **perfect secrecy**.
It becomes **computationally secure**, assuming $G$ is a secure PRG.

The lecture states that if $G$ is a pseudorandom generator, then this scheme has indistinguishable encryptions in the presence of an eavesdropper.

### Important rule
> **Rule:** OTP + PRG gives computational security, not information-theoretic security.

### Additional example
Let:
- seed $s$ be 128 bits
- PRG expands it to 2048 bits
- message length is 2048 bits

Then we can encrypt that message by XORing it with the PRG output.

### How to solve this type of question
If asked why the scheme is secure:
1. Say the ciphertext is XORed with a string that looks random
2. Since PRG output is indistinguishable from random, the scheme behaves like OTP to efficient attackers
3. Therefore it is EAV-secure, assuming the PRG is secure

---

# 9) Stream Cipher vs PRG

Page 15 explains the difference. A PRG typically outputs all pseudorandom bits at once, while a stream cipher can produce pseudorandom bits gradually as a stream.

## PRG
- input seed
- output all bits in one shot

## Stream cipher
- input initial state or seed
- produce bits one after another

This is useful when encrypting continuous data streams.

### Important rule
> **Rule:** A stream cipher is suitable when you want keystream bits generated progressively, not all at once.

### Example
If you are encrypting live audio or network traffic, stream output is more natural than generating a giant keystream in advance.

---

# 10) Synchronized vs Unsynchronized Stream Ciphers

Page 16 shows two modes: synchronized and unsynchronized.

## Synchronized mode
- sender and receiver keep the same internal state progression
- both generate matching keystream blocks
- if they stay aligned, decryption works

## Unsynchronized mode
- receiver can recover synchronization from ciphertext or some feedback mechanism
- useful when communication errors or dropped bits occur

### Important rule
> **Rule:** In synchronized stream ciphers, losing synchronization can break decryption until alignment is restored.

### Simple example
If sender sends keystream positions 1, 2, 3, 4 but receiver misses position 2, then later XORs will be wrong because the two sides are no longer aligned.

### How to answer exam questions here
If asked for the difference:
- synchronized: both sides must remain in lockstep
- unsynchronized: can recover alignment from the data flow or internal design

---

# 11) RC4 Overview

Page 17 gives a short overview of the RC4 cipher. It says RC4 maintains:
- a 256-byte array $S$, containing a permutation of 0 to 255
- two indexes $i$ and $j$
- a repeated update process that swaps values and outputs bytes based on the current state

## Why the slide matters
You may not need to memorize every low-level operation, but you should understand:
- RC4 is a **stream cipher**
- it has an **internal state**
- output depends on state updates over time

### Important rule
> **Rule:** RC4 is stateful. Its output depends on its current internal permutation and indexes.

### Exam note
If asked conceptually, say:
- RC4 generates a keystream byte by byte
- encryption is usually done by XORing plaintext with keystream

---

# 12) Limitation of One-Ciphertext Security

Pages 18 and 19 point out that the current definition is too weak if it only assumes the attacker sees one ciphertext. The slides specifically ask:
- what if the attacker sees two ciphertexts?
- what if the attacker tries to modify a ciphertext?

## Important issue 1, seeing two ciphertexts
If the same keystream $G(s)$ is reused for two messages:
$$
c_1 = G(s) \oplus m_1
$$
$$
c_2 = G(s) \oplus m_2
$$
then:
$$
c_1 \oplus c_2 = m_1 \oplus m_2
$$
This leaks information.

## Important issue 2, malleability
Because XOR is linear, an attacker can flip chosen bits in the ciphertext and cause predictable flips in plaintext after decryption.

### Important rule
> **Rule:** Reusing the same keystream across multiple messages is dangerous.

### Important rule
> **Rule:** Simple XOR-based encryption without integrity protection is malleable.

### Additional example
Suppose the amount in a payment message contains the bit pattern for 1000, and the attacker knows where those bits are. They can flip bits in the ciphertext to try to turn 1000 into 9000 or another amount.

### How to solve this type of question
If the exam asks how an attacker can tamper:
1. Write ciphertext as $c = k \oplus m$
2. Let attacker choose mask $\Delta$
3. New ciphertext is $c' = c \oplus \Delta$
4. After decryption:
$$
   m' = c' \oplus k = (c\oplus\Delta)\oplus k = m \oplus \Delta
$$
5. So the attacker can force controlled changes in the plaintext

---

# 13) Multiple-Message Eavesdropping Security

Pages 20 and 21 define the next stronger notion: the attacker chooses **multiple pairs of messages**, then receives ciphertexts corresponding to one side of all pairs, using the same hidden bit $b$. The attacker still should not be able to tell whether the left set or right set was encrypted.

## Intuition
The attacker is no longer limited to one challenge message pair.

### Important rule
> **Rule:** Multiple-message security is stronger than single-message eavesdropping security.

---

# 14) Attack on Deterministic Multiple Encryption

Page 22 gives a very important attack. The attacker cleverly chooses messages so that one world produces equal ciphertexts and the other produces different ciphertexts. Then the attacker simply checks whether $c_1 = c_2$.

## Intuition behind the attack
If encryption is deterministic, then the same plaintext always gives the same ciphertext.

So if the attacker can arrange for:
- in one case: same plaintext twice
- in the other case: different plaintexts

then equality of ciphertexts leaks which world was chosen.

### Important rule
> **Rule:** Deterministic stateless encryption cannot provide secure indistinguishable multiple encryptions.

Page 23 explicitly states the theorem:
- if the scheme is stateless and deterministic, it does **not** provide secure indistinguishable multiple encryptions in the presence of an eavesdropper.

### Additional example
Attacker chooses:
- pair 1: $(0^n, 0^n)$
- pair 2: $(0^n, 1^n)$

If encrypted messages come from the left side, both ciphertexts are encryptions of $0^n$, so for deterministic encryption they are equal.

If they come from the right side, one is encryption of $0^n$, the other of $1^n$, so they differ.

### How to solve this kind of question
If asked to break deterministic multiple-message security:
1. Choose messages that repeat in one world but differ in the other
2. Use equality of ciphertexts as the test
3. Show attacker can distinguish with high probability
4. Conclude the scheme is insecure

---

# 15) Why Randomized Encryption Is Needed

Page 24 explains the fix: do not weaken the definition. Instead, use **randomized encryption** so that encrypting the same message twice can give different ciphertexts.

### Important rule
> **Rule:** Modern encryption should be randomized so that the same plaintext can encrypt to different ciphertexts.

### Example
If encryption uses fresh randomness $r$:
- $Enc_k(m;r_1)$
- $Enc_k(m;r_2)$

then even for the same message $m$, the ciphertexts can differ if $r_1 \neq r_2$.

---

# 16) Chosen-Plaintext Attacks, CPA

Pages 26 and 27 introduce chosen-plaintext attacks. The attacker is allowed to influence or choose messages that honest parties encrypt. The lecture uses WWII stories to motivate this idea, including the Battle of Midway example.

## Meaning in simple words
The attacker is stronger now:
- not just passively observing
- but also asking for encryptions of chosen plaintexts

This is much closer to real life.

### Important rule
> **Rule:** CPA models attackers who can obtain encryptions of messages of their choice.

### Why it matters
Real systems often expose encryption as a service. If an attacker can submit chosen inputs and see outputs, they may learn patterns.

---

# 17) CPA Security, Single Message

Page 31 gives the single-message CPA game. The attacker can query the encryption oracle on messages of choice, then submits challenge messages $m_0,m_1$, receives encryption of one of them, continues interacting, and finally guesses which one was encrypted. The success probability still should be at most $1/2 + \mu(n)$.

## Intuition
Even if the attacker has seen many ciphertexts of chosen messages, they still should not learn which challenge message was encrypted.

### Important rule
> **Rule:** CPA security is stronger than ordinary eavesdropping security.

---

# 18) CPA Security, Multiple Messages

Page 32 extends CPA to the multiple-message setting. The attacker can adaptively choose many message pairs and receive encryptions corresponding to the hidden bit $b$. Again the attacker should not do much better than guessing.

## Important takeaway
CPA security covers a much stronger attacker than plain EAV security.

Page 33 states that CPA security is a **stronger guarantee** than multiple-message encryption and is the **minimal security notion for a modern cryptosystem**. It also notes limitations: CPA does not model attackers who modify messages or who can get decryption help from honest parties.

### Important rule
> **Rule:** CPA security is the minimum security notion expected from modern encryption.

### Exam note
If asked to compare:
- EAV security < multiple-message security < CPA security

in terms of attacker strength.

---

# 19) CPA Security and Message Length

On page 34, the lecture gives an observation: if you already have a CPA-secure scheme for one-bit messages, you can build a scheme for n-bit messages by encrypting each bit separately:
$$
Enc'_k(m) = (Enc_k(m_1), \ldots, Enc_k(m_n))
$$
where $m = m_1\ldots m_n$.

## Intuition
If each component encryption is CPA-secure, then the whole tuple remains secure.

### Important rule
> **Rule:** Security for longer messages is often built by composition from secure smaller-message encryption.

### Additional example
If you can securely encrypt one byte, then in principle you can encrypt a 4-byte message as:
$$
(Enc_k(b_1), Enc_k(b_2), Enc_k(b_3), Enc_k(b_4))
$$
The proof idea is usually a hybrid argument.

### How to answer proof-style exam questions here
If asked how to prove the new scheme is CPA-secure:
1. Assume there exists an attacker that breaks the long-message scheme
2. Build another attacker using it to break the original one-bit scheme
3. Replace encrypted components one by one using hybrids
4. Show any noticeable difference would contradict the CPA security of the base scheme

---

# 20) High-Yield Comparison Table

| Concept                  | Main Idea                                                 | Security Type                 | Exam Danger                               |
| ------------------------ | --------------------------------------------------------- | ----------------------------- | ----------------------------------------- |
| One-Time Pad             | XOR with truly random one-time key                        | Perfect secrecy               | Key reuse destroys security               |
| PRG                      | Stretch short random seed into longer pseudorandom string | Computational                 | Output only looks random                  |
| OTP + PRG                | XOR message with PRG output                               | EAV security if PRG secure    | Not perfect secrecy                       |
| Stream Cipher            | Generate keystream as a stream                            | Computational                 | Synchronization issues                    |
| Deterministic encryption | Same plaintext gives same ciphertext                      | Weak for multiple encryptions | Equality leaks info                       |
| Randomized encryption    | Same plaintext can give different ciphertexts             | Stronger, modern              | Fresh randomness required                 |
| CPA security             | Attacker can request encryptions of chosen messages       | Modern minimum notion         | Still does not cover ciphertext tampering |
|                          |                                                           |                               |                                           |

---

# 21) Most Important Rules to Memorize

> **Rule 1:** OTP is perfectly secret only if the key is random, secret, message-length, and never reused.

> **Rule 2:** PRG output is not truly random. It is only computationally indistinguishable from random.

> **Rule 3:** A bad PRG can often be broken by finding a simple detectable pattern.

> **Rule 4:** $Enc_s(m)=G(s)\oplus m$ is computationally secure assuming $G$ is a secure PRG.

> **Rule 5:** Reusing the same keystream for multiple messages leaks information.

> **Rule 6:** Deterministic stateless encryption cannot securely support indistinguishable multiple encryptions.

> **Rule 7:** Same plaintext encrypted twice should generally produce different ciphertexts in modern schemes.

> **Rule 8:** CPA security is the minimal modern notion for secure encryption.

---

# 22) Common Exam Question Types and How to Solve Them

## Type A. “Is this a secure PRG?”
### Method
1. Look for obvious structure in the output
2. Build a distinguisher
3. Compute probabilities in PRG world and random world
4. Compare

### Red flags
- fixed first bit
- fixed last bit
- parity always even
- duplicate halves

---

## Type B. “Why is OTP impractical?”
### Method
Mention:
- key must be as long as message
- truly random
- never reused
- difficult for many messages or broadcast

---

## Type C. “Break this deterministic encryption under multiple-message attack”
### Method
1. Choose repeated message in one world
2. Choose different messages in the other world
3. Check ciphertext equality
4. Distinguish the worlds

---

## Type D. “Show tampering attack on XOR-based encryption”
### Method
1. Write $c = k \oplus m$
2. Choose desired mask $\Delta$
3. Send $c' = c \oplus \Delta$
4. Receiver gets $m' = m \oplus \Delta$

---

## Type E. “What is the difference between EAV and CPA security?”
### Good answer
- EAV: attacker only observes ciphertext
- CPA: attacker can also request encryptions of chosen plaintexts
- CPA is stronger and is the modern minimum

---

## Type F. “How do we extend security from one-bit messages to n-bit messages?”
### Method
1. Encrypt each part securely
2. Build a hybrid argument
3. Reduce any attacker on long messages to attacker on the base scheme

---

# 23) Fast Revision Summary

This lecture shows the transition from perfect secrecy to practical modern security.

- OTP is theoretically perfect but hard to use in practice
- PRGs let us stretch short randomness into long pseudorandom-looking bits
- with a secure PRG, we can encrypt longer messages using XOR and get computational security
- one-message eavesdropping security is not enough
- multiple-message settings and chosen-plaintext attacks are stronger and more realistic
- deterministic stateless encryption fails in these stronger settings
- randomized encryption is necessary
- CPA security is the minimum standard for modern encryption

---

# 24) Final Mini-Checklist Before the Exam

Before answering any question from this lecture, ask yourself:

- Is the scheme **perfectly secure** or only **computationally secure**?
- Is the attacker just observing, or can they choose plaintexts?
- Is encryption deterministic or randomized?
- Is the same keystream or key reused?
- Can I build a distinguisher?
- Can I show a noticeable advantage over $1/2$?
- Is there a reduction or hybrid proof idea hiding in the question?

If you can answer those, you will usually know the right direction.
