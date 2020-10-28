---
title: Chunk Norris 
date: 2020-08-30 14:50:00 +0200
categories: [Google CTF, Cryptography]
tags: [crypto, RSA, prime generation]
math: true
---
# Chunk Norris (easy)

## Code given

```python
#!/usr/bin/python3 -u

import random
from Crypto.Util.number import *
import gmpy2

a = 0xe64a5f84e2762be5
chunk_size = 64

def gen_prime(bits):
  s = random.getrandbits(chunk_size)

  while True:
    s |= 0xc000000000000001
    p = 0
    for _ in range(bits // chunk_size):
      p = (p << chunk_size) + s
      s = a * s % 2**chunk_size
    if gmpy2.is_prime(p):
      return p

n = gen_prime(1024) * gen_prime(1024)
e = 65537
flag = open("flag.txt", "rb").read()
print('n =', hex(n))
print('e =', hex(e))
print('c =', hex(pow(bytes_to_long(flag), e, n)))
```

```txt
n = 0xab802dca026b18251449baece42ba2162bf1f8f5dda60da5f8baef3e5dd49d155c1701a21c2bd5dfee142fd3a240f429878c8d4402f5c4c7f4bc630c74a4d263db3674669a18c9a7f5018c2f32cb4732acf448c95de86fcd6f312287cebff378125f12458932722ca2f1a891f319ec672da65ea03d0e74e7b601a04435598e2994423362ec605ef5968456970cb367f6b6e55f9d713d82f89aca0b633e7643ddb0ec263dc29f0946cfc28ccbf8e65c2da1b67b18a3fbc8cee3305a25841dfa31990f9aab219c85a2149e51dff2ab7e0989a50d988ca9ccdce34892eb27686fa985f96061620e6902e42bdd00d2768b14a9eb39b3feee51e80273d3d4255f6b19
e = 0x10001
c = 0x6a12d56e26e460f456102c83c68b5cf355b2e57d5b176b32658d07619ce8e542d927bbea12fb8f90d7a1922fe68077af0f3794bfd26e7d560031c7c9238198685ad9ef1ac1966da39936b33c7bb00bdb13bec27b23f87028e99fdea0fbee4df721fd487d491e9d3087e986a79106f9d6f5431522270200c5d545d19df446dee6baa3051be6332ad7e4e6f44260b1594ec8a588c0450bcc8f23abb0121bcabf7551fd0ec11cd61c55ea89ae5d9bcc91f46b39d84f808562a42bb87a8854373b234e71fe6688021672c271c22aad0887304f7dd2b5f77136271a571591c48f438e6f1c08ed65d0088da562e0d8ae2dadd1234e72a40141429f5746d2d41452d916
```

## RSA algorithm (reminder)

$$n = p*q$$

$$c = m^e mod\ n$$

where:

- `p` and q are large prime numbers
- `e` is the public key
- `m` is the message
- `c` is the cipher text

The RSA algorithm is based around the idea of asymeytric keys or in this case, the fact that for large primes, it is hard to computer a number `d` such that:

$$m = c^d mod\ n$$

## Large Prime number generation

```python
a = 0xe64a5f84e2762be5
chunk_size = 64

def gen_prime(bits):
  s = random.getrandbits(chunk_size)

  while True:
    s |= 0xc000000000000001
    p = 0
    for _ in range(bits // chunk_size):
      p = (p << chunk_size) + s
      s = a * s % 2**chunk_size
    if gmpy2.is_prime(p):
      return p
```

When we look at the above function, we can see the way the challenge generates primes. It can be resumed by the following equation:

$$p = \sum^{15}_{i = 0}[s_i*2^{64*(15-i)}]$$

$$s_i \equiv a*s_{i-1}\ mod\ 2^{64}$$

where `s` is the random number that was used to generate the prime.

NB: as we can see in the function, the number s might be different than the one picked by the system originaly. For our purposes, this has no incidence as we only care about the number that generated the prime and we can consider it to be random.

## Reversing the primes

Now we know that `n=pq` thus we have:

$$n = \sum^{15}_{i = 0}[s_i*2^{64*(15-i)}] * \sum^{15}_{i = 0}[s'_i*2^{64*(15-i)}]$$


where $s_i$ and $s'_i$ are the two random numbers that generate the primes

Now the trick to this challenge was to not try and factor `n` directly but rather factor the random numbers. Here let's first try and find their product

### Getting the product of random numbers

We know:

$$s_i \equiv a*s_{i-1}\ mod\ 2^{64}$$

Howver $a = 16 594 180 801 339 730 917$ and is an odd number thus it has an inverse $mod\ 2^{64}$

Thus we have:

$$s_i \equiv a*s_{i-1}\ mod\ 2^{64}$$

$$\Rightarrow s_{i-1} \equiv a^{-1}*s_{i}\ mod\ 2^{64}$$

Now we can simplify $n$:

$$n = \sum^{15}_{i = 0}[s_i*2^{64*(15-i)}] * \sum^{15}_{i = 0}[s'_i*2^{64*(15-i)}]$$

$$\Rightarrow n = s_{15}*s'_{15}*2^{64}*(s_{14}s'_{15}+s_{15}s'_{14})\ mod\ 2^{128}$$

$$\Rightarrow n = s_{15}*s'_{15}*2^{64}*(2*s_{15}s'_{15}*a^{-1})\ mod\ 2^{128}$$

$$\Rightarrow n = s_{15}*s'_{15}*(2^{65}*a^{-1}+1)\ mod\ 2^{128}$$


Since $(2^{65}*a^{-1}+1)$ is an odd number it must have an inverse $mod\ 2^{128}$. Thus we get:


$$n = s_{15}*s'_{15}*(2^{65}*a^{-1}+1)\ mod\ 2^{128}$$

$$\Leftrightarrow s_{15}*s'_{15} = n*(2^{65}*a^{-1}+1)^{-1}\ mod\ 2^{128}$$

Now once we calculate and factor this we get the following factors:

```txt
13, 167541865434116759, 11, 109, 223, 1290533, 4608287
```

Now we just need to find two numbers out of this set that have more or less the same length bit wise which are

```txt
s1 = 13 * 167541865434116759
t1 = 11 * 109 * 223 * 1290533 * 4608287
```

## Back to decrypting RSA

Now that we have the two random numbers we can  get to getting the `d` value that we need to decrypt:

```python
b = inverse(a, pow(2, 64))

def remverse_primes(s):
    p = s
    for i in range(1,16):
        s = b * s % 2^64
        p += s * 2^(64*i)
    return p
```

Now with both prime we calculate the inverse of `e` modulo phi of n which is equal to $\phi = (p-1)(q-1)$. Once we have `d` we just need to raise the cipher text to that power and we have the clear flag

## FULL CODE

```python
from Crypto.Util.number import long_to_bytes
from euclid import inverse # custom inverse fucntion nothing fancy

a = 0xe64a5f84e2762be5

n = 0xab802dca026b18251449baece42ba2162bf1f8f5dda60da5f8baef3e5dd49d155c1701a21c2bd5dfee142fd3a240f429878c8d4402f5c4c7f4bc630c74a4d263db3674669a18c9a7f5018c2f32cb4732acf448c95de86fcd6f312287cebff378125f12458932722ca2f1a891f319ec672da65ea03d0e74e7b601a04435598e2994423362ec605ef5968456970cb367f6b6e55f9d713d82f89aca0b633e7643ddb0ec263dc29f0946cfc28ccbf8e65c2da1b67b18a3fbc8cee3305a25841dfa31990f9aab219c85a2149e51dff2ab7e0989a50d988ca9ccdce34892eb27686fa985f96061620e6902e42bdd00d2768b14a9eb39b3feee51e80273d3d4255f6b19
e = 0x10001
c = 0x6a12d56e26e460f456102c83c68b5cf355b2e57d5b176b32658d07619ce8e542d927bbea12fb8f90d7a1922fe68077af0f3794bfd26e7d560031c7c9238198685ad9ef1ac1966da39936b33c7bb00bdb13bec27b23f87028e99fdea0fbee4df721fd487d491e9d3087e986a79106f9d6f5431522270200c5d545d19df446dee6baa3051be6332ad7e4e6f44260b1594ec8a588c0450bcc8f23abb0121bcabf7551fd0ec11cd61c55ea89ae5d9bcc91f46b39d84f808562a42bb87a8854373b234e71fe6688021672c271c22aad0887304f7dd2b5f77136271a571591c48f438e6f1c08ed65d0088da562e0d8ae2dadd1234e72a40141429f5746d2d41452d916

ss = n * inverse(pow(2,65) * inverse(a, pow(2,64)) + 1, pow(2,128)) % pow(2,128)
print(ss)

#Use factordb or anything to get the factors of the product
s1 = 13 * 167541865434116759
s2 = 11 * 109 * 223 * 1290533 * 4608287

b = inverse(a, pow(2,64))

def reverse_prime(s):
    p = s
    for i in range(1,16):
        s = b * s % pow(2,64)
        p += s * 2^(64*i)
    return p

p = reverse_prime(s1)
q = reverse_prime(s2)
phi = (p-1)*(q-1)
d = inverse(e, phi)
print(long_to_bytes(pow(c,d,n)).decode())
```

## FLAG

CTF{__donald_knuths_lcg_would_be_better_well_i_dont_think_s0__}