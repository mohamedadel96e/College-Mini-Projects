---
created: 2026-06-19T18:14
updated: 2026-06-19T18:15
---
# Cryptography. Sheet 6 and Sheet 7. Simple Solutions

This file gives:
- the question
- the answer
- a brief explanation
- the steps to solve similar questions

For **Sheet 7**, the uploaded PDF contained only a title page and a blank page, so there were no visible questions to solve.

---

# Sheet 6

## Q1. Illustrate how the Diffie-Hellman key exchange algorithm works.

### Answer
Diffie-Hellman lets two parties create the same shared secret key over a public channel.

The basic idea is:

1. Both parties agree on public values:
   - a prime number $p$
   - a generator $g$

2. Alice chooses a private secret $a$ and computes:
   - public value $A = g^a mod p$

3. Bob chooses a private secret $b$ and computes:
   - public value $B = g^b mod p$

4. They exchange $A$ and $B$ publicly.

5. Alice computes:
   - shared key $K = B^a mod p$

6. Bob computes:
   - shared key $K = A^b mod p$

Both keys are equal because:

- $B^a = (g^b)^a = g^{ab}$
- $A^b = (g^a)^b = g^{ab}$

So both get the same secret.

### Brief explanation
The public values $p$ and $g$ do not reveal the private exponents.  
Even though everyone sees $A$ and $B$, only Alice and Bob can compute the shared secret.

### Steps to solve similar questions
1. Write the public parameters $p$ and $g$.
2. Compute each person’s public key using $g^{private} mod p$.
3. Exchange the public keys.
4. Raise the other party’s public key to your private exponent.
5. Check that both sides give the same result.

---

## Q2. Perform a Diffie-Hellman key exchange between Alice and Bob with $p = 21929$ and $g = 3$. Alice’s private key is 4 and Bob’s private key is 6.

### Answer
- Alice’s public key: $A = 81$
- Bob’s public key: $B = 729$
- Shared secret key: $K = 19899$

### Brief explanation
We compute each public key using modular exponentiation.

#### Alice’s public key
$A = 3^4 mod 21929 = 81$

#### Bob’s public key
$B = 3^6 mod 21929 = 729$

#### Shared secret
Alice computes:

$K = B^4 mod 21929 = 729^4 mod 21929 = 19899$

Bob computes:

$K = A^6 mod 21929 = 81^6 mod 21929 = 19899$

So both obtain the same secret key.

### Steps to solve similar questions
1. Compute the public keys:
   - $A = g^a mod p$
   - $B = g^b mod p$
2. Compute the shared key using the other party’s public key:
   - $K = B^a mod p$
   - or $K = A^b mod p$
3. Verify that both sides match.

---

## Q3. Work out a method by which n people can use Diffie-Hellman to create a secure key.

### Answer
For a group of $n$ people, one common idea is to let each person raise the current value to their private exponent in turn.

Suppose the public values are:
- $p$
- $g$

and the private keys are:
- person 1: $x_1$
- person 2: $x_2$
- ...
- person n: $x_n$

Then the final shared key becomes:

$K = g^{x_1 x_2 ... x_n} mod p$

### Brief explanation
Each person contributes one secret exponent.  
After all exponents are applied, everyone ends at the same final value.

### Simple method
One person starts with $g$ and sends it around the group.
Each person does:
- raise the received value to their private exponent
- pass the result to the next person

At the end, the value is raised by all private exponents, so the result is shared by everyone.

### Steps to solve similar questions
1. Write down $p$, $g$, and all private exponents.
2. Start from $g$.
3. Apply each private exponent one by one.
4. Reduce everything modulo $p$.
5. The final value is the group shared key.

---

## Q4. Perform the 3-person Diffie-Hellman exchange with $p = 23$, $g = 5$, Alice’s private key 4, Bob’s private key 3, and Charlie’s private key 6.

### Answer
The shared secret key is:

$K = 8$

### Brief explanation
The combined exponent is:

$4 \times 3 \times 6 = 72$

So the shared key is:

$5^{72} mod 23 = 8$

This matches the answer given in the sheet.

### Step-by-step computation
Using the group idea:

1. Start with the public generator:
   - $5$

2. Alice raises it to 4:
   - $5^4 mod 23 = 4$

3. Bob raises that result to 3:
   - $4^3 mod 23 = 18$

4. Charlie raises that result to 6:
   - $18^6 mod 23 = 8$

So the final shared secret is:

$K = 8$

### Steps to solve similar questions
1. Compute the combined exponent product.
2. Calculate $g^{x_1 x_2 ... x_n} mod p$.
3. If needed, simulate the value passing through each person step by step.
4. Confirm the same final key is obtained.

---
