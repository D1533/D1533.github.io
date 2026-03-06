---
layout: default
title: No Padding, No Problem
---
---
## PicoCTF - No Padding, No Problem
---

## Description

Oracles can be your best friend, they will decrypt anything, except the flag's ciphertext. How will you break it? Connect with nc mercury.picoctf.net 10333.

---
## Solution

Suppose we have an oracle that, given a ciphertext $$c \in \mathbb{Z}_N$$, returns the plaintext
{% raw %}
$$ m = c^d \ mod \ N.$$
{% endraw %}
Naturally, the oracle does not accept the encrypted flag for decryption. 

At the begining of the connection, the server sends us the public key $$(N, e) $$ and the ciphertext $$c$$. If we choose some $$k \in \mathbb{Z}_N^x$$, compute $$c_k = k^e \ mod \ N$$, and send the oracle the product $$c_kc$$ to decrypt, the oracle will return some $$M$$ computed as
{% raw %}
$$ M \equiv (c_kc)^d \ mod \ N.$$
{% endraw %}
Since $$c_k^d \equiv k \ mod \ N$$ and $$c^d \equiv m \ mod \ N$$, by the propeties of modular arithmetic, we have
{% raw %}
$$ (c_kc)^d \equiv c_k^dc^d \equiv km \ mod \ N,$$
{% endraw %}
so $$M \equiv km \ mod \ N$$ and therefore we can recover $$m$$ as
{% raw %}
$$m \equiv Mk^{-1} \ mod \ N.$$ 
{% endraw %}

```python
from pwn import *
from Crypto.Util.number import long_to_bytes

r = remote("mercury.picoctf.net", 10333)
r.recvuntil(b'n: ')
n = int(r.recvline().decode())
r.recvuntil(b'e: ')
e = int(r.recvline().decode())
r.recvuntil(b'ciphertext: ')
ciphertext = int(r.recvline().decode())

r.recvuntil('Give me ciphertext to decrypt: ')

payload = (pow(2,e,n)*ciphertext) % n

r.sendline(str(payload).encode())
r.recvuntil('Here you go: ')
M = int(r.recv().decode())
flag = (pow(2,-1,n)*M) % n
print(long_to_bytes(flag).decode())
```