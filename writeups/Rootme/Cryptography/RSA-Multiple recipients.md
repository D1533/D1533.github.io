---
layout: default
title: RSA - Multiple recipients
---

---
## Rootme - RSA - Multiple recipients
---
## Description


A message sent to 3 different people has been intercepted. We have the public keys of those people. Find the message to validate the challenge.

---
## Solution

We are given three RSA-encrypted ciphertexts of the same plaintext message $$m$$, encrypted using the same public exponent $$e$$, but different pairwise coprime moduli $$n_0,n_1,n_2$$. That is,
{% raw %}
$$ c_0 \equiv m^e \ mod \ n_0, \quad c_1 \equiv m^e \ mod \ n_1, \quad c_2 \equiv m^e \ mod \ n_2.$$
{% endraw %}

Since the $$n_i$$ are pairwise coprime, by the Chinese Remainder Theorem we can compute a unique integer $$x \ mod \ n_0n_1n_2$$ such that
{% raw %}
$$x \equiv c_0 \ mod \ n_0, \quad x \equiv c_1 \ mod \ n_1, \quad x \equiv c_2 \ mod \ n_2.$$ 
{% endraw %}
This implies that 
{% raw %}
$$x \equiv m^e \ mod \ n_0n_1n_2.$$ 
{% endraw %}
In this challenge, $$e=3$$ is sufficiently small, so we might have $$m^3 < n_0n_1n_2$$. Therefore we can take the cubic root of $$x$$ and retrieve the flag.


*Python script*
```python
from Crypto.PublicKey import RSA
from Crypto.Util.number import long_to_bytes
from sympy.ntheory.modular import crt
from gmpy2 import iroot
import base64


msg0 = 'vjXJvWis95tc25G+wxC5agClCJFB9vUslFyV+I4bSiwS4Sm6k8eF61EizKo4hZFwROlO3Ci3YQaTrAm+Y9/qEbM7asvwKTePKX+cLVN61l0xxfTL8CdoXkRE2rSczp1AzzmFz83OHgszX/Wf7kgWU4M73+efPvU9FmcWOauakrdJZx8B0ErJ5cYWNS0ZCam0Nlz+pISqdSJ6MSz0Ek62Ulb3ei8I41FOdHtd8mhC7dfpdfmVLOSEW4yEnn5iuMW0ydvW055dodLc9RKcvJafH9e3zf8/S1/RORXZoUHWnBXBEFOl8iVXz70GcDTPzIhxh6imi0ynLbV68qW2vw2XRQ=='
msg1 = 'Mi4MnobVabt1+Q7R/0aIYBBpeRxPRuR6gkhr/Wbw/D23ywu2KbUYBab2XbEguRz8yzBlxScbFjjb98DuILxURoFN8lNKVJLS3d/IrGGr0hjocbz27uBS97hseX0S4nd+BL6LUn0o7qe/yCqk7Y0tEhhcuSMrGn/l9N/6UjgN38TyoyvVbd1UH6BHGLdj9g5JRBKAvcGumymfiE0qS7IOnM3mN5W8gRJaEGDkhDim21Pm1Yg2GRBJ+z7C8AEy+Dz6OFvWuDsY24Gb+643D7KrmFObJ1n8Qyme/Y1bfvBkG+xdvGoBzyQrlDT12Qjkfoqb37HNrGUUD5cj2q54gyp80w=='
msg2 = 'hMQVVspFGh7NxCqLf7DVto3OxgDLn9n9gOCjOjEOYhi/VwZ3adFsbBL+zLZxYNdkKLNWCNRktRwpnWriEsW1uDnVt2LbxSLvjvRKbR/hyvpY+0UUZFS6wCWQjGyxUydDxQ88jNM5dY58/1nxsd04I3n3Mt97SuqwBN1+4VS3SsqtbR0GU1C7ODkPoCeGVd3PNkGHPgbT7QzMwxl63Pl3i/sp0I2/gqSnKu5CDS7e2WELz0hfiOJ2v2RvIon2EEbPwx1/6zxZlMhHuGXHNZKDtyqe6Dd+EIjpwhQFW3eH7fDIirRbaPPXAYsoypS5eFD3mIWUs4yVOH9ykkdKQ9FNwg=='

with open("clef0_pub.pem", "rb") as f:
    key0 = f.read()
with open("clef1_pub.pem", "rb") as f:
    key1 = f.read()
with open("clef2_pub.pem", "rb") as f:
    key2 = f.read()

key0 = RSA.import_key(key0)
key1 = RSA.import_key(key1)
key2 = RSA.import_key(key2)

e = 3
n0 = key0.n
n1 = key1.n
n2 = key2.n

c0 = int.from_bytes(base64.b64decode(msg0),byteorder='big')
c1 = int.from_bytes(base64.b64decode(msg1),byteorder='big')
c2 = int.from_bytes(base64.b64decode(msg2),byteorder='big')

x, mod = crt([n0,n1,n2],[c0,c1,c2])
flag = iroot(x,3)[0]

print(long_to_bytes(flag).decode())

```