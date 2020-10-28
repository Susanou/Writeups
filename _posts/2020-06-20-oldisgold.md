---
title: Old Is Gold
date: 2020-06-20 14:50:00 +0200
categories: [Hack The Box, Challenges, Misc]
tags: [misc, algorithm, pdfcrack, easy]
---

# [0ld is g0ld](https://app.hackthebox.eu/challenges/31)

Once we unzip the folder, we get a password protected pdf file. Use `pdfcrack` in combination with `rockyou.txt` to get the password:

```bash
pdfcrack -f 0ld\ is\ g0ld.pdf -w /usr/share/wordlists/rockyou.txt 
PDF version 1.6
Security Handler: Standard
V: 2
R: 3
P: -1060
Length: 128
Encrypted Metadata: True
FileID: 5c8f37d2a45eb64e9dbbf71ca3e86861
U: 9cba5cfb1c536f1384bba7458aae3f8100000000000000000000000000000000
O: 702cc7ced92b595274b7918dcb6dc74bedef6ef851b4b4b5b8c88732ba4dac0c
Average Speed: 51086.5 w/s. Current Word: 'ashley76'
Average Speed: 51789.1 w/s. Current Word: 'aleyna17'
Average Speed: 51673.8 w/s. Current Word: 'treefrog23'
Average Speed: 52042.8 w/s. Current Word: 'rubiceltqmbogi'
Average Speed: 51936.8 w/s. Current Word: 'naysha11'
Average Speed: 51216.5 w/s. Current Word: 'lilmizhyper15'
found user-password: 'jumanji69'
```

Use the password and we can see this:

![pdfScreen]({{site.baseurl}}/assets/img/HackTheBox/misc/pdfScreen.png)

The picture is of `Samuel Morse` which gives you a little hint at to what is to come. At the bottom of the pdf, you see:

![morseCode]({{site.baseurl}}/assets/img/HackTheBox/misc/morse.png)

Use a morse decoder and get the flag