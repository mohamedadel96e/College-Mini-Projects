---
created: 2026-04-05T07:15
updated: 2026-06-19T22:33
---
# Cryptography Sheets 3 & 4 - Simple Solutions

---

# Sheet 3

## Q1. Explain the Multiple-Message Attack Model

### Answer

The multiple-message attack model studies whether an encryption scheme stays secure when the attacker sees **more than one ciphertext**, not just one.

In this model, the attacker chooses two lists of messages of the same lengths:

- List 0: $(m_{0,1}, m_{0,2}, \dots, m_{0,t})$
- List 1: $(m_{1,1}, m_{1,2}, \dots, m_{1,t})$

A hidden bit $b \in \{0,1\}$ is chosen, and the challenger encrypts **all messages from one list only**. The attacker receives the ciphertexts and tries to guess whether the encrypted list was List 0 or List 1.

If the attacker cannot do better than random guessing, then the scheme is secure in the multiple-message eavesdropping sense.

### Explanation

The main idea is:

- In the single-message model, the attacker sees only **one ciphertext**.
- In the multiple-message model, the attacker sees **many ciphertexts**.
- Sometimes a scheme may appear secure for one ciphertext but reveal patterns when many ciphertexts are observed.

This model is important because real systems usually encrypt many messages.

### Steps to Solve Similar Questions

1. Identify what the attacker is allowed to see.
2. Check whether the attacker chooses one pair of messages or two lists.
3. Mention that a secret bit $b$ is chosen.
4. Explain that the challenger encrypts one whole list based on $b$.
5. State the goal: the attacker tries to guess $b$.
6. Conclude that security means the attacker cannot do better than random guessing.

---

## Q2. Explain the Chosen-Plaintext Attack (CPA) Model

### Answer

In the chosen-plaintext attack model, the attacker can request encryptions of plaintexts of its own choice.

The attacker interacts with an encryption oracle and then tries to distinguish which of two chosen messages was encrypted.

If the attacker cannot distinguish better than random guessing, the scheme is CPA-secure.

### Explanation

This model is stronger than simple eavesdropping because the attacker is active.

The attacker can:

- Choose messages.
- Request their encryptions.
- Analyze the resulting ciphertexts.
- Use this information to attack a challenge ciphertext.

### Steps to Solve Similar Questions

1. State that the attacker chooses plaintexts.
2. Mention the encryption oracle.
3. Explain why this is stronger than passive eavesdropping.
4. Define the challenge messages $m_0$ and $m_1$.
5. State that the challenger encrypts one of them.
6. Explain that the attacker tries to guess the hidden bit $b$.
7. Conclude with CPA security.

---

## Q3. Explain Stream Ciphers and Their Types

### Answer

A stream cipher encrypts data bit-by-bit (or symbol-by-symbol) using a keystream.

General formula:

$$
c_i = m_i \oplus k_i
$$

where:

- $m_i$ = plaintext symbol
- $k_i$ = keystream symbol
- $c_i$ = ciphertext symbol

### Types of Stream Ciphers

#### 1. Synchronous Stream Cipher

The keystream depends only on:

- Secret key
- Internal state

It does **not** depend on plaintext or ciphertext.

#### 2. Self-Synchronizing Stream Cipher

The keystream depends on:

- Secret key
- Previous ciphertext symbols

The receiver can automatically regain synchronization after receiving enough ciphertext.

### Explanation

A stream cipher:

1. Generates a pseudorandom keystream.
2. XORs it with the plaintext.

This resembles the One-Time Pad, except the keystream is generated algorithmically.

### Steps to Solve Similar Questions

1. Define the stream cipher.
2. Write:

$$
c_i = m_i \oplus k_i
$$

3. Explain the keystream.
4. Describe synchronous and self-synchronizing types.
5. Compare their synchronization behavior.

---

## Q4. Simplified RC4 Example

### Question

Use simplified RC4 with:

- Key: $K=[1,2,3,6]$
- Plaintext: $P=[1,2,2,2]$

### Answer

The ciphertext is:

$$
C=[4,1,2,0]
$$

### Explanation

After key scheduling:

$$
S=[2,3,7,4,0,1,6,5]
$$

Generated keystream:

$$
k_1=5,\quad
k_2=3,\quad
k_3=0,\quad
k_4=2
$$

Encryption:

$$
5 \oplus 1 = 4
$$

$$
3 \oplus 2 = 1
$$

$$
0 \oplus 2 = 2
$$

$$
2 \oplus 2 = 0
$$

Therefore:

$$
C=[4,1,2,0]
$$

### Steps to Solve Similar Questions

1. Perform KSA (Key Scheduling Algorithm).
2. Generate the keystream.
3. XOR each plaintext value with the corresponding keystream value.
4. Collect the ciphertext.

---

# Sheet 4

## Q1. Explain How Decryption Works in ECB, CBC, and CTR

### 1. ECB Mode

#### Answer

Encryption:

$$
c_i = E_k(m_i)
$$

Decryption:

$$
m_i = D_k(c_i)
$$

#### Explanation

Each block is processed independently.

#### Steps

1. Apply the inverse block cipher.
2. Decrypt each block separately.

---

### 2. CBC Mode

#### Answer

For block $i$:

$$
m_i = D_k(c_i) \oplus c_{i-1}
$$

For the first block:

$$
m_1 = D_k(c_1) \oplus IV
$$

#### Explanation

CBC requires the previous ciphertext block (or IV) during decryption.

#### Steps

1. Compute $D_k(c_i)$.
2. XOR with:
   - $IV$ if first block.
   - $c_{i-1}$ otherwise.

---

### 3. CTR Mode

#### Answer

Encryption:

$$
c_i = m_i \oplus F_k(\text{ctr}+i)
$$

Decryption:

$$
m_i = c_i \oplus F_k(\text{ctr}+i)
$$

#### Explanation

CTR behaves like a stream cipher.

Because XOR is its own inverse:

$$
a \oplus b \oplus b = a
$$

#### Steps

1. Generate the same keystream.
2. XOR it with ciphertext.
3. Recover plaintext.

---

## Q2. Show That CBC Is Not CPA-Secure if the IV Is Predictable

### Answer

CBC encryption for the first block is:

$$
C_1 = E_k(IV \oplus M_1)
$$

Assume the attacker knows the next IV.

Choose:

$$
M_1 = IV \oplus P
$$

Substituting:

$$
C_1
=
E_k(IV \oplus IV \oplus P)
=
E_k(P)
$$

Since:

$$
IV \oplus IV = 0
$$

the attacker cancels the IV completely.

### Conclusion

The encryption becomes deterministic:

$$
C_1 = E_k(P)
$$

This breaks CPA security.

### Steps

1. Start with:

$$
C_1=E_k(IV\oplus M_1)
$$

2. Assume the IV is known.
3. Choose:

$$
M_1=IV\oplus P
$$

4. Substitute.
5. Show the IV cancels.
6. Conclude that CPA security fails.

---

## Q3. Show That CTR Mode Is Not CCA-Secure

### Answer

CTR encryption:

$$
c_i=m_i\oplus F_k(\text{ctr}+i)
$$

Modify the ciphertext:

$$
c_i' = c_i \oplus \Delta
$$

Decrypt:

$$
m_i'
=
c_i'
\oplus F_k(\text{ctr}+i)
$$

Substitute:

$$
m_i'
=
(c_i\oplus\Delta)
\oplus F_k(\text{ctr}+i)
$$

Since:

$$
c_i=m_i\oplus F_k(\text{ctr}+i)
$$

we obtain:

$$
m_i'
=
m_i\oplus\Delta
$$

### Explanation

The attacker can flip chosen bits in the plaintext by flipping the same bits in the ciphertext.

This property is called **malleability**.

### Conclusion

CTR mode is not secure against chosen-ciphertext attacks (CCA).

### Steps

1. Write the CTR formula.
2. Modify ciphertext using:

$$
c' = c \oplus \Delta
$$

3. Decrypt.
4. Simplify.
5. Show:

$$
m' = m \oplus \Delta
$$

6. Conclude that CTR is malleable and therefore not CCA-secure.

---

# Final Quick Summary

## Sheet 3

- Multiple-message attack: attacker sees many ciphertexts.
- CPA model: attacker chooses plaintexts.
- Stream ciphers use XOR with a keystream.
- RC4 generates a keystream then XORs it with plaintext.

## Sheet 4

- ECB: decrypt each block independently.
- CBC: decrypt then XOR with previous ciphertext (or IV).
- CTR: XOR ciphertext with the same keystream.
- Predictable IV breaks CBC CPA-security.
- CTR is not CCA-secure because it is malleable.