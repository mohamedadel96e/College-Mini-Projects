---
created: 2026-06-20T11:03
updated: 2026-06-20T11:03
---
### Question 4.6

**Question:** Consider the following MAC for messages of length $l(n) = 2n - 2$ using a pseudorandom function $F$. On input a message $m_0 || m_1$ with $|m_0| = |m1| = n - 1$ and key $k \in \{0,1\}^n$, algorithm Mac outputs

$t = F_k(0 || m_0) || F_k(1 || m_1)$

Algorithm Vrfy is defined in the natural way. Is (Gen, Mac, Vrfy) secure? Prove your answer.

**Answer:** No. This MAC is completely insecure.

**Detailed Explanation:**

To prove a MAC is insecure, we must demonstrate a successful Chosen-Message Attack (CMA). This means an attacker can query the MAC oracle for a few messages and then produce a valid tag for a _new_ message they never queried.

The fatal flaw here is that the tag is generated in two completely isolated, independent halves:

- $t_{left} = F_k(0 || m_0)$
    
- $t_{right} = F_k(1 || m_1)$
    

Because the left half does not cryptographically bind to the right half, an attacker can perform a "Mix-and-Match" (or cut-and-paste) attack.

1. The attacker chooses two completely different messages: $M^{(1)} = m_0 || m_1$ and $M^{(2)} = m'_0 || m'_1$.
    
2. The attacker queries $M^{(1)}$ to the oracle and gets tag $t^{(1)} = t_0 || t_1$.
    
3. The attacker queries $M^{(2)}$ to the oracle and gets tag $t^{(2)} = t'_0 || t'_1$.
    
4. The attacker now constructs a _new_ forged message by mixing the halves: $M^* = m_0 || m'_1$.
    
5. The attacker constructs the forged tag by combining the corresponding halves from the previous queries: $t^* = t_0 || t'_1$.
    

When the verifier checks $M^*$, it computes $F_k(0 || m_0)$ (which matches $t_0$) and $F_k(1 || m'_1)$ (which matches $t'_1$). The verifier accepts the tag as valid. Since $M^*$ was never explicitly queried, the adversary wins.

**Steps to solve similar questions:**

1. Check if the tag generation is split into independent parts (concatenation without a shared random nonce or overlapping data).
    
2. If the parts are independent, ask: "Can I query two different messages, chop their tags in half, and paste them together?"
    
3. Construct the exact forgery payload to prove the mix-and-match attack works.
    

### Question 4.7(a)

**Question:** Let $F$ be a pseudorandom function. Show that the following MAC is insecure, even for fixed-length messages:

$t = F_k(m_1) \oplus ... \oplus F_k(m_\ell)$

**Answer:** This MAC is insecure.

**Detailed Explanation:**

The vulnerability lies in the mathematical properties of the XOR ($\oplus$) operation. XOR is strictly **commutative**, meaning the order of operations does not matter ($A \oplus B = B \oplus A$).

Because the tag generation ignores the order of the blocks, an attacker can simply rearrange the blocks of a known message to forge a tag for a new message.

1. The attacker chooses a 2-block message: $M = m_1 || m_2$ (ensuring $m_1 \neq m_2$).
    
2. The attacker queries $M$ to the oracle and receives the tag $t = F_k(m_1) \oplus F_k(m_2)$.
    
3. The attacker constructs a forged message by swapping the blocks: $M^* = m_2 || m_1$.
    
4. The attacker outputs the exact same tag $t$ for $M^*$.
    

When the verifier checks $M^*$, they will compute $F_k(m_2) \oplus F_k(m_1)$. Because of commutativity, this perfectly equals $F_k(m_1) \oplus F_k(m_2) = t$. The forgery is successful without needing to calculate anything new.

**Steps to solve similar questions:**

1. Look at how the block outputs are combined.
    
2. If the combining operation is commutative (like XOR or regular addition without position-dependent padding), test a reordering attack.
    
3. Show that swapping two blocks produces a mathematically identical tag.
    

### Question 4.7(b)

**Question:** Let $F$ be a pseudorandom function. Show that the following MAC is insecure, even for fixed-length messages:

$t = F_k(\langle 1 \rangle || m_1) \oplus ... \oplus F_k(\langle \ell \rangle || m_\ell)$

where $\langle i \rangle$ is the $n/2$-bit encoding of the integer $i$.

**Answer:** This MAC is insecure.

**Detailed Explanation:**

The designer added the block index $\langle i \rangle$ to prevent the simple block-swapping attack from part (a). However, XOR is a **linear** operation ($x \oplus x = 0$). An attacker can exploit this cancellation property by querying multiple carefully chosen messages and XORing their resulting tags together to forge a new tag.

1. The attacker chooses four distinct blocks: $m_1, m'_1, m_2, m'_2$.
    
2. The attacker queries three specific 2-block messages:
    
    - $M_A = m_1 || m_2 \implies t_A = F_k(1||m_1) \oplus F_k(2||m_2)$
        
    - $M_B = m_1 || m'_2 \implies t_B = F_k(1||m_1) \oplus F_k(2||m'_2)$
        
    - $M_C = m'_1 || m_2 \implies t_C = F_k(1||m'_1) \oplus F_k(2||m_2)$
        
3. The attacker wants to forge a tag for the unqueried message $M^* = m'_1 || m'_2$.
    
4. The attacker computes $t^* = t_A \oplus t_B \oplus t_C$.
    

Let's expand the math to see what happens:

$t^* = [F_k(1||m_1) \oplus F_k(2||m_2)] \oplus [F_k(1||m_1) \oplus F_k(2||m'_2)] \oplus [F_k(1||m'_1) \oplus F_k(2||m_2)]$

Because $X \oplus X = 0$, the identical terms cancel out:

- $F_k(1||m_1) \oplus F_k(1||m_1) = 0$
    
- $F_k(2||m_2) \oplus F_k(2||m_2) = 0$
    

We are left entirely with:

$t^* = F_k(2||m'_2) \oplus F_k(1||m'_1) = F_k(1||m'_1) \oplus F_k(2||m'_2)$

This is exactly the valid tag for $M^*$. The attacker successfully forged it.

**Steps to solve similar questions:**

1. If the tags are combined using XOR, look for linear combinations that allow cancellation.
    
2. Query 3 messages that share common blocks.
    
3. XOR their tags together mathematically and cross out the duplicates.
    
4. If the remaining terms represent a valid tag for a 4th, unqueried message, the scheme is broken.
    

### Question 4.7(c)

**Question:** Let $F$ be a pseudorandom function. Show that the following MAC is insecure, even for fixed-length messages:

choose random $r$ from $\{0,1\}^n$

$t = F_k(r) \oplus F_k(\langle 1 \rangle || m_1) \oplus ... \oplus F_k(\langle \ell \rangle || m_\ell)$

and let the tag be $\langle r, t \rangle$.

**Answer:** This MAC is insecure.

**Detailed Explanation:**

In a secure randomized MAC, the random value $r$ must be generated honestly by the sender. However, during verification, the verifier just blindly reads the $r$ provided by the attacker in the tag pair $\langle r, t \rangle$. This means an attacker can manipulate $r$ to intentionally cause a catastrophic mathematical collision.

Let's look at the easiest case: a 1-block message.

The valid tag formula is: $t = F_k(r) \oplus F_k(\langle 1 \rangle || m_1)$.

1. The attacker wants to forge a tag for a message $m_1$ _without ever querying the oracle_.
    
2. The attacker maliciously chooses the "random" value to be exactly equal to the block data: $r = \langle 1 \rangle || m_1$.
    
3. Substitute this chosen $r$ into the tag formula:
    
    $t = F_k(\langle 1 \rangle || m_1) \oplus F_k(\langle 1 \rangle || m_1)$
    
4. Because any value XORed with itself is 0, the equation collapses: $t = 0^n$ (a string of all zeros).
    
5. The attacker outputs the tag pair $\langle (\langle 1 \rangle || m_1), 0^n \rangle$.
    

When the verifier checks this, they will calculate $F_k(r) \oplus F_k(\langle 1 \rangle || m_1)$, which will evaluate to $0^n$, perfectly matching the $t$ provided by the attacker. This is an existential forgery requiring **zero** queries to the oracle.

**Steps to solve similar questions:**

1. Determine if the adversary has control over any inputs that feed into the PRF during verification (like an attached nonce or random $r$).
    
2. Check if the adversary can set that controllable input equal to another part of the equation.
    
3. If it uses XOR, forcing two inputs to be identical will cancel them out to a predictable value (like $0^n$).
    

### Question 4.13(a)

**Question:** Say the sender and receiver do not agree on the message length in advance, and the sender only authenticates messages of length 2n. Show that an adversary can forge a valid tag on a message of length 4n.

**Answer:** Basic CBC-MAC is insecure for variable-length messages.

**Detailed Explanation:**

CBC-MAC processes blocks in a chain, where the output of one block becomes the XOR input for the next. This creates a "Length Extension Attack" vulnerability.

1. The attacker queries a 2-block message $M = m_1 || m_2$ and receives the valid tag $t$.
    
    _(Note: The internal state after block 1 is $C_1 = F_k(m_1)$, and the tag is $t = F_k(C_1 \oplus m_2)$)._
    
2. The attacker wants to forge a 4-block message. They construct:
    
    $M^* = m_1 || m_2 || (m_1 \oplus t) || m_2$
    
3. Let's trace how the verifier calculates CBC-MAC on $M^*$:
    
    - **Block 1:** $C_1 = F_k(m_1)$
        
    - **Block 2:** $C_2 = F_k(C_1 \oplus m_2) = t$
        
    - **Block 3 (The trick):** The verifier XORs the state ($C_2 = t$) with the next block ($m_1 \oplus t$).
        
        $C_3 = F_k(t \oplus (m_1 \oplus t))$. The $t$'s cancel out! $C_3 = F_k(m_1) = C_1$.
        
    - **Block 4:** The state is exactly what it was after Block 1. So, $C_4 = F_k(C_3 \oplus m_2) = F_k(C_1 \oplus m_2) = t$.
        
4. The final calculated tag for the 4-block message is exactly $t$. The attacker outputs $t$ as the forged tag.
    

**Steps to solve similar questions:**

1. Write down the internal state chaining rule for CBC-MAC: $C_i = F_k(C_{i-1} \oplus m_i)$.
    
2. Use a previously obtained tag $t$ to craft a block that cancels out the current chain state (e.g., $m_{new} = m_1 \oplus t$).
    
3. This resets the internal state, allowing you to append old blocks and predictably arrive at the same final tag.
    

### Question 4.13(b)

**Question:** Say the receiver only accepts 3-block messages, but the sender authenticates messages of any length that is a multiple of n. Show that an adversary can forge a valid tag on a new message.

**Answer:** This construction is insecure.

**Detailed Explanation:**

Here, the attacker exploits the oracle's willingness to sign messages of _any_ length to simulate the internal chaining states of a 3-block message, step by step.

1. The attacker wants to forge a tag for the 3-block message $M^* = m_1 || m_2 || m_3$. They will do this by querying three separate 1-block messages.
    
2. **Query 1:** Ask the oracle to sign the 1-block message $m_1$. Receive tag $t_1 = F_k(m_1)$.
    
3. **Query 2:** Calculate a new 1-block message by XORing the first tag with the desired second block: $(t_1 \oplus m_2)$. Ask the oracle to sign it. Receive tag $t_2 = F_k(t_1 \oplus m_2)$.
    
4. **Query 3:** Calculate another 1-block message: $(t_2 \oplus m_3)$. Ask the oracle to sign it. Receive tag $t_3 = F_k(t_2 \oplus m_3)$.
    
5. The attacker outputs $t_3$ as the valid tag for the full 3-block message $M^*$.
    

Let's prove the verifier will accept this. The verifier runs CBC-MAC on $m_1 || m_2 || m_3$:

- State 1: $C_1 = F_k(m_1)$. (This equals $t_1$).
    
- State 2: $C_2 = F_k(C_1 \oplus m_2) = F_k(t_1 \oplus m_2)$. (This equals $t_2$).
    
- State 3: $C_3 = F_k(C_2 \oplus m_3) = F_k(t_2 \oplus m_3)$. (This equals $t_3$).
    

The final tag is mathematically guaranteed to be $t_3$. Because the attacker only ever asked the oracle to sign 1-block messages, they successfully executed an existential forgery on a 3-block message.

**Steps to solve similar questions:**

1. Break down the target MAC algorithm into its step-by-step internal states.
    
2. Ask the oracle for the tag of step 1.
    
3. Use that tag to manually compute the required input for step 2, and query _that_ as a new, independent message.
    
4. Repeat until you reach the final state, using the last returned tag as your forgery.
    

### Question 5.3

**Question:** Let (Gen, H) be a collision-resistant hash function. Is (Gen, $\hat{H}$) defined by $\hat{H}(x) = H(H(x))$ necessarily collision resistant?

**Answer:** Yes. Composing a collision-resistant hash function with itself maintains collision resistance.

**Detailed Explanation:**

In cryptography, we prove this using a "Proof by Contradiction" (or a Reduction). We assume the new function $\hat{H}$ is broken, and prove that this implies the original function $H$ must also be broken. Since $H$ is defined as unbreakable, our assumption must be false.

**Proof:**

Assume there exists an adversary that can find a collision in $\hat{H}$. This means they can find two distinct inputs $x \neq x'$ such that:

$H(H(x)) = H(H(x'))$

Let's analyze the internal states. Let $y = H(x)$ and $y' = H(x')$.

This leaves us with exactly two possible cases:

- **Case 1: $y = y'$**
    
    This means $H(x) = H(x')$. Since we already established that $x \neq x'$, the pair $(x, x')$ constitutes a direct collision for the original hash function $H$.
    
- **Case 2: $y \neq y'$**
    
    If the internal states are different, but the final output is the same, this means $H(y) = H(y')$. Since $y \neq y'$, the pair $(y, y')$ constitutes a valid collision for the original hash function $H$.
    

In _both_ possible scenarios, finding a collision in $\hat{H}$ immediately yields a collision in $H$. Because $H$ is strictly defined as collision-resistant (meaning it is computationally infeasible to find a collision in it), it must also be computationally infeasible to find a collision in $\hat{H}$.

**Steps to solve similar questions:**

1. Start the proof by assuming the new construction is broken (i.e., a collision exists: $f(x) = f(x')$ where $x \neq x'$).
    
2. Substitute the definition of the new function into the equation.
    
3. Break the logic down into cases based on whether the inner functions evaluate to the same value or different values.
    
4. Show that in every single case, the equality yields a direct collision for the underlying cryptographic primitive, completing the reduction.