---
layout: default
title: Discrete logarithm problem
---

---
## Rootme - Discrete logarithm problem
---
## Description

Alice and Bob use the Discrete Logarithm problem in order to find securely a common session key.
You have found Alice’s public key (g, y, p), Now you have to find the private one x as :
y = g^x mod p

For information, p is a primer number of 522bits length and g is a primitive root modulo p.

For validating this challenge, you just have to input the value of x (also 522 bits wide).

<div style="overflow-x: auto; white-space: nowrap; border: 1px solid #ccc; padding: 10px;">
<strong>p =</strong> 7863166752583943287208453249445887802885958578827520225154826621191353388988908983484279021978114049838254701703424499688950361788140197906625796305008451719
</div>

<div style="overflow-x: auto; white-space: nowrap; border: 1px solid #ccc; padding: 10px;">
<strong>g =</strong> 2862392356922936880157505726961027620297475166595443090826668842052108260396755078180089295033677131286733784955854335672518017968622162153227778875458650593
</div>
<div style="overflow-x: auto; white-space: nowrap; border: 1px solid #ccc; padding: 10px;">
<strong>y =</strong> 6289736695712027841545587266292164172813699099085672937550442102159309081155467550411414088175729823598108452032137447608687929628597035278365152781494883808
</div>
<br>

---
## Solution

We can solve this challenge using the Pohlig Hellman Algorithm.

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

*Python script*
```python
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



p = 7863166752583943287208453249445887802885958578827520225154826621191353388988908983484279021978114049838254701703424499688950361788140197906625796305008451719
y = 6289736695712027841545587266292164172813699099085672937550442102159309081155467550411414088175729823598108452032137447608687929628597035278365152781494883808
g = 2862392356922936880157505726961027620297475166595443090826668842052108260396755078180089295033677131286733784955854335672518017968622162153227778875458650593

factors = list(factor(p-1))
print(pohlig_hellman(g,y,p,factors))
```