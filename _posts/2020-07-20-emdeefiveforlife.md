---
title: Emdee Five for Life
date: 2020-07-20 14:50:00 +0100
categories: [Hack The Box, Challenges, Web]
tags: [web, md5, programing]
---

# [Emdee Five For Life](https://app.hackthebox.eu/challenges/67)

The idea of this challenge is to get the string and encrypt it using MD5 as fast as possible.

Just write a small python script and get the flag from the result

```python
import requests, os, re
import urllib, hashlib

address = 'http://docker.hackthebox.eu:31450/' # CHANGE THIS

r = requests.session()

html = r.get(address) 
#print(html.text)

regex = r"<h3 align=\'center\'>(.*)<\/h3>"
html = re.search(regex, html.text)
print(html)
match = html.group(1)
match.encode("utf-8")

print(match)

result = r.post(address, data={'hash':hashlib.md5(str(match).encode('utf-8')).hexdigest()})
print(result.text)
```