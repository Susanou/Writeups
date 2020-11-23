---
title: RSA Common Modulus
date: 2020-11-20 00:00:00 +0200
categories: [Root Me, Crypto]
tags: [crypto, RSA, modulus, Bezout]
math: true
---

# Common modulus attack

In this case imagine that Alice sent the SAME message more than once using the same public key but thanks to the laws of the world, a problem happened and the public key changed while the modulus stayed the same.

## Conditions

The first conditions for this attack to work is as follows

$$
gcd(e_1, e_2) = 1
\newline
gcd(c_2, n) = 1
$$

## Math to solve

Now let's dive inside of the math required to solve this. RSA is as follows:

$$c = m^e\ mod\ n$$

However, now we have something: we know that e1 and e2 are co-prime. Thus using Bezout's Theorem we can get:

$$xe_1 + ye_2 = gcd(e_1, e_2) = 1$$

Using this we cam derive the original message $m$:

$$
C_1^x * C_2^y = (m_{e_1})^x * (m_{e_2})^y
\newline
 = m^{e_1x + e_2y}
\newline
 = m^1 = m
$$

We do need to adapt this equation a bit as normally in Bezout's Theorem we would get a positive and negative number pair. Let's suppose in this case that y is the negative number:

$$
Let\ y = -a
\newline
$$

$$
C_2^y = C_2^{-a}
\newline
 = (C_2^{-1})^a
\newline
 = (C_2^{-1})^{-y}
$$

For that to be possible, we need to have an inverse for $C_2$ thus why we have two conditions for this attack to work.

Thus to really recover the plaintext we are actually doing:

$$C_1^x * (C_2^{-1})^{-y}$$

# Code for solving

```python
from Crypto.PublicKey import RSA
from base64 import b64decode
from Crypto.Util.number import long_to_bytes, bytes_to_long, GCD as gcd, inverse
from euclid import egcd, inverse


public_key1 = RSA.importKey(open('key1_pub.pem', 'r').read())
print( "n = " + str(n1 := public_key1.n))
print( "e = " + str(e1 := public_key1.e))

public_key2 = RSA.importKey(open('key2_pub.pem', 'r').read())
print( "n = " + str(n2 := public_key2.n))
print( "e = " + str(e2 := public_key2.e))

with open('message1', 'r') as f:
    ct1 = b64decode(f.read())

with open('message2', 'r') as f:
    ct2 = b64decode(f.read())

print(ct1 := bytes_to_long(ct1))
print(ct2 := bytes_to_long(ct2))

assert n1 == n2
assert gcd(e1, e2) == 1
assert gcd(ct2, n1) == 1 # both steps confirm we can use common modulus attack

n = n1

# attack part

s1 = inverse(e1, e2)
s2 = (gcd(e1, e2) - e1*s1)//e2
temp = inverse(ct2, n)
m1 = pow(ct1, s1, n)
m2 = pow(temp, -s2, n)

print(long_to_bytes((m1*m2) % n))
```
