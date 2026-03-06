---
layout: default
title: RSA - Decipher Oracle
---

---
## Rootme - RSA - Decipher Oracle
---
## Description
You have sucessfully intercepted a C text, encrypted with a RSA keypair which you know the (n, e) public component.

Luckily, your victim has made a mistake by using a server that allows you to decrypt any ciphered text with his public key, except for those including a secret that must stay encrypted.

Your job is to handle it in order to decrypt this weird message!
<div style="overflow-x: auto; white-space: pre; border: 1px solid #ccc; padding: 10px;">
n = 456378902858290907415273676326459758501863587455889046415299414290812776158851091008643992243505529957417209835882169153356466939122622249355759661863573516345589069208441886191855002128064647429111920432377907516007825359999
e = 65537
c = 41662410494900335978865720133929900027297481493143223026704112339997247425350599249812554512606167456298217619549359408254657263874918458518753744624966096201608819511858664268685529336163181156329400702800322067190861310616
</div>

---
## Solution

We can decrypt any message we want, except for the flag. Let $$m$$ be the flag which has been encrypted with
{% raw %}
$$c \equiv m^e \ mod \ n.$$
{% endraw %}

Let $$k > 0$$. If we send to the oracle $$k^ec$$, then we recieve $$(k^ec)^d \ mod \ n$$. However, notice that
{% raw %}
$$(k^ec)^d \equiv kc^d \equiv km \ mod \ n.$$
{% endraw %}
Therefore, we can multiply the message received from the oracle by the modular inverse of $$k$$ modulo $$n$$ and retrieve the flag
{% raw %}
$$k^{-1} (k^ec)^d \equiv k^{-1}km \equiv m \ mod \ n.$$
{% endraw %}
*Python script*
```python
from pwn import *
from Crypto.Util.number import long_to_bytes

n = 456378902858290907415273676326459758501863587455889046415299414290812776158851091008643992243505529957417209835882169153356466939122622249355759661863573516345589069208441886191855002128064647429111920432377907516007825359999
e = 65537
c = 41662410494900335978865720133929900027297481493143223026704112339997247425350599249812554512606167456298217619549359408254657263874918458518753744624966096201608819511858664268685529336163181156329400702800322067190861310616


r = remote('challenge01.root-me.org',51031)

k = 2
payload = pow(k,e,n)*c % n

r.recv()
r.sendline(str(payload).encode())
r.recvuntil(b'The corresponding plaintext is: ')
c_k = int(r.recvline().decode())

flag = pow(k,-1,n)*c_k % n
print(long_to_bytes(flag).decode())

```