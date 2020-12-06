---
title: Bank Heist
date: 2020-06-30 14:50:00 +0200
categories: [Hack The Box, Challenges, Cryptography]
tags: [crypto, atbash, cipher, phone]
---

# [Bank Heist](https://app.hackthebox.eu/challenges/65)

We get this text message:

```
444333 99966688 277733 7773323444664 84433 22244474433777, 99966688 277733 666552999. 99966688777 777744277733 666333 84433 443344477778 4447777 44466 99966688777 4466688777733. 84433 5533999 8666 84433 55566622255 4447777 22335556669. 4666 8666 727774447777.

47777888 995559888 4555 47777888 44999988 666555997 : 8555444888477744488866888648833369!!
```

and this hint:

```
You get to the scene of a bank heist and find that you have caught one person. Under further analysis of the persons flip phone you see a message that seems suspicious. Can you figure out what the message to put this guy in jail?
```

We can guess that the message is the keys from the flip phone. Usinge [Dcode](https://www.dcode.fr/multitap-abc-cipher) we get:

```
IIGF YOU ARE READING THE CIPHER YOU ARE OKAY YOUR SHARE OF/ODE/OED THE HEIST GHS/HGS/IRP/IS IN/IMM YOUR HOUSE THE KEY TO THE LOCK GHS/HGS/IRP/IS BELOW GO TO PARIS GSV XLWV GL GSV HZU OLXP TLIVGRIVNVMGUFMW
```

Now we know that the first word of the last sentence are probably: `THE`
Either you realize that the sum of `G+T=25` and the same goes for the rest and you can build a python program to crack it or you realize that its the atbash cipher and you get the following

> ! THE CODE TO THE SAF LOCK [FLAG]
