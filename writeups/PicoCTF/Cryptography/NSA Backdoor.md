---
layout: default
title: NSA Backdoor
---
---
## PicoCTF - NSA Backdoor
---

## Description
I heard someone has been sneakily installing backdoors in open-source implementations of Diffie-Hellman... I wonder who it could be... ;)

---
## Solution

We can solve this challenge using the Pollard's $$p-1$$ factorization and the Pohlig Hellman algorithm. First, let's introduce the math behind these concepts.

### Pollard's $$p-1$$ factorization
Pollard's p-1 factoring method leverages the fact that if $$p$$ is a prime divisor of $$n$$ and $$p-1$$ is $$B$$-smooth, then for some suitably chosen $$M$$ such that $$M \mid (p-1)$$, i.e , $$M = k(p-1)$$ for some $$ k \in \mathbb{Z}$$, Fermat's Little Theorem implies
{% raw %}
$$a ^ M \equiv a^{k(p-1)} \equiv (a^{p-1})^k \equiv 1 \ mod \ p,$$
{% endraw %}
and therefore $$p \mid (a^M - 1)$$. 

Since $$p$$ also divides $$n$$, it follows that
{% raw %}
$$gcd(a^M-1,n)$$
{% endraw %}
might return a non-trivial factor of $$n$$.

There are efficient ways to find such $$M$$, but for this challenge, brute forcing it is sufficient.

Once $$p$$ is found, we can recover $$q = n/p$$.

<br>
### Pohlig Hellman Algorithm 
Suppose we want to solve the discrete logarithm problem
{% raw %}
$$h \equiv g^x \ mod \ p,$$
{% endraw %}
where $$p-1$$ is $$B$$-smooth, that is, we have the prime factorization $$p-1 = q_1^{e_1}\cdots q_k^{e_k}$$, with $$q_i \leq B$$ for some $$B > 0$$.

Let
{% raw %}
$$
\begin{align*}
g_i &\equiv g^{(p-1)/q_i^{e_i}} \ mod \ p, \\
h_i &\equiv h^{(p-1)/q_i^{e_i}} \ mod \ p. 
\end{align*}
$$
{% endraw %}
If we solve the discrete logaritm problems
{% raw %}
$$
\begin{align*}
h_1 &\equiv g_1^{x_1} \ mod \ p, \\
h_2 &\equiv g_2^{x_2} \ mod \ p, \\
 & \ \ \vdots \\
h_k &\equiv g_k^{x_k} \ mod \ p,
\end{align*}
$$
{% endraw %}
then $$ x \equiv x_i \ mod \ q_i^{e_i}$$ for all $$i = 1, \cdots, k $$ and we can use the Chinese Remainder Theorem to recover $$x \ mod \ p - 1$$. That's because

{% raw %}
$$h \equiv g^x \ mod \ p \implies h ^{(p-1)/q_i^{e_i}} \equiv g^{x(p-1)/q_i^{e_i}} \ mod \ p,$$
{% endraw %}
therefore $$h_i \equiv g_i^x \ mod \ p $$, and since $$ord(g_i) = q_i^{e_i}$$, then $$x \equiv x_i \ mod \ q_i^{e_i}$$.

<br>
### Challenge application
In our challenge, we have to solve 
{% raw %}
$$c \equiv 3^x \ mod \ n,$$
{% endraw %}
where $$n = pq$$. 

Once we know the factorization of $$n$$ using Pollard's $$p-1$$, we can use Pohlig Hellman algorithm for $$p$$ and $$q $$. The only subtlety here is that $$gcd(p-1,q-1) = 2$$. Therefore, we can apply Pohlig Hellman to $$(p-1)/2$$ and $$q$$, finding $$x_p \ mod \ (p-1)/2 $$ and $$x_q \ mod \ q-1$$, and recovering 
$$x$$ applying again the Chinese Remainder Theorem.

Finally, after decoding $$x$$, one can find that the flag is not recovered, that is because even though we have found a $$x$$ such that $$c \equiv 3^x \ mod  \ n $$, we also have that $$c \equiv 3^{x \pm kord(3)} \ mod \ p$$. After some tries, one can find that $$x - \dfrac{(p-1)(q-1)}{4}$$ gives the correct solution.

*SageMath script*
```python
from math import gcd
from Crypto.Util.number import long_to_bytes

n = 0x98a3425eee4016a2592706867127e6c52ab2cf8077806f5626095e3afadc73cb4d0e747c5b9bf6234242e9578b12aba5e391e04a5cd2730f6e45d9f0758fb69eb32e0070b9efd3470f6571a8443bae63cd16efcb3e945dc3da1ce46993be4c8b4467ffb4e0525428bb8673ba144b0d36d1c34fe87307d68439070da27a8809551aa6cdf55c39c79bb7b6b7b9c26b45ef79f6c1ebf68033e4beab2d24df66f69dfb7f54d70d3b477fc7b67592cb029dfe6341c591c34a127f84b33626cd117707b69d1ed55f1773e3ba8d26b76f2db95e85de14a6aa1ff3de7fa23ce9f7ebd0e6c18c2fef4bbff47b6bd632d2d767aab7d35bf4d8577e50556626096704f0c425
c = 0x8788542cefd7490c9282c06b8d24280d56c6706b996bdf580290cdf2cb90e45efd2ce185fc07d2b916c24b0512d38ca14de0ee608a9d6003f258859bbbed97dad15c1d07410a34fd55cd8305eb43418d38f1ca6e024725b97fd9da701a39c23fe55a13d43b4bf9a3d9ebb44d7fe67bd60beffc29ec27bb4baf05ec5b250bfa68360df0d1379c066297a7878e59d27e68cf6a0da90755450827623e54e4f3d9f280fef53c7620d58decfbd10dd64e9d1d5507b5460603c58f5be70c82e2a8e613d730a950caea4c4389c5fc0521f8207ead5fb26c04eb6d0486fd6fe8d015fdabbda00139b42163acc86ffb30c12988058c6247344c42b8f3cdc984c06f4276f8


def pollard(n):
    a = 2
    j = 2
    while True: 
        a = pow(a,j,n)
        g = gcd(a-1,n)
        if 1 < g and g < n:
            return g
        j += 1

def pohlig_hellman(g,h,p, factors):
    remainders = []
    moduli = []
    
    for q, e in factors:
        qe = q^e
        
        g_q = pow(g,(p-1) // qe,p)
        h_q = pow(h, (p-1) // qe, p)

        x_q = discrete_log(h_q, g_q, ord=qe)

        remainders.append(x_q)
        moduli.append(qe)

    x = crt(remainders, moduli)
    return x


p = pollard(n)
q = n//p


x_p = pohlig_hellman(3,c,p,list(factor(p-1))[1:])
x_q = pohlig_hellman(3,c,q,list(factor(q-1)))

x = crt([x_p,x_q],[(p-1)//2,q-1])

x -= (p-1)*(q-1)//4
flag = long_to_bytes(x).decode()
print(flag)

```