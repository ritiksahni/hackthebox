# RastaLabs - HackTheBox Prolab

Sshuttle Command for proxying VPS to local machine:

```
sshuttle -vr root@192.227.223.218 -x 192.227.223.218 0/0 --ssh-cmd 'ssh -i /root/Documents/hackthebox/prolabs/rastalabs/http.deep-id_rsa.key'
```

Entry Point: 10.10.110.0/24

## 10.10.110.254

It has 2 ports open (80,443).

Website enumeration on port 80 tells us about employees and their potential usernames:

```
rweston
epugh
ngodfrey
ahope
bowen
tquinn
```
