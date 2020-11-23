---
title: RSA Multiple Recipients
date: 2020-11-17 00:00:00 +0200
categories: [Root Me, Cryptography]
tags: [crypto, RSA, public key, Hastad]
math: true
---

# Hastad Attack

Let's go over the math of the attack:

Assume Bob sends the same message to multiple people using RSA with different modulus but the same e for all of the messages. Assume that this e is equal to 3 for this example. We know the following:

$$
P_i = (N_i, e)\ such\ that\ gcd(N_i, N_j) = 1\ \forall i\not = j
\newline
y_1 = x^3[N_1]\newline
y_2 = x^3[N_2]\newline
y_3 = x^3[N_3]
$$

Using the chinese reminder theorem, we can find:

$$
y'=x^3 [N_1*N_2*N_3]
$$

Since we have $x \leq N_i$ we also have the followinf:

$$
x \leq N_i \forall i \Rightarrow x^3 \leq \prod_i^3 N_i
$$

Then to calculate x we just need to take the 3rd root of y'. This attack can be generalized to any exponent e provided that the number of messages sent is greater or equal to the value of the public exponent.

# Solution

```python
from Crypto.PublicKey import RSA
import base64
from binascii import hexlify, unhexlify
from Crypto.Util.number import long_to_bytes, bytes_to_long, GCD as gcd, inverse
from sage.all import *
import libnum

msg1='''-----BEGIN PUBLIC KEY-----
MIIBIDANBgkqhkiG9w0BAQEFAAOCAQ0AMIIBCAKCAQEAvtCUgebFQ43qMwaIZZ+w
3aO5NJJLhtUi8THUGk/dScoGY0a6cW0rKuawqmVeoccvx4ZyUy4htOpeJoqjIbij
HF/6saD36DvDeJKZlynZujIkZSN/ywCsaDsWypqkDIRJaFfGAY2Px8WPOX0GVui2
yFVT+aFPYZZf7OC4Xu7pm4Eov4tM/+Jb54pLJplEMgE41F9KFshBkqtYlfxkpPJ5
aocjS0jEby34cZ8o79rQIhGGUuPSXkTaPlH8EzROHDq9deQMYklJspN1urNuH/mm
kt856tyhefJrRBfNXXEgH33u970FMzlBV+uc0pJgLEgUSMPCCjbhiddzOxKsOYiP
OwIBAw==
-----END PUBLIC KEY-----'''
msg2='''-----BEGIN PUBLIC KEY-----
MIIBIDANBgkqhkiG9w0BAQEFAAOCAQ0AMIIBCAKCAQEAugXwzOVLmHw+2t0g/jhq
/JG7oF9ryVktXuPU/UMNfcMVA/WFemx1aVtGqpP/eSxVw/f8gUpZtQ8BA3x6Zr2a
irKBO0Wfx2BPDL8+e0rsqHW2wIBoV1n4TglWonap94nfmmz10y7Q7SjQVzDQaAif
v6LObE2smiuCCQobhAGaP2meDSa44M0ZuQmAlw0yMb+b4XwS6XCwX0JS2Yt4Rw51
xhtBV0idQc3acrLg7J9+MQrfuL/yneNEZUy/7nJDKAsfgh/lyQI5qFCqS+Q9UMQg
I1LMl7IR18tyihQs8t961zmZEZ1HjftnapQ8W3yuMLoOGidOI9TMY1UEtLRUxxf2
IQIBAw==
-----END PUBLIC KEY-----'''
msg3='''-----BEGIN PUBLIC KEY-----
MIIBIDANBgkqhkiG9w0BAQEFAAOCAQ0AMIIBCAKCAQEAq8dX4iDR8NpG0BfQVCJz
+q6sqZ1o6jtAD5l1K13r22euMQfY9JH5F7oHnhRwj3AvyPwaYYn8H0HrozRIJpVk
KvyAg9U92newtYwlGSbtd5RGoTe7xO/jVysNYd/BpSTCx4kabea9pbFbFgQoTMqY
uX9uYJdMsX8KWxoXvvIAfgL6iovw4Cad5E2sJcfHgmy6wgGUNTc+iHZY5fr3M+CK
4EoFL6UUR//7O57t9/7sDI5iqw5RxRbbU37pPZ94CyDhZl2OnlQdXo3pqi6hRHtW
0HdwouW3LKoIkZiNl7d/OCdIMIFL1I4ziACwg3/vQTW62I9ReJQUBs/bgx/hgXGh
PwIBAw==
-----END PUBLIC KEY-----'''

key1 = RSA.importKey(msg1)
n0= int(key1.n)

key2 = RSA.importKey(msg2)
n1= int(key2.n)

key3= RSA.importKey(msg3)
n2= int(key3.n)

m0='''vjXJvWis95tc25G+wxC5agClCJFB9vUslFyV+I4bSiwS4Sm6k8eF61EizKo4hZFwROlO3Ci3YQaT
rAm+Y9/qEbM7asvwKTePKX+cLVN61l0xxfTL8CdoXkRE2rSczp1AzzmFz83OHgszX/Wf7kgWU4M7
3+efPvU9FmcWOauakrdJZx8B0ErJ5cYWNS0ZCam0Nlz+pISqdSJ6MSz0Ek62Ulb3ei8I41FOdHtd
8mhC7dfpdfmVLOSEW4yEnn5iuMW0ydvW055dodLc9RKcvJafH9e3zf8/S1/RORXZoUHWnBXBEFOl
8iVXz70GcDTPzIhxh6imi0ynLbV68qW2vw2XRQ=='''
m1='''Mi4MnobVabt1+Q7R/0aIYBBpeRxPRuR6gkhr/Wbw/D23ywu2KbUYBab2XbEguRz8yzBlxScbFjjb
98DuILxURoFN8lNKVJLS3d/IrGGr0hjocbz27uBS97hseX0S4nd+BL6LUn0o7qe/yCqk7Y0tEhhc
uSMrGn/l9N/6UjgN38TyoyvVbd1UH6BHGLdj9g5JRBKAvcGumymfiE0qS7IOnM3mN5W8gRJaEGDk
hDim21Pm1Yg2GRBJ+z7C8AEy+Dz6OFvWuDsY24Gb+643D7KrmFObJ1n8Qyme/Y1bfvBkG+xdvGoB
zyQrlDT12Qjkfoqb37HNrGUUD5cj2q54gyp80w=='''
m2='''hMQVVspFGh7NxCqLf7DVto3OxgDLn9n9gOCjOjEOYhi/VwZ3adFsbBL+zLZxYNdkKLNWCNRktRwp
nWriEsW1uDnVt2LbxSLvjvRKbR/hyvpY+0UUZFS6wCWQjGyxUydDxQ88jNM5dY58/1nxsd04I3n3
Mt97SuqwBN1+4VS3SsqtbR0GU1C7ODkPoCeGVd3PNkGHPgbT7QzMwxl63Pl3i/sp0I2/gqSnKu5C
DS7e2WELz0hfiOJ2v2RvIon2EEbPwx1/6zxZlMhHuGXHNZKDtyqe6Dd+EIjpwhQFW3eH7fDIirRb
aPPXAYsoypS5eFD3mIWUs4yVOH9ykkdKQ9FNwg=='''


m0=base64.b64decode(m0)
m0=bytes_to_long(m0)
m1=base64.b64decode(m1)
m1=bytes_to_long(m1)
m2=base64.b64decode(m2)
m2=bytes_to_long(m2)

N0 = n1*n2
N1 = n0*n2
N2 = n0*n1

N012 = n0*n1*n2

u0 = inverse(N0, n0)
u1 = inverse(N1, n1)
u2 = inverse(N2, n2)

valeur=(m0*u0*N0+m1*u1*N1+m2*u2*N2)%(n0*n1*n2)

root = libnum.nroot(valeur, 3)

print(long_to_bytes(root))

```
