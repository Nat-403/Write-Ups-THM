### Snort Challenge - Live Attacks

### Scenario 1 | Brute-Force

First of all, start Snort in sniffer mode and try to figure out the attack source, service and port.

Then, write an IPS rule and run Snort in IPS mode to stop the brute-force attack. Once you stop the attack properly, you will have the flag on the desktop!

Here are a few points to remember:

Create the rule and test it with "-A console" mode. 
Use "-A full" mode and the default log path to stop the attack.
Write the correct rule and run the Snort in IPS "-A full" mode.
Block the traffic at least for a minute and then the flag file will appear on your desktop.

```

sudo snort -v -l /home/ubuntu/Desktop  ### Sniffer Mode

sudo snort -v -r snort.log.1750007376   ### Read snort file

sudo snort -r snort.log.1750007376 -X | grep :22   ### Read and filtred snort file

WARNING: No preprocessors configured for policy 0.
06/15-17:09:37.184492 10.10.245.36:46654 -> 10.10.140.29:22
TCP TTL:64 TOS:0x0 ID:21086 IpLen:20 DgmLen:948 DF
***AP*** Seq: 0x65DDA6D3  Ack: 0xF4F08BC  Win: 0x1EB  TcpLen: 32
TCP Options (3) => NOP NOP TS: 1884579559 4119686988 
0x0000: 02 6D 84 B4 B4 1B 02 67 7A 27 40 23 08 00 45 00  .m.....gz'@#..E.
0x0010: 03 B4 52 5E 40 00 40 06 4F 90 0A 0A F5 24 0A 0A  ..R^@.@.O....$..
0x0020: 8C 1D B6 3E 00 16 65 DD A6 D3 0F 4F 08 BC 80 18  ...>..e....O....
0x0030: 01 EB 72 1A 00 00 01 01 08 0A 70 54 66 E7 F5 8D  ..r.......pTf...
0x0040: 6F 4C 00 00 03 7C 0B 14 82 BB B5 D8 84 E0 F6 DF  oL...|..........
0x0050: 21 64 52 BE EF 3F 79 7C 00 00 00 71 63 75 72 76  !dR..?y|...qcurv
0x0060: 65 32 35 35 31 39 2D 73 68 61 32 35 36 40 6C 69  e25519-sha256@li
0x0070: 62 73 73 68 2E 6F 72 67 2C 65 63 64 68 2D 73 68  bssh.org,ecdh-sh
0x0080: 61 32 2D 6E 69 73 74 70 32 35 36 2C 65 63 64 68  a2-nistp256,ecdh
0x0090: 2D 73 68 61 32 2D 6E 69 73 74 70 33 38 34 2C 65  -sha2-nistp384,e
0x00A0: 63 64 68 2D 73 68 61 32 2D 6E 69 73 74 70 35 32  cdh-sha2-nistp52
0x00B0: 31 2C 64 69 66 66 69 65 2D 68 65 6C 6C 6D 61 6E  1,diffie-hellman
0x00C0: 2D 67 72 6F 75 70 31 34 2D 73 68 61 31 00 00 01  -group14-sha1...
0x00D0: 8B 72 73 61 2D 73 68 61 32 2D 35 31 32 2D 63 65  .rsa-sha2-512-ce
0x00E0: 72 74 2D 76 30 31 40 6F 70 65 6E 73 73 68 2E 63  rt-v01@openssh.c
0x00F0: 6F 6D 2C 72 73 61 2D 73 68 61 32 2D 32 35 36 2D  om,rsa-sha2-256-
0x0100: 63 65 72 74 2D 76 30 31 40 6F 70 65 6E 73 73 68  cert-v01@openssh
0x0110: 2E 63 6F 6D 2C 73 73 68 2D 72 73 61 2D 63 65 72  .com,ssh-rsa-cer
0x0120: 74 2D 76 30 31 40 6F 70 65 6E 73 73 68 2E 63 6F  t-v01@openssh.co
0x0130: 6D 2C 73 73 68 2D 64 73 73 2D 63 65 72 74 2D 76  m,ssh-dss-cert-v
...


sudo snort -r snort.log.1750007376 -X | grep :"ssh"

sudo snort -r snort.log.1750007376 -X -n 30

sudo nano local.rules    ### Créate snort rules

alert tcp any 22 <> any any (msg:"SSH connection attemped"; sid:10000001; rev:1;)

sudo snort -c local.rules -v -A full   ### IPS mode

```

Desktop -> flag.txt

THM{81b7fef657f8aaa6e4e200d616738254}

What is the name of the service under attack?   SSH

What is the used protocol/port in the attack?   TCP/22 

### Scenario 2 | Reverse-Shell

First of all, start Snort in sniffer mode and try to figure out the attack source, service and port.

Then, write an IPS rule and run Snort in IPS mode to stop the brute-force attack. Once you stop the attack properly, you will have the flag on the desktop!

Here are a few points to remember:

Create the rule and test it with "-A console" mode. 
Use "-A full" mode and the default log path to stop the attack.
Write the correct rule and run the Snort in IPS "-A full" mode.
Block the traffic at least for a minute and then the flag file will appear on your desktop.

```

sudo snort -v -l /home/ubuntu/Desktop   ### Sniffer Mode

sudo snort -v -r snort.log.1750011165 -X  ### Read snort file

sudo snort -r snort.log.1750011165 -X | grep :4444

sudo nano local.rules   ### Créate snort rules

#alert tcp any 4444 <> any any (msg:"Traffic detectied"; sid:10000001; rev:1;)
drop tcp any 4444 <> any any (msg:"Reverse Shell detected"; sid:10000001; rev:1;)


sudo snort -c local.rules -v -A full -l /home/ubuntu/Desktop

cat alert

sudo snort -c local.rules -q -Q --daq afpacket -i eth0:eth1 -A full -l /home/ubuntu/Desktop

```

Desktop -> flag.txt

THM{0ead8c494861079b1b74ec2380d2cd24}

What is the used protocol/port in the attack?  tcp/4444

Which tool is highly associated with this specific port number?  Metasploit
