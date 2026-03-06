---
layout: default
title: b00tl3gRSA3
---
---
## PicoCTF - b00tl3gRSA3
---
## Description
Why use p and q when I can use more? Connect with nc jupiter.challenges.picoctf.org 3726.

---
## Solution
The description of the challenge tells us that $$n$$ is built as product of multiple primes. The server gives us the numbers $$c,n$$ and $$e$$. 
We can then just try to factorize $$n$$, so the totient function of $$n = p_1p_2\cdots p_k$$ can be computed as
{% raw %}
$$\varphi(n) = \prod_{j=1}^k (p_j-1)$$
{% endraw %}

We then compute $$d = e^{-1}\ mod \ \varphi(n) $$ and decrypt the flag as $$m = c^d \ mod \ n$$.

*SageMath script*
```python
import os
os.environ["TERM"] = "xterm"
from pwn import *
import re 
from Crypto.Util.number import long_to_bytes

r = remote("jupiter.challenges.picoctf.org", 3726)
data = r.recv().decode()

c = int(re.search(r"c: (\d+)", data).group(1))
n = int(re.search(r"n: (\d+)", data).group(1))
e = int(re.search(r"e: (\d+)", data).group(1))

factors_n = factor(n)

phi = 1
for p in factors_n:
    phi *= (p[0]-1)

d = int(pow(e,-1,phi))
flag = long_to_bytes(pow(c,d,n)).decode()
print(flag)
```