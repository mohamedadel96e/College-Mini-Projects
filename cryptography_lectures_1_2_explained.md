# Cryptography Lectures 1 & 2. Easy Notes + Exam Guide

Source lecture: **Lec_1&2.pdf**  
Prepared from the uploaded lecture slides.

---

# 1) What is Cryptography?

## Simple meaning
Cryptography is the study of mathematical techniques used to secure digital information, systems, and communication against attackers.

## Very short intuition
It is not only about "hiding text". It is about:
- defining security clearly,
- building algorithms carefully,
- proving what attackers can and cannot do.

## Important rule
**Modern cryptography is not based only on clever tricks. It is based on definitions, assumptions, and proofs.**

## Easy picture to remember
- Old cryptography: more like an art.
- Modern cryptography: science + math + proofs.

## Exam note
If the question asks **"What makes modern cryptography modern?"**, mention:
1. formal definitions,  
2. algorithmic assumptions,  
3. security proofs/reductions.

---

# 2) What does “secure information” mean?

The lecture focuses first on two basic goals.

## 2.1 Confidentiality
Only the intended receiver should understand the message.

### Example
Alice sends a message to Bob. Eve may see the ciphertext, but Eve should learn nothing useful about the actual message.

## 2.2 Integrity / Authenticity
The received message should really come from the claimed sender and should not be changed.

### Example
Bob sends:  
> I love you Alice. - Bob

Mallory changes it to:  
> We need to break up. - Bob

This is an **integrity failure**.

## Important rule
**Confidentiality protects secrecy. Integrity protects correctness/authenticity.**

## Exam trick
If the question says the attacker only reads messages, think **confidentiality**.  
If the attacker changes messages, think **integrity** too.

---

# 3) Attacker Models

## Passive attacker
A passive attacker can only observe or eavesdrop.

### Needs protection against
- confidentiality attacks

## Active attacker
An active attacker can observe, modify, inject, delete, or reorder messages.

### Needs protection against
- confidentiality attacks
- integrity attacks

## Important rule
**Passive attacker = listen only. Active attacker = control the channel.**

## Quick comparison table

| Attacker | What they do | What we need |
|---|---|---|
| Passive | Observe only | Confidentiality |
| Active | Observe + modify | Confidentiality + Integrity |

## Extra exam example
Suppose Mallory intercepts an encrypted bank transfer and changes one bit so that `500` becomes `900` after decryption.  
This is an **active attack**.

---

# 4) Steganography vs Cryptography

## Steganography
Hide the **existence** of the message.

Example: invisible ink, hidden text inside an image, hidden tattoo under hair.

## Cryptography
Hide the **meaning** of the message.

Even if the attacker knows a message is being sent, the attacker should not understand it.

## Kerckhoffs’s Principle
A cryptosystem should remain secure even if the attacker knows the algorithm. Only the key should be secret.

## Important rule
**The algorithm should not need to be secret. The key should be secret.**

## Exam answer pattern
If asked: **Why is key secrecy preferred over algorithm secrecy?**
Say:
- keys are short and easy to change,
- algorithms are usually public and reverse-engineerable,
- if one key is leaked we can replace it, but replacing the entire algorithm is much harder.

---

# 5) Symmetric Key Encryption

## Idea
Both sender and receiver share the same secret key.

- sender encrypts using the key,
- receiver decrypts using the same key.

## Basic notation
- plaintext: `m`
- ciphertext: `c`
- key: `k`

## Correctness rule
For every valid plaintext `m` and key `k`:

```text
Dec_k(Enc_k(m)) = m
```

## Important rule
**If a scheme is correct, decrypting an encrypted message gives back the original message.**

---

# 6) Basic Terminology

## Plaintext
The original readable message.

## Ciphertext
The encrypted message.

## Message space `M`
The set of all possible plaintexts.

## Ciphertext space `C`
The set of all possible ciphertexts.

## Key space `K`
The set of all possible keys.

## Private-key encryption syntax
A private-key encryption scheme consists of 3 algorithms:
- `Gen` for key generation
- `Enc` for encryption
- `Dec` for decryption

## Important rule
In many definitions, you should always remember these three parts:

```text
Π = (Gen, Enc, Dec)
```

## Exam tip
When defining an encryption scheme, always mention:
1. message space,  
2. key space,  
3. algorithms `Gen, Enc, Dec`,  
4. correctness.

---

# 7) Shift Cipher / Caesar Cipher

This is the first classical example.

## Idea
Shift every letter by `k` positions.

For Caesar cipher, the key is fixed at `k = 3`.

## Encryption formula
If letters are represented mod 26:

```text
C = M + k mod 26
```

## Decryption formula
```text
M = C - k mod 26
```

## Lecture example
`BEGIN THE ATTACK NOW` with shift `k=3` becomes something like:
`EHJLQ WKH DWWDFN QRZ`

## Extra exam example 1
Encrypt `HELLO` with `k = 4`:
- H -> L
- E -> I
- L -> P
- L -> P
- O -> S

So ciphertext is:

```text
LIPPS
```

## Extra exam example 2
Decrypt `FDHVDU` with `k = 3`:
- F -> C
- D -> A
- H -> E
- V -> S
- D -> A
- U -> R

Answer:

```text
CAESAR
```

## Important rule
**Shift cipher has very small key space: 26 possible keys only.**

That makes brute force easy.

---

# 8) Brute Force Attack

## Idea
Try every possible key until the plaintext makes sense.

For shift cipher, there are only 26 keys. So an attacker can try all of them.

## Important rule
**Small key space means exhaustive search is feasible.**

## Lecture point
The correct message appears when the attacker tries the right key.

## Exam-style solving rule
If the cipher is shift cipher and no key is given:
1. try all 26 keys,  
2. inspect each output,  
3. choose the meaningful English sentence.

## Extra exam example
Ciphertext: `KHOOR`  
Try `k = 3` backward:
- K -> H
- H -> E
- O -> L
- O -> L
- R -> O

Answer: `HELLO`

---

# 9) Sufficient Key Space Principle

A secure encryption scheme must have a key space large enough so that exhaustive search is infeasible.

## Important rule
**Large key space is necessary, but not sufficient, for security.**

## Very important exam point
If the question is:
> If the key space is huge, is the scheme automatically secure?

Answer:
> No.

Because the scheme may still leak patterns or allow smarter attacks than brute force.

## Example
Substitution cipher has a huge key space (`26!`) but is still weak because of frequency analysis.

---

# 10) Substitution Cipher

## Idea
Instead of shifting letters, each letter is replaced by another letter according to a permutation of the alphabet.

Example key mapping:
- A -> X
- B -> E
- C -> U
- ...

## Key space size
The number of permutations of 26 letters is:

```text
26!
```

This is approximately `2^88`, which is much bigger than 26.

## Important rule
**A large key space does not stop statistical attacks.**

---

# 11) Frequency Analysis

Frequency analysis breaks substitution ciphers by using the fact that some letters occur more often than others in normal English.

## Main observations
1. If plaintext `e` always maps to ciphertext `d`, then every `e` becomes `d`.  
2. In English, some letters are much more frequent than others, especially `e`.  
3. Long enough text shows approximate English frequency patterns.

## How the attack works
1. Count frequencies of ciphertext letters.  
2. Guess the most common ciphertext letter corresponds to `e`.  
3. Use common short words and patterns to refine the guesses.

## Important rule
**Substitution ciphers preserve letter frequencies. This is why frequency analysis works.**

## Extra exam example
Suppose in a ciphertext the letter `Q` appears much more than all others.  
A reasonable first guess is:

```text
Q ≈ e
```

Then test whether nearby patterns start forming common words.

## How to solve substitution questions in exams
1. Find the most frequent ciphertext letter.  
2. Compare with English frequency: `e, t, a, o, i, n...`  
3. Look at one-letter words. They are usually `a` or `I`.  
4. Look at repeated patterns like double letters.  
5. Look for common words like `the`, `and`, `is`, `to`.  
6. Keep updating your mapping table.

---

# 12) Cryptograms in the Lecture

The lecture shows an example of solving a substitution-style puzzle step by step.

## Main lesson
You do not always recover the plaintext instantly. You gradually improve guesses by combining:
- letter frequency,
- common words,
- grammar intuition,
- repeated letter patterns.

## Final recovered sentence in the slide example
> The mind is not a vessel to be filled, but a fire to be kindled.

## Important rule
**Cryptanalysis is often iterative. One correct guess unlocks the next one.**

---

# 13) Vigenère Cipher

## Idea
It generalizes the shift cipher by using multiple shifts instead of one fixed shift.

If the key is `k1, k2, ..., kt`, then:
- first letter is shifted by `k1`,
- second letter by `k2`,
- ...
- then the pattern repeats.

## Important point
This is stronger than a simple Caesar cipher because the same plaintext letter may encrypt differently depending on its position.

## Lecture note
Its key space can be very large, but brute force is still not the main issue. Statistical attacks can still work if the message is long enough.

## Important rule
**Vigenère is harder than shift cipher, but it is still not fully secure.**

## Special note from lecture
If the key is as long as the message and used only once, it becomes the **One-Time Pad**, which is perfectly secret.

## Extra exam example
Suppose key = `ABC`, meaning shifts `(0,1,2)`.
Encrypt `DOG`:
- D shifted by 0 -> D
- O shifted by 1 -> P
- G shifted by 2 -> I

Ciphertext:

```text
DPI
```

---

# 14) Rotor Machines

Rotor machines are more advanced historical ciphers.

## Idea
They use rotating internal states, so encryption changes from one character to the next.

## Example from lecture
- Hebern machine
- Enigma

## Important lesson
Even very complicated-looking historical systems were eventually broken.

## Important rule
**Complexity alone does not imply security.**

---

# 15) Historical Conclusion

The lecture’s big conclusion is:

## Important rule
**All historical ciphers have fallen.**

This teaches us that intuition and cleverness are not enough. We need formal security definitions and proofs.

---

# 16) Principles of Modern Cryptography

## Core message
To say a scheme is secure, we must define exactly:
1. what security goal we want,  
2. what attacker we allow,  
3. what assumptions we are making.

## Why older attempts fail
The lecture shows weak candidate definitions like:
- attacker cannot recover the key,  
- attacker cannot recover all plaintext,  
- attacker cannot recover even one character.

Each of these can be too weak or too strong in the wrong way.

## Better idea
A ciphertext should leak **no additional information** about the plaintext.

## Important rule
**A security definition must include both a goal and a threat model.**

---

# 17) Perfect Secrecy. Intuition

## Intuition
Even after seeing the ciphertext, the attacker should not learn anything new about the plaintext.

### Informal meaning
The attacker’s beliefs before seeing ciphertext and after seeing ciphertext should be the same.

## Lecture example
Before seeing ciphertext:
- `Pr[m = wait] = 0.3`
- `Pr[m = attack] = 0.7`

After seeing ciphertext, if it changes to:
- `Pr[m = wait | c] = 0.2`
- `Pr[m = attack | c] = 0.8`

then the attacker learned something. So the scheme is **not perfectly secret**.

## Important rule
**Perfect secrecy means posterior = prior.**

That is:

```text
Pr[M = m | C = c] = Pr[M = m]
```

for every message `m` and every ciphertext `c` that can occur.

---

# 18) Bayes’ Theorem in These Questions

Bayes’ Theorem is used in the lecture to reason about probabilities after seeing ciphertext.

## Formula
```text
Pr[A | B] = Pr[B | A] Pr[A] / Pr[B]
```

In crypto questions, a common form is:

```text
Pr[M = m | C = c] = Pr[C = c | M = m] Pr[M = m] / Pr[C = c]
```

## How to solve this type of exam question
If asked to compute `Pr[M=m | C=c]`, do this:
1. identify `Pr[C=c | M=m]`,  
2. identify the prior `Pr[M=m]`,  
3. compute `Pr[C=c]` using total probability,  
4. plug into Bayes’ formula.

## Extra exam example
Suppose:
- `Pr[M='hi'] = 0.4`
- `Pr[M='no'] = 0.6`
- `Pr[C='xy' | M='hi'] = 1/2`
- `Pr[C='xy' | M='no'] = 1/4`

Then:

```text
Pr[C='xy'] = (1/2)(0.4) + (1/4)(0.6) = 0.2 + 0.15 = 0.35
```

Now:

```text
Pr[M='hi' | C='xy'] = ((1/2)(0.4)) / 0.35 = 0.2 / 0.35 = 4/7
```

---

# 19) Formal Definitions of Perfect Secrecy

The lecture gives two equivalent definitions.

## Definition 1
For every probability distribution over messages `D`, every message `m`, and every ciphertext `c` with `Pr[C=c] > 0`:

```text
Pr[M = m | C = c] = Pr[M = m]
```

This says ciphertext gives no extra information.

## Definition 2
For every pair of messages `m, m'` and every ciphertext `c`:

```text
Pr[Enc_K(m) = c] = Pr[Enc_K(m') = c]
```

This says ciphertext distributions are identical no matter which message was encrypted.

## Important rule
**Definition 1 is more intuitive. Definition 2 is often easier to prove.**

## Exam tip
If asked why they are equivalent, say:
- Definition 2 means every message leads to the same ciphertext distribution.
- Therefore seeing ciphertext cannot favor one message over another.
- So posterior probabilities stay equal to prior probabilities.

---

# 20) Another Equivalent View. Game-Based Definition

The lecture also gives a game form.

## Idea
The attacker chooses two equal-length messages `m0, m1`. A random bit `b` is chosen, and the challenger sends encryption of `mb`. If the scheme is perfectly secret, the attacker should do no better than random guessing.

## Win probability
```text
Pr[guess correctly] = 1/2
```

## Important rule
**In perfect secrecy, even the best attacker wins with probability exactly 1/2.**

---

# 21) One-Time Pad (OTP)

This is the star example of perfect secrecy.

## Encryption
```text
Enc_K(m) = K ⊕ m
```

## Decryption
```text
Dec_K(c) = K ⊕ c
```

because XORing twice with the same key cancels out.

## Example
```text
1011 ⊕ 0011 = 1000
```

## Why it works
For any message `m` and ciphertext `c`, there is exactly one key `K = c ⊕ m` that maps `m` to `c`.

## Important rule
**The one-time pad is perfectly secret if the key is:**
- truly random,
- as long as the message,
- used only once,
- kept secret.

---

# 22) Limitations of Perfect Secrecy and OTP

## Theorem from lecture
If a scheme is perfectly secret, then:

```text
|K| ≥ |M|
```

That means the key space must be at least as large as the message space.

## OTP limitations
1. Key must be as long as the message.  
2. Key must be exchanged securely.  
3. Key cannot be reused.  
4. It is impractical for many long messages.

## Reuse disaster
If the same OTP key is reused on two messages:

```text
c  = m  ⊕ k
c' = m' ⊕ k
```

then:

```text
c ⊕ c' = m ⊕ m'
```

The key disappears, which leaks a relation between the messages.

## Important rule
**Never reuse a one-time pad key.**

## Historical example
VENONA project exploited OTP reuse by Soviet communications.

---

# 23) Shannon’s Theorem

The lecture states a characterization of perfect secrecy when:

```text
|K| = |M| = |C|
```

The scheme is perfectly secret iff:
1. every key is chosen uniformly,  
2. for every message `m` and ciphertext `c`, there is a unique key `k` such that `Enc_k(m)=c`.

## Important rule
**Uniform key choice and unique message-to-ciphertext mapping by keys are central to perfect secrecy in this setting.**

---

# 24) Randomness in Practice

The lecture stresses that crypto assumes access to real randomness.

## Good randomness should be
- unbiased,
- independent,
- hard to predict.

## Practical sources mentioned
- coin flips,
- radioactive decay,
- mouse movement / user input,
- randomness extractors,
- dedicated hardware randomness devices.

## Warning from lecture
`rand()` in standard C library is not suitable for cryptography.

## Important rule
**Weak randomness can destroy security, even if the algorithm itself is good.**

---

# 25) OTP and Active Attacks

Perfect secrecy only models a passive eavesdropper. It does not stop tampering.

## Example from lecture
If:

```text
c = K ⊕ m
```

then the attacker can flip one bit of `c` to get `c'`. After decryption, the same bit of `m` flips.

## Important rule
**OTP protects confidentiality, but by itself it does not protect integrity.**

## Extra exam example
Suppose a bit encodes whether a bank transfer is approved:
- `0` = no
- `1` = yes

If the attacker flips that ciphertext bit, the decrypted bit flips too.

---

# 26) Perfect Secrecy vs Computational Security

## Perfect secrecy
Security does not depend on attacker computing power.
Even an infinitely powerful attacker learns nothing.

## Computational security
Security only holds against computationally bounded attackers.
A super-powerful attacker might break it in theory.

## Important rule
**Perfect secrecy is stronger than computational security, but usually less practical.**

---

# 27) Concrete Security

Concrete security gives bounds like:

> Any attacker running in time at most `t` succeeds with probability at most `ε`.

This is written as `(t, ε)`-security.

## Example idea from lecture
A scheme might resist all attacks under some time limit like `2^60` operations.

## Important rule
**Concrete security gives real-world style security numbers.**

---

# 28) Asymptotic Security

This is the main formal style used later in the course.

## Big idea
A scheme is secure if every probabilistic polynomial-time attacker succeeds only with negligible probability.

### Two central concepts
1. **PPT attacker**: runs in probabilistic polynomial time.  
2. **Negligible function**: becomes smaller than `1/p(n)` for every positive polynomial `p(n)` once `n` is large enough.

## Negligible definition
A function `f(n)` is negligible if for every polynomial `p(n) > 0`, there exists `N` such that for all `n > N`:

```text
f(n) < 1 / p(n)
```

## Important rule
Exponential decay like `2^-n` is negligible.  
Inverse polynomial like `1/n^5` is **not** negligible.

## Easy classification guide
Usually:
- `2^-n` -> negligible  
- `2^{-sqrt(n)}` -> negligible  
- `1/n` -> not negligible  
- `1/n^5` -> not negligible  
- `2^{-log n}` -> not negligible, because it behaves like `1/n`

## Extra exam example
Is `f(n)=3^{-n}` negligible?  
Yes, because it decreases exponentially.

Is `g(n)=1/n^100` negligible?  
No, because it is still inverse polynomial.

---

# 29) Polynomial Time

An attacker is efficient if it runs in polynomial time in the security parameter `n`.

## Important rule
**Brute force over a key space of size `2^n` is not polynomial time.**

## Lecture examples
- trying all keys is exponential,  
- guessing one random key succeeds with probability `2^-n`, which is negligible.

---

# 30) Why Asymptotic Definitions Are Useful

## Advantages
- closure properties are nice,  
- reductions compose well,  
- independent of machine details,  
- supports theory cleanly.

## Disadvantage
It does not directly tell you the best security parameter for real systems.

## Important rule
**Asymptotic security is great for theory. Concrete security is great for practical estimation.**

---

# 31) Revised Private-Key Encryption Syntax

Later definitions include the security parameter explicitly.

## Algorithms
- `Gen(1^n; R)` outputs key `k`
- `Enc_k(m; R)` encrypts using randomness
- `Dec_k(c)` decrypts

## Important rule
**In modern private-key encryption, encryption is often randomized.**

That randomness is one reason we can achieve stronger security than classical ciphers.

---

# 32) EAV Security / Indistinguishability Experiment

This is the lecture’s computational security notion against an eavesdropper.

## Game
1. attacker chooses two equal-length messages `m0, m1`,  
2. challenger picks random bit `b`,  
3. challenger sends `c = Enc_k(mb)`,  
4. attacker outputs guess `b'`.

## Secure if
For every PPT attacker, there exists negligible `μ(n)` such that:

```text
Pr[b' = b] ≤ 1/2 + μ(n)
```

## Important rule
**A secure scheme should make the attacker do only negligibly better than random guessing.**

## Why equal lengths?
If messages have different lengths, ciphertext length may reveal which one was encrypted.

---

# 33) Message Length Limitation

The lecture notes that hiding all information about plaintext length is impossible in general.

## Important rule
**In indistinguishability games, we usually require `|m0| = |m1|`.**

## Why this matters
Length itself can leak information.

### Examples from lecture
- 5-digit salary vs 6-digit salary  
- database search result size  
- compression leaks redundancy

---

# 34) How to Solve Common Exam Questions from Lectures 1 & 2

## A) Shift cipher encryption/decryption
### Method
1. map letters to numbers `A=0, ..., Z=25`,  
2. use mod 26 arithmetic,  
3. encrypt with `+k`, decrypt with `-k`,  
4. convert back to letters.

### Rule
**Always wrap around mod 26.**

---

## B) Brute force on classical ciphers
### Method
1. identify the key space size,  
2. check whether exhaustive search is feasible,  
3. test candidate keys until plaintext is meaningful.

### Rule
**If the key space is tiny, brute force is likely the intended attack.**

---

## C) Frequency analysis questions
### Method
1. count frequent letters,  
2. compare with English frequencies,  
3. test common short words and repeated patterns,  
4. refine mapping gradually.

### Rule
**Substitution ciphers preserve frequency information.**

---

## D) Bayes theorem / posterior probability questions
### Method
Use:

```text
Pr[M=m | C=c] = Pr[C=c | M=m] Pr[M=m] / Pr[C=c]
```

Then compute `Pr[C=c]` by total probability.

### Rule
**If posterior changes after seeing ciphertext, the scheme is not perfectly secret.**

---

## E) Perfect secrecy proof questions
### What to show
You usually show one of these:

1. `Pr[M=m | C=c] = Pr[M=m]`, or  
2. `Pr[Enc_k(m)=c] = Pr[Enc_k(m')=c]` for all `m,m',c`.

### Rule
**Definition 2 is usually easier for proving a scheme is perfectly secret.**

---

## F) Negligible vs non-negligible questions
### Method
Check the growth rate.

- exponential decay -> negligible  
- inverse polynomial -> not negligible

### Rule
**Do not confuse “small” with “negligible”.**  
For example, `1/n^100` is very small, but still not negligible in cryptography’s formal sense.

---

# 35) Final Big-Picture Summary

## Classical world
- Shift cipher, substitution cipher, Vigenère, rotor machines  
- clever, historical, but broken

## Modern world
- formal security definitions  
- explicit attacker models  
- mathematical proofs  
- randomized algorithms  
- precise notions like perfect secrecy and EAV security

## Most important ideas to remember
1. **Security must be defined formally.**  
2. **Threat model matters.**  
3. **Large key space alone is not enough.**  
4. **Perfect secrecy means ciphertext gives no extra information.**  
5. **OTP is perfectly secret, but impractical and fragile if reused.**  
6. **Modern computational security means PPT attackers succeed only with negligible probability.**

---

# 36) Ultra-Short Revision Sheet

## Definitions to memorize
- confidentiality  
- integrity  
- passive attacker  
- active attacker  
- Kerckhoffs’s principle  
- private-key encryption scheme `(Gen, Enc, Dec)`  
- perfect secrecy  
- negligible function  
- PPT adversary  
- EAV security

## Rules to memorize
- `Dec_k(Enc_k(m)) = m`  
- OTP: `Enc_k(m)=k⊕m`, `Dec_k(c)=k⊕c`  
- perfect secrecy: `Pr[M=m|C=c]=Pr[M=m]`  
- EAV security: attacker wins with at most `1/2 + negligible`

## Common traps
- large key space does not guarantee security  
- OTP reused twice is broken  
- perfect secrecy does not provide integrity  
- message length can leak information  
- `1/n^5` is not negligible

---

If you continue with the next lecture, keep this mental flow:

```text
classical ciphers -> formal definitions -> perfect secrecy -> OTP limits -> computational security -> indistinguishability games
```
