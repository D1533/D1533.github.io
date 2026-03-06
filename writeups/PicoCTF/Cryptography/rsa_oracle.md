---
layout: default
title: rsa_oracle
---
---
## PicoCTF - rsa_oracle
---

## Description

Can you abuse the oracle? An attacker was able to intercept communications between a bank and a fintech company. They managed to get the message (ciphertext) and the password that was used to encrypt the message. After some intensive reconassainance they found out that the bank has an oracle that was used to encrypt the password and can be found here nc titan.picoctf.net 59219. Decrypt the password and use it to decrypt the message. The oracle can decrypt anything except the password.

---
## Solution

Suppose we have an oracle that, given a message $$k \in \mathbb{Z}_N$$, returns the ciphertext
{% raw %}
$$ c_k = k^e \ mod \ N,$$
{% endraw %}
and, given the ciphertext $$c_k$$, returns the corresponding plaintext
{% raw %}
$$ k = c_k^d \ mod \ N.$$
{% endraw %}
Naturally, the oracle does not accept the encrypted password for decryption. 

Now, let $$c$$ be the ciphertext corresponding to the password $$m$$, so our objective is to recover $$m$$. If we send the oracle the product $$c_kc$$ to decrypt, the oracle will return some $$M$$ computed as
{% raw %}
$$ M \equiv (c_kc)^d \ mod \ N.$$
{% endraw %}
Since $$c_k^d \equiv k \ mod \ N$$ and $$c^d \equiv m \ mod \ N$$, by the propeties of modular arithmetic, we have
{% raw %}
$$ (c_kc)^d \equiv c_k^dc^d \equiv km \ mod \ N,$$
{% endraw %}
so $$M \equiv km \ mod \ N$$ and therefore
{% raw %}
$$m \equiv Mk^{-1} \ mod \ N.$$ 
{% endraw %}

Notice that we do not know $$N$$ so we cannot compute $$k^{-1} \ mod  \ N$$. However, if we choose $$k = 2$$, it is possible that $$km < N$$ and therefore $$m = M/2$$.

```python
from pwn import *

enc_password = 3567252736412634555920569398403787395170577668834666742330267390011828943495692402033350307843527370186546259265692029368644049938630024394169760506488003

r = remote("titan.picoctf.net", 59219)

r.sendline(b'E')
r.recv()
r.sendline(b'\x02')
r.recvuntil(b'mod n) ')

k = r.recvline().decode()
payload = int(k)*enc_password

r.recv()
r.sendline(b'D')
r.sendline(str(payload).encode())
r.recvuntil(b'mod n): ')
decr = r.recvline().decode()

password = int(decr,16) // 2

password = password.to_bytes(len(str(password)), "big").decode("utf-8").lstrip("\x00")

print(password)
```

Once we have recovered the password, we can decrypt the secret.enc using the following terminal command
```sh
openssl enc -aes-256-cbc -d -in secret.enc -pass pass:3319c
```
