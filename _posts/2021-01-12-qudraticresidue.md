---
title: Quadratic Residues
date: 2021-01-12 00:00 +0200
categories: [CryptoHack, Mathematics]
tags: [modular math, easy]
---

This is an easy challenge. The prime is small so we can just brute force it. Just run through every possible int from `1` to `p`, square them and check if there are equal to the numbers. If they are then you found a quadratic residue.

```python
p = 29
ints = [14, 6, 11] 

results = []

for i in ints:
    for j in range(p):
        if j**2 % p == i:
            results.append(j)

print(results)
```
