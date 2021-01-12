---
title: Legendre Symbol
date: 2021-01-12 00:00 +0200
categories: [CryptoHack, Mathematics]
tags: [modular math, easy]
---

This makes you create a function to calculate if a number is a quadratic root faster. The math is pretty simple so here is the code for the function.
*NB: since pow() returns a positive interger and I personnaly check for {1,-1,0} I return -1 if ls=p-1 which is the same mod p*

```python
def legendre_symbol(a: int, p: int):
    ls = pow(a, (p-1)//2, p)
    return -1 if ls == p - 1 else ls
```
