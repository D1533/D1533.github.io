---
layout: default
title: Shamir Secret Sharing - Introduction
---

---
## Rootme - Shamir Secret Sharing - Introduction
---
## Description

You find yourself in possession of pairs of numbers and your colleague, with a glance, assures you that you have everything you need to decipher the message.

---
## Solution

In the Shamir Secret Sharing, the secret is kept as the constant term $$a_0$$ of a polynomial 
{% raw %}
$$f(x) = \sum_{j=0}^k a_jx^j \ mod \ p.$$ 
{% endraw %}
The challenge gives us five pairs  $$(x_i,y_i)$$ where $$f(x_i) = y_i \ mod \ p$$. The statement of the challenge ensure us that we have everything needed in order to decipher the message. This implies that $$k = 5$$ and the polynomial is of the form
{% raw %}
$$f(x) = a_0 + a_1x + a_2x^2 + a_3x^3 +a_4x^4 +a_5x^5 \ mod \ p$$
{% endraw %}

where $$a_0$$ is the secret.

Notice that by Lagrange interpolation, we have that
{% raw %}
$$f(x) = \sum_{j=0}^k y_j \ell_j(x) \ mod \ p$$
{% endraw %}
where $$\ell(x) = \displaystyle \prod_{\substack{i=0 \\ i \neq j}}^k \dfrac{x-x_i}{x_j-x_i} \ mod \ p.$$

Therefore,

{% raw %}
$$a_0 = f(0) = \sum_{j=0}^k y_j \ell_j(0) = \sum_{j=0}^k y_j \prod_{\substack{i=0 \\ i \neq j}}^k \frac{x_i}{x_i-x_j} \ mod \ p $$
{% endraw %}
*Python script*
```python
from Crypto.Util.number import long_to_bytes


shares=[(1, 79013100870464050914993118468910211729261196722179511531703421264943938779231), 
        (2, 81859373475737111187817325328442924861970610151705005884070969402629136831933),
        (3, 79176316510763586877115268719176284779510214031366879703315941177151748443090), 
        (4, 21018248157552489768219539396148682077728313865167501020008576966214434846295), 
        (5, 76790569466834289402766026417764998519096595406736309045776987620445353131023)]

p = 84983565845414213007403331223956084955206046802241925707351494168212278668259
s = 0

for j in range(5):
    l_j = 1
    for i in range(5):
        if i != j:
            l_j *= (shares[i][0] * pow(shares[i][0]-shares[j][0],-1,p) ) % p
   
    s += shares[j][1]*l_j

print(long_to_bytes(s % p).decode())

```