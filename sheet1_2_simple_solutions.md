# Cryptography. Sheet 1 and Sheet 2. Simple Solutions

This file gives each question first, then the answer, then a simple explanation, and finally a short method you can use to solve similar questions.

---

# Sheet 1

## Question 1
**Give examples for classical encryption techniques.**

### Answer
Examples of classical encryption techniques include:
- Caesar cipher (shift cipher)
- Monoalphabetic substitution cipher
- Vigenère cipher
- Playfair cipher
- Transposition cipher
- Rotor machine encryption, such as Enigma

### Explanation
These are called **classical** techniques because they were used before modern mathematical cryptography became standard.

- In the **Caesar cipher**, each letter is shifted by a fixed number.
- In a **substitution cipher**, each letter is replaced by another letter according to a fixed mapping.
- In the **Vigenère cipher**, the shift changes from letter to letter depending on the key.
- In **transposition ciphers**, the letters themselves stay the same, but their positions are rearranged.
- **Rotor machines** are mechanical systems that apply changing substitutions.

### Steps to solve similar questions
1. Recall the most common historical ciphers.
2. Separate them into categories if needed:
   - substitution-based
   - transposition-based
   - machine-based
3. Give 3 to 6 clear examples.
4. If the question says “explain,” add one short sentence for each.

---

## Question 2
**Explain how the Caesar cipher works, and how to break it.**

### Answer
The Caesar cipher works by shifting every plaintext letter by the same fixed number `k`.

If we map letters as:
- `A = 0, B = 1, ..., Z = 25`

then encryption is:

```text
C = (M + k) mod 26
```

and decryption is:

```text
M = (C - k) mod 26
```

It is broken by trying all 26 possible shifts, or by frequency analysis.

### Explanation
The important idea is that the Caesar cipher uses **one fixed shift for the whole message**.
That makes the key space very small, only 26 possible keys.
So an attacker can simply try all keys until one result makes sense.

Example:
- Plaintext: `HELLO`
- Key: `k = 3`
- Ciphertext: `KHOOR`

Breaking it:
- Try shift 0, 1, 2, 3, ... , 25
- One output will look like readable English
- That is the correct plaintext

This is why Caesar cipher is not secure.

### Steps to solve similar questions
1. Write the alphabet mapping `A=0` to `Z=25`.
2. Use the formula `C = (M + k) mod 26` for encryption.
3. Use the formula `M = (C - k) mod 26` for decryption.
4. If the key is unknown, try all 26 possibilities.
5. Pick the result that forms meaningful text.

---

## Question 3(a)
**Using the Vigenère cipher, encrypt the plaintext `sendmoremoney` with the key stream `9 0 1 7 23 15 21 14 11 11 2 8 9`.**

### Answer
Using `a=0, b=1, ..., z=25`, we convert:

- Plaintext `sendmoremoney` to numbers:

```text
s e n d m o r e m o n e y
18 4 13 3 12 14 17 4 12 14 13 4 24
```

- Key stream:

```text
9 0 1 7 23 15 21 14 11 11 2 8 9
```

Now encrypt letter by letter:

```text
(18+9) mod 26 = 1   -> B
(4+0) mod 26 = 4    -> E
(13+1) mod 26 = 14  -> O
(3+7) mod 26 = 10   -> K
(12+23) mod 26 = 9  -> J
(14+15) mod 26 = 3  -> D
(17+21) mod 26 = 12 -> M
(4+14) mod 26 = 18  -> S
(12+11) mod 26 = 23 -> X
(14+11) mod 26 = 25 -> Z
(13+2) mod 26 = 15  -> P
(4+8) mod 26 = 12   -> M
(24+9) mod 26 = 7   -> H
```

So the ciphertext is:

```text
BEOKJDMSXZPMH
```

### Explanation
Vigenère cipher is like Caesar cipher, but the shift is **different for each letter**.
Instead of one fixed key value, you use a sequence of key values.
So each position is encrypted using:

```text
c_i = (p_i + k_i) mod 26
```

### Steps to solve similar questions
1. Convert plaintext letters into numbers.
2. Write the key value under each letter.
3. Add plaintext value and key value.
4. Reduce modulo 26.
5. Convert the result back to letters.

---

## Question 3(b)
**Using the ciphertext from part (a), find a key so that the ciphertext decrypts to the plaintext `cashnotneeded`.**

### Answer
We have ciphertext:

```text
BEOKJDMSXZPMH
```

Target plaintext:

```text
cashnotneeded
```

Convert target plaintext to numbers:

```text
c a s h n o t n e e d e d
2 0 18 7 13 14 19 13 4 4 3 4 3
```

Convert ciphertext to numbers:

```text
B E O K J D M S X Z P M H
1 4 14 10 9 3 12 18 23 25 15 12 7
```

Since decryption is:

```text
p_i = (c_i - k_i) mod 26
```

then the needed key is:

```text
k_i = (c_i - p_i) mod 26
```

Compute:

```text
(1-2) mod 26 = 25
(4-0) mod 26 = 4
(14-18) mod 26 = 22
(10-7) mod 26 = 3
(9-13) mod 26 = 22
(3-14) mod 26 = 15
(12-19) mod 26 = 19
(18-13) mod 26 = 5
(23-4) mod 26 = 19
(25-4) mod 26 = 21
(15-3) mod 26 = 12
(12-4) mod 26 = 8
(7-3) mod 26 = 4
```

So one valid key stream is:

```text
25 4 22 3 22 15 19 5 19 21 12 8 4
```

### Explanation
Here the ciphertext is fixed, and the plaintext you want is also fixed.
So instead of using the key to get ciphertext, you work backward to recover the key.

The rule is:

```text
k_i = (c_i - p_i) mod 26
```

### Steps to solve similar questions
1. Convert ciphertext letters to numbers.
2. Convert desired plaintext letters to numbers.
3. Compute `k_i = (c_i - p_i) mod 26` for each position.
4. Write the sequence of values as the key stream.
5. Check by decrypting once more.

---

## Question 4
**Provide a formal definition of the Gen, Enc, and Dec algorithms for the Vigenère cipher.**

### Answer
One valid formal definition is:

### Gen
Choose a period `t` and choose a key sequence

```text
k = (k_0, k_1, ..., k_{t-1})
```

where each `k_i` is chosen uniformly from `{0,1,...,25}`.

### Enc
Given plaintext

```text
p = p_0, p_1, ..., p_n
```

and key

```text
k = k_0, ..., k_{t-1}
```

compute:

```text
c_i = (p_i + k_{i mod t}) mod 26
```

Output:

```text
c = c_0, c_1, ..., c_n
```

### Dec
Given ciphertext

```text
c = c_0, c_1, ..., c_n
```

and the same key,

compute:

```text
p_i = (c_i - k_{i mod t}) mod 26
```

Output:

```text
p = p_0, p_1, ..., p_n
```

### Explanation
A formal definition must clearly state:
- what the key is
- how the key is generated
- how encryption is computed
- how decryption is computed

The key repeats every `t` positions, so this is a periodic sequence.

### Steps to solve similar questions
1. Define the message space and key space.
2. Define `Gen` clearly.
3. Define `Enc` as a mathematical rule.
4. Define `Dec` as the inverse rule.
5. Make sure `Dec(Enc(m)) = m`.

---

## Question 5
**Write a program that can perform a letter frequency attack on monoalphabetic substitution cipher (shift cipher) without human intervention.**

### Answer
The idea of the program is:
1. Take a ciphertext as input.
2. Try all 26 shifts.
3. Score each resulting plaintext according to how likely it is to be English.
4. Output the best candidates in sorted order.

A simple scoring method is:
- count common English letters such as `E, T, A, O, I, N`
- reward common words such as `the`, `and`, `is`
- penalize nonsense letter combinations

### Explanation
For a shift cipher, full automation is easy because there are only 26 possibilities.
The program decrypts using each possible key and ranks the results.

This is called a frequency or likelihood attack because English letters do not appear equally often.

### Steps to solve similar questions
1. Identify the key space size.
2. Generate all possible decryptions.
3. Score each candidate.
4. Sort the outputs by score.
5. Return the top results.

---

# Sheet 2

## Question 2.3
**Prove or refute: An encryption scheme with message space `M` is perfectly secret if and only if for every probability distribution over `M` and every `c_0, c_1 ∈ C` we have `Pr[C = c_0] = Pr[C = c_1]`.**

### Answer
This statement is **false**.

### Explanation
Perfect secrecy means that the ciphertext should not reveal information about the plaintext.
It does **not** mean that all ciphertexts must appear with exactly the same probability.

A counterexample is a modified one-time pad where, after normal encryption, we append one extra bit such that:
- appended bit = 0 with probability `1/4`
- appended bit = 1 with probability `3/4`

This scheme is still perfectly secret, because the extra bit is independent of the plaintext.
But ciphertexts ending in `1` are more likely than ciphertexts ending in `0`.
So:

```text
Pr[C = c_0] ≠ Pr[C = c_1]
```

for some ciphertexts.

Hence the statement is false.

### Steps to solve similar questions
1. Identify what the claim says.
2. Check whether it matches the real definition of perfect secrecy.
3. If it feels too strong, try a counterexample.
4. Use a scheme where extra randomness changes ciphertext probabilities but not plaintext leakage.
5. Conclude whether the claim is true or false.

---

## Question 2.6(a)
**The message space is `M = {0, ..., 4}`. Algorithm Gen chooses a uniform key from `{0, ..., 5}`. `Enc_k(m)` returns `[k + m mod 5]`, and `Dec_k(c)` returns `c - k mod 5`. Is the scheme perfectly secret?**

### Answer
The scheme is **not perfectly secret**.

### Explanation
To test perfect secrecy, we can compare ciphertext probabilities for two different messages.
Take ciphertext `0`.

If the message is `0`, then:

```text
Enc_k(0) = 0  iff  k mod 5 = 0
```

Since `k ∈ {0,1,2,3,4,5}`, this happens when `k = 0` or `k = 5`.
So:

```text
Pr[Enc_k(0) = 0] = 2/6 = 1/3
```

If the message is `1`, then:

```text
Enc_k(1) = 0  iff  (k + 1) mod 5 = 0
```

This happens only when `k = 4`.
So:

```text
Pr[Enc_k(1) = 0] = 1/6
```

These probabilities are not equal, so the ciphertext distribution depends on the message.
Therefore the scheme is not perfectly secret.

### Steps to solve similar questions
1. Pick two different messages.
2. Pick one ciphertext value.
3. Compute the probability of getting that ciphertext under each message.
4. If the probabilities differ, the scheme is not perfectly secret.
5. State the conclusion clearly.

---

## Question 2.6(b)
**The message space is `M = {m ∈ {0,1}^ℓ :` the last bit of `m` is `0`}. Gen chooses a uniform key from `{0,1}^{ℓ-1}`. `Enc_k(m)` returns `m ⊕ (k||0)`, and `Dec_k(c)` returns `c ⊕ (k||0)`. Is the scheme perfectly secret?**

### Answer
Yes. The scheme **is perfectly secret**.

### Explanation
Every message in the space has last bit `0`.
The key is an `(ℓ-1)`-bit string, and encryption XORs the message with `(k || 0)`.
So the first `ℓ-1` bits are masked by a uniform random string, while the last bit is always `0` in every allowed plaintext anyway.

This behaves like a one-time pad on the meaningful part of the message space.
Since the part that varies is perfectly masked, the ciphertext reveals no information about which valid message was encrypted.

### Steps to solve similar questions
1. Identify what part of the message actually varies.
2. Check whether that part is masked by a uniform random key.
3. Compare the construction to the one-time pad.
4. If every varying bit is hidden uniformly, the scheme is perfectly secret.
5. Mention any fixed bits and why they do not matter.

---

## Question 1(a)
**For the shift cipher, is it perfectly secret?**

### Answer
No. The shift cipher is **not perfectly secret**.

### Explanation
Even though the key is chosen uniformly from 26 possibilities, the same key is used for the entire message.
That means the attacker can try all 26 possible shifts and recover the plaintext.

Also, the shift cipher preserves structure:
- repeated letters remain repeated
- English-looking outputs appear after a small search

So it is not perfectly secret, and in practice it is not secure at all.

### Steps to solve similar questions
1. Ask whether one key is reused over the full message.
2. Check whether the key space is small.
3. Ask if ciphertext patterns still reflect plaintext structure.
4. If yes, it is not perfectly secret.
5. If you can brute force easily, mention that too.

---

## Question 1(b)
**Can you modify the shift cipher description to make it perfectly secret?**

### Answer
Yes. Use a **different random key for each character**.

Instead of one key `k`, choose a key vector:

```text
k = (k_1, k_2, ..., k_n)
```

where each `k_i` is chosen independently and uniformly from `{0,1,...,25}`.
Then encrypt each character as:

```text
c_i = (m_i + k_i) mod 26
```

This modified scheme is perfectly secret.

### Explanation
The ordinary shift cipher fails because the same shift is reused.
If each character gets its own fresh random shift, then every plaintext letter is independently masked.
That is exactly the same idea as a one-time pad, but over alphabet symbols instead of bits.

For any fixed plaintext letter `m_i` and ciphertext letter `c_i`, there is exactly one key value:

```text
k_i = (c_i - m_i) mod 26
```

Because each key value is chosen uniformly, every ciphertext is equally likely for every plaintext.
That gives perfect secrecy.

### Steps to solve similar questions
1. Identify why the original cipher fails.
2. Usually the reason is key reuse.
3. Replace the reused key with fresh randomness per symbol.
4. Show that for every plaintext-ciphertext pair there is exactly one key.
5. Conclude perfect secrecy.

---

## Question 2
**Prove or refute: Every encryption scheme for which the size of the key space `K` equals the size of the message space `M`, and for which the key is chosen uniformly from `K`, is perfectly secret.**

### Answer
This statement is **false**.

### Explanation
Having:

```text
|K| = |M|
```

and choosing the key uniformly is **not enough** for perfect secrecy.
You also need the encryption behavior to satisfy the perfect secrecy condition.

Counterexample:
- Message space `M = {a, b}`
- Key space `K = {k_1, k_2}`
- Ciphertext space `C = {0, 1}`
- Let encryption always output:
  - `0` when the plaintext is `a`
  - `1` when the plaintext is `b`

This scheme is correct because decryption can recover the message from the ciphertext.
But:

```text
Pr[C = 0 | M = a] = 1
Pr[C = 0 | M = b] = 0
```

So the ciphertext completely reveals the plaintext.
Therefore the scheme is not perfectly secret.

### Steps to solve similar questions
1. Check whether the claim gives only size conditions.
2. Remember: size conditions alone do not guarantee secrecy.
3. Build a counterexample where encryption leaks the message directly.
4. Verify that the scheme is still correct.
5. Show that ciphertext probabilities depend on the plaintext.

---

# Quick Summary of Main Ideas

- **Classical ciphers** include Caesar, substitution, Vigenère, transposition, and rotor-based systems.
- **Caesar cipher** is broken by brute force because the key space is tiny.
- **Vigenère cipher** uses a different shift for each position.
- **Perfect secrecy** means ciphertext gives no information about plaintext.
- Equal key-space size and message-space size is **not sufficient** by itself.
- Reusing one key across many symbols often breaks secrecy.
- Fresh randomness per symbol can turn a weak scheme into a perfectly secret one.

