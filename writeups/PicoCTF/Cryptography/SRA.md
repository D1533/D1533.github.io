---
layout: default
title: SRA
---
---
## PicoCTF - SRA
---

## Description
Description
I just recently learnt about the SRA public key cryptosystem... or wait, was it supposed to be RSA? Hmmm, I should probably check... Connect to the program on our server: nc saturn.picoctf.net 50736 Download the program: chal.py

---
## Solution
After deobfuscating the source code, we see that the challenge provides us the ciphertext $$c$$ (anger), the private key $$d$$ (envy), and the public exponent $$e$$. However, $$n$$ is not given. 

Notice that since $$ed \equiv 1 \ mod \ \varphi(n)$$, then
{% raw %}
$$ed -1 = k(p-1)(q-1)$$
{% endraw %} 
for some $$k \in \mathbb{Z}$$. \\
From this we can infer that $$p-1$$ and $$q-1$$ are both divisors of $$ed -1 $$, so our strategy is to find all divisors $$t$$ of $$ed - 1$$ such that $$t+1$$ is a prime of at least 128 bits. \\
Once we have all such primes, we can check for each pair $$p,q$$ the condition $$d \equiv e^{-1} \ mod \ n$$, with $$n=pq$$. If this condition is satisfied, we have correctly recovered $$p$$ and $$q$$, and thus $$n$$. Therefore we can recover the *pride* variable by computing $$c^d \ mod \ n $$ and send it to the server in order to recieve the flag.


```python
import sympy as sp
from Crypto.Util.number import isPrime, long_to_bytes
from pwn import *

e = 65537


r = remote("saturn.picoctf.net", 62609)
r.recvuntil(b'anger = ')
c = int(r.recvline())
r.recvuntil(b'envy = ')
d = int(r.recvline())

divisors = sp.divisors(e*d-1)

primes = [d+1 for d in divisors if isPrime(d+1)]
primes = [p for p in primes if p <= 2**128]

for i in range(len(primes)-1):
    for j in range(i+1,len(primes)):
        phi = (primes[i]-1)*(primes[j]-1)
        n = primes[i]*primes[j]
        if pow(e,-1,phi) == d:
            try:
                pride = long_to_bytes(pow(c,d,n)).decode()
                break
            except:
                pass


r.recv()
r.sendline(pride.encode())
print(r.recv().decode())

```