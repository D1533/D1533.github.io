---
layout: default
title: RSA - Common Modulus
---

---
## Rootme - RSA - Common Modulus
---
## Description
A message was sent by a company to two of its engineers, but they were negligent in the choice of the keys of its employees. Decrypt the sent message.

---
## Solution

We are given the RSA encryption of the same message $$m$$ using the same modulus but different exponents. That is,
{% raw %}
$$c_1 \equiv m^{e_1} \ mod \ n, \quad c_2 \equiv m^{e_2} \ mod \ n.$$
{% endraw %} 

We can see that in this challenge, $$gcd(e_1, e_2) = 1$$. Therefore, there exist $$a,b \in \mathbb{Z}$$ such that 
{% raw %}
$$ae_1 + be_2 = 1,$$
{% endraw %} 
and thus

{% raw %}
$$m \equiv m^{ae_1 + be_2} \equiv c_1^ac_2^b \ mod \ n.$$
{% endraw %} 

*Python script*
```python
from Crypto.PublicKey import RSA
from Crypto.Util.number import long_to_bytes
import base64
from sympy import gcdex


msg1 = 'BzFd4riBUZdFuPCkB3LOh+5iyMImeQ/saFLVD+ca2L8VKSz0+wtTaL55RRpHBAQdl24Fb3XyVg2N9UDcx3slT+vZs7tr03W7oJZxVp3M0ihoCwer3xZNieem8WZQvQvyNP5s5gMT+K6pjB9hDFWWmHzsn7eOYxRJZTIDgxA4k2w='
msg2 = 'jmVRiKyVPy1CHiYLl8fvpsDAhz8rDa/Ug87ZUXZ//rMBKfcJ5MqZnQbyTJZwSNASnQfgel3J/xJsjlnf8LoChzhgT28qSppjMfWtQvR6mar1GA0Ya1VRHkhggX1RUFA4uzL56X5voi0wZEpJITUXubbujDXHjlAfdLC7BvL/5+w='

with open("key1_pub.pem", "rb") as f:
    key1 = f.read()

with open("key2_pub.pem", "rb") as f:
    key2 = f.read()


key1 = RSA.import_key(key1)
key2 = RSA.import_key(key2)

e1 = key1.e
e2 = key2.e
n = key1.n

c1 = int.from_bytes(base64.b64decode(msg1),byteorder='big')
c2 = int.from_bytes(base64.b64decode(msg2),byteorder='big')

a,b, gcd = gcdex(e1, e2)

m = (pow(c1,int(a),n)*pow(c2,int(b),n) ) % n
print(long_to_bytes(m).decode())
```