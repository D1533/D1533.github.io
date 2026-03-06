---
layout: default
title: Sum-O-Primes
---
---
## PicoCTF - Sum-O-Primes
---

## Description
We have so much faith in RSA we give you not just the product of the primes, but their sum as well!

---
## Solution

The challenge provides us with the following python script
```python
#!/usr/bin/python

from binascii import hexlify
from gmpy2 import mpz_urandomb, next_prime, random_state
import math
import os
import sys

if sys.version_info < (3, 9):
    import gmpy2
    math.gcd = gmpy2.gcd
    math.lcm = gmpy2.lcm

FLAG  = open('flag.txt').read().strip()
FLAG  = int(hexlify(FLAG.encode()), 16)
SEED  = int(hexlify(os.urandom(32)).decode(), 16)
STATE = random_state(SEED)

def get_prime(bits):
    return next_prime(mpz_urandomb(STATE, bits) | (1 << (bits - 1)))

p = get_prime(1024)
q = get_prime(1024)

x = p + q
n = p * q

e = 65537

m = math.lcm(p - 1, q - 1)
d = pow(e, -1, m)

c = pow(FLAG, e, n)

print(f'x = {x:x}')
print(f'n = {n:x}')
print(f'c = {c:x}')

```
We are given $$n,c$$ and $$x$$ so do not get confused by the weird notation of the challenge, $$x = p + q$$ is known.

Since $$n = pq$$ and $$x = p + q $$, we have that
{% raw %}
$$ p = x - q = x - \frac{n}{p},$$
{% endraw %}
Multypling both sides by $$p$$ and rearranging, we obtain 
{% raw %}
$$ p^2 - xp + n = 0,$$
{% endraw %}
which can be solved easily for $$p$$ using the quadratic formula
{% raw %}
$$ p = \frac{x \pm \sqrt{x^2 - 4n}}{2}.$$
{% endraw %}
In this case, both roots work, so you can use whichever you prefer.

Once we have $$p$$, we can compute $$q = n/p$$, recover the private key $$d = e^{-1} \ mod \ \varphi(n) $$ where $$\varphi(n) = (p-1)(q-1)$$ and decrypt the flag as $$m = c^d \ mod \ n$$.

```python
from gmpy2 import iroot
from Crypto.Util.number import long_to_bytes

x = 0x152a1447b61d023bebab7b1f8bc9d934c2d4b0c8ef7e211dbbcf841136d030e3c829f222cec318f6f624eb529b54bcda848f65574896d70cd6c3460d0c9064cd66e826578c2035ab63da67d069fa302227a9012422d2402f8f0d4495ef66104ebd774f341aa62f493184301debf910ab3d1e72e357a99c460370254f3dfccd9ae
n = 0x6fc1b2be753e8f480c8b7576f77d3063906a6a024fe954d7fd01545e8f5b6becc24d70e9a5bc034a4c00e61f8a6176feb7d35fe39c8c03617ea4552840d93aa09469716913b58df677c785cd7633d1b7d31e2222cab53be235aa412ac5c5b07b500cf3fd5d6b91e2ddc51bff1e6eec2cb68723af668df36e10e332a9cbb7f3e2df9593fa0e553ed58afec2aa3bc4ae8ef1140e4779f61bdeae4c0b46136294cf151622e83c3d71b97c815b542208baa28207225f134c5a4feac998aeb178a5552f08643717819c10e8b5ec7715696c3bf4434fbea8e8a516dfd90046a999e24a0fb10d27291eb29ef3f285149c20189e7d0190417991094948180196543b8c91
c = 0x16acf84a73cefd321ed491a15c640a495b09050cdce435ec37442faf9a694775e1ebffb6dbad6133cbc54e3f641506b0613f711625594fcb467f915f2708714b4c9936f5f4752c3299157cff4eb68eb82c0054dae351314829974f4feadaf126cda92b97e348dbef2640ec3a729a064e615df73d644700f93bf87579683e253d29622525bea3644f59aac8e0b2553bfea48d99e9b323e03cbf55166659974eb8c51cc7b2c2c5d6aa6c01b056a8ed7283d96656a3496f266726605af1be139d586f208d4d7c59c2771dc8036d490d3672ee4513301002775d7c39eac421c6cb4f01344e061549a4cb11c057accef1726572e447501004c772ec91b4a55811280f
e = 65537


delta = iroot(x**2-4*n,2)[0]
p = (x + delta) // 2

q = n//p
phi = (p-1)*(q-1)
d = pow(e,-1,phi)

flag = long_to_bytes(pow(c,d,n)).decode()
print(flag)
```