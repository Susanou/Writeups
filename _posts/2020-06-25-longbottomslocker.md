---
title: Longbottom's Locker
date: 2020-06-25 00:00:00 +0200
categories: [Hack The Box, Challenges, Misc]
tags: [misc, binwalk, easy]
---

# Start

When we unzip the folder, we get three files `index.html`, `neville.gif` and `socute.jpg`.

The index file and gif file look part of one another and may be there for validation. So let's look at `socute.jpg`

When we use binwalk, we realize that there is more stuff in that file. Let's unzip everything in there.

# Solution

Using binwalk, we get a binary file named `donotshare`. Looking at it, we can use pickle to get something we can understand.
Using this script below we get a text banner. Enter that into the form in `index.html` and you get the flag for the challeneg

```python
#!/usr/bin/python2

import pickle

with open('banner.txt', 'rb') as f:
    o = pickle.load(f)

outstr = ''
for line in o:
    for char,n in line:
        outstr += char*n
    outstr += '\n'
print outstr
```

<html>
<head>
  <link rel="stylesheet" type="text/css" href="{{site.baseurl}}/assets/css/asciinema-player.css" />
</head>
<body>
  <asciinema-player src="{{site.baseurl}}/assets/recs/challenges/misc/longbottom.cast" autoplay="1"  speed="3" loop="1" cols="150" rows="20"></asciinema-player>
  <script src="{{site.baseurl}}/assets/js/asciinema-player.js"></script>
</body>
</html>