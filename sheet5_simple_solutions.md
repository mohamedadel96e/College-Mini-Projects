# Cryptography. Sheet 5. Simple Solutions

This file follows the same style as the simple files for Sheets 1 & 2 and Sheets 3 & 4:

- question
- answer
- simple explanation
- steps to solve similar questions

---

# Sheet 5

## Question 4.6
**Consider the following MAC for messages of length l(n) = 2n - 2 using a pseudorandom function F. On input a message m0 || m1 with |m0| = |m1| = n - 1 and key k in {0,1}^n, algorithm Mac outputs**

```text
t = F_k(0 || m0) || F_k(1 || m1)
```

**Algorithm Vrfy is defined in the natural way. Is (Gen, Mac, Vrfy) secure? Prove your answer.**

### Answer
No. This MAC is **not secure**.

### Explanation
The tag is split into two independent parts:

```text
left part  = F_k(0 || m0)
right part = F_k(1 || m1)
```

So the first half of the tag depends only on `m0`, and the second half depends only on `m1`.

That means an attacker can ask for tags on two different messages:

```text
m  = m0 || m1
m' = m0' || m1'
```

and receive:

```text
t  = t0 || t1
t' = t0' || t1'
```

Then the attacker can build a new message:

```text
m_tilde = m0 || m1'
```

and output the forged tag:

```text
t_tilde = t0 || t1'
```

Why does this work?

- `t0` is correct for `m0`
- `t1'` is correct for `m1'`

So together they form a valid tag for `m0 || m1'`.

This is a successful forgery on a new message, so the MAC is insecure.

### Steps to solve similar questions
1. Check whether the tag is built from independent parts.
2. Ask: can I take one part from one valid tag and another part from another valid tag?
3. Try to combine them into a new valid tag for a new message.
4. If yes, then the MAC is insecure because of a cut-and-paste forgery.

---

## Question 4.7(a)
**Let F be a pseudorandom function. Show that the following MAC is insecure, even for fixed-length messages:**

```text
t = F_k(m1) XOR ... XOR F_k(m_ell)
```

### Answer
This MAC is **insecure**.

### Explanation
XOR does not care about order.

So for two-block messages:

```text
tag(m1, m2) = F_k(m1) XOR F_k(m2)
tag(m2, m1) = F_k(m2) XOR F_k(m1)
```

These two tags are equal.

That means if an attacker asks for the tag of:

```text
(m1, m2)
```

then the same tag is also valid for:

```text
(m2, m1)
```

as long as `m1 != m2`.

So the attacker can forge a valid tag for a new message just by swapping the blocks.

### Steps to solve similar questions
1. Look at the operation used to combine block tags.
2. If the operation is commutative, like XOR, test whether reordering blocks keeps the same tag.
3. If reordering produces the same tag, then the MAC is insecure.
4. Show one concrete forgery using two different blocks.

---

## Question 4.7(b)
**Let F be a pseudorandom function. Show that the following MAC is insecure, even for fixed-length messages:**

```text
t = F_k(<1> || m1) XOR ... XOR F_k(<ell> || m_ell)
```

where `<i>` is the n/2-bit encoding of the integer `i`.

### Answer
This MAC is **insecure**.

### Explanation
Adding the block index prevents the simple reordering attack from part (a), but XOR is still linear, so the attacker can combine several valid tags to create a new one.

Take four blocks of length `n/2`:

```text
m1, m1', m2, m2'
```

with:

```text
m1 != m1'
m2 != m2'
```

Query these three messages:

```text
message A = (m1,  m2)   with tag t1
message B = (m1,  m2')  with tag t2
message C = (m1', m2)   with tag t3
```

Then:

```text
t1 = F_k(<1>||m1)  XOR F_k(<2>||m2)
t2 = F_k(<1>||m1)  XOR F_k(<2>||m2')
t3 = F_k(<1>||m1') XOR F_k(<2>||m2)
```

Now XOR the three tags:

```text
t1 XOR t2 XOR t3
```

The repeated terms cancel, and the result becomes:

```text
F_k(<1>||m1') XOR F_k(<2>||m2')
```

which is exactly the valid tag for the new message:

```text
(m1', m2')
```

So the attacker forges a valid tag on a new message.

### Steps to solve similar questions
1. If tags are combined using XOR, think about cancellation.
2. Query a few related messages whose tags share repeated terms.
3. XOR the tags together and see which terms cancel.
4. If the result becomes the tag of a new message, then the MAC is insecure.

---

## Question 4.7(c)
**Let F be a pseudorandom function. Show that the following MAC is insecure, even for fixed-length messages:**

```text
choose random r from {0,1}^n
t = F_k(r) XOR F_k(<1> || m1) XOR ... XOR F_k(<ell> || m_ell)
```

and let the tag be:

```text
<r, t>
```

### Answer
This MAC is **insecure**.

### Explanation
Take a one-block message `m1`.

The attacker chooses:

```text
r = <1> || m1
```

Now for a one-block message, the tag formula becomes:

```text
t = F_k(r) XOR F_k(<1> || m1)
```

But since:

```text
r = <1> || m1
```

the two terms are equal, so:

```text
t = F_k(<1> || m1) XOR F_k(<1> || m1) = 0^n
```

So the attacker can output the pair:

```text
<r, 0^n>
```

for the message `m1`, and it will verify successfully.

That gives a valid forgery, so the MAC is insecure.

### Steps to solve similar questions
1. Look for a way to choose the random-looking part so that two terms become equal.
2. If the tag uses XOR, equal terms cancel to zero.
3. Try the one-block case first because it is usually the easiest.
4. If you can force the tag into a predictable value like `0^n`, then the scheme is insecure.

---

## Question 4.13(a)
**We explore what happens when the basic CBC-MAC construction is used with messages of different lengths. Say the sender and receiver do not agree on the message length in advance, and the sender only authenticates messages of length 2n. Show that an adversary can forge a valid tag on a message of length 4n.**

### Answer
Basic CBC-MAC is **insecure for variable-length messages**.

### Explanation
Take any two blocks:

```text
m1, m2
```

Ask for the CBC-MAC tag on the 2-block message:

```text
(m1, m2)
```

and receive tag:

```text
t
```

Now consider the 4-block message:

```text
(m1, m2, m1 XOR t, m2)
```

This new message also has valid CBC-MAC tag `t`.

Why?

For CBC-MAC:

- after processing `m1, m2`, the internal chaining value becomes `t`
- then the next block is `m1 XOR t`
- CBC-MAC xors the next block with the current chaining value, so the input becomes:

```text
t XOR (m1 XOR t) = m1
```

So the computation restarts exactly as if it is processing `m1` again.
Then the last block `m2` leads to the same final tag `t`.

So the attacker has forged a valid tag on a new 4-block message.

### Steps to solve similar questions
1. Start from the chaining rule of CBC-MAC.
2. Ask for a tag on a shorter message.
3. Use the returned tag to build an extra block that cancels the chaining value.
4. Continue with old blocks so the computation repeats and ends with the same tag.
5. Conclude that variable-length CBC-MAC is insecure.

---

## Question 4.13(b)
**Say the receiver only accepts 3-block messages, but the sender authenticates messages of any length that is a multiple of n. Show that an adversary can forge a valid tag on a new message.**

### Answer
This construction is **insecure**.

### Explanation
Take any three blocks:

```text
m1, m2, m3
```

The attacker does the following:

1. Obtain tag `t1` on the one-block message:

```text
m1
```

2. Obtain tag `t2` on the one-block message:

```text
t1 XOR m2
```

3. Obtain tag `t3` on the one-block message:

```text
t2 XOR m3
```

Then `t3` is a valid CBC-MAC tag for the 3-block message:

```text
(m1, m2, m3)
```

Why?

CBC-MAC for three blocks works as follows:

```text
state1 = E_k(m1) = t1
state2 = E_k(state1 XOR m2) = E_k(t1 XOR m2) = t2
state3 = E_k(state2 XOR m3) = E_k(t2 XOR m3) = t3
```

So `t3` is exactly the real tag of `(m1, m2, m3)`.

But the attacker never asked for the tag of the 3-block message itself.
They only asked for tags of one-block messages.

So this is a valid forgery on a new accepted message.

### Steps to solve similar questions
1. Write the CBC-MAC chaining process block by block.
2. Notice that each chaining value can be simulated by querying a carefully chosen one-block message.
3. Build the next query using:
   - previous tag XOR next block
4. Continue until you get the final tag.
5. Use that final tag as the forgery on the longer message.

---

## Question 5.3
**Let (Gen, H) be a collision-resistant hash function. Is (Gen, H_hat) defined by**

```text
H_hat(x) = H(H(x))
```

**necessarily collision resistant?**

### Answer
Yes. `H_hat(x) = H(H(x))` is also collision resistant.

### Explanation
Assume there is a collision for `H_hat`, meaning there exist `x` and `x'` such that:

```text
H(H(x)) = H(H(x'))
```

Now there are two cases.

### Case 1
If:

```text
H(x) = H(x')
```

then `x` and `x'` are already a collision for the original hash function `H`.

That is hard to find because `H` is collision resistant.

### Case 2
If:

```text
H(x) != H(x')
```

but still:

```text
H(H(x)) = H(H(x'))
```

then define:

```text
y  = H(x)
y' = H(x')
```

Now:

```text
y != y'
```

and:

```text
H(y) = H(y')
```

So `y` and `y'` are a collision for `H`.

Again, that should be hard because `H` is collision resistant.

In both cases, a collision for `H_hat` gives a collision for `H`.
So if `H` is collision resistant, then `H_hat` is collision resistant too.

### Steps to solve similar questions
1. Assume there is a collision in the new construction.
2. Expand the definition carefully.
3. Split into cases:
   - inner outputs equal
   - inner outputs different
4. In each case, show how that gives a collision for the original hash function.
5. Conclude that breaking the new function would break the original one.

---

# Final Quick Summary

## Main ideas in Sheet 5
- independent tag parts can be cut and pasted
- XOR-based MAC constructions are often insecure because of cancellation
- CBC-MAC is safe only under the right fixed-length conditions
- variable-length CBC-MAC is dangerous
- composition of a collision-resistant hash with itself stays collision resistant
