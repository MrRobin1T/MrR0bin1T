# Netmon

## NMAP

```bash
└─# nmap -sC -sV -p- -T4 -A 10.129.63.69
Starting Nmap 7.94 ( https://nmap.org ) at 2023-11-07 21:14 EST
Nmap scan report for 10.129.63.69
Host is up (0.063s latency).
Not shown: 65522 closed tcp ports (reset)
PORT      STATE SERVICE      VERSION
21/tcp    open  ftp          Microsoft ftpd
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| 02-02-19  11:18PM                 1024 .rnd
| 02-25-19  09:15PM       <DIR>          inetpub
| 07-16-16  08:18AM       <DIR>          PerfLogs
| 02-25-19  09:56PM       <DIR>          Program Files
| 02-02-19  11:28PM       <DIR>          Program Files (x86)
| 02-03-19  07:08AM       <DIR>          Users
|_08-23-23  01:28AM       <DIR>          Windows
| ftp-syst: 
|_  SYST: Windows_NT
80/tcp    open  http         Indy httpd 18.1.37.13946 (Paessler PRTG bandwidth monitor)
| http-title: Welcome | PRTG Network Monitor (NETMON)
|_Requested resource was /index.htm
|_http-trane-info: Problem with XML parsing of /evox/about
|_http-server-header: PRTG/18.1.37.13946
135/tcp   open  msrpc        Microsoft Windows RPC
139/tcp   open  netbios-ssn  Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds Microsoft Windows Server 2008 R2 - 2012 microsoft-ds
5985/tcp  open  http         Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
47001/tcp open  http         Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
49664/tcp open  msrpc        Microsoft Windows RPC
49665/tcp open  msrpc        Microsoft Windows RPC
49666/tcp open  msrpc        Microsoft Windows RPC
49667/tcp open  msrpc        Microsoft Windows RPC
49668/tcp open  msrpc        Microsoft Windows RPC
49669/tcp open  msrpc        Microsoft Windows RPC
No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).
TCP/IP fingerprint:
OS:SCAN(V=7.94%E=4%D=11/7%OT=21%CT=1%CU=32714%PV=Y%DS=2%DC=T%G=Y%TM=654AEF7
OS:8%P=aarch64-unknown-linux-gnu)SEQ(SP=103%GCD=1%ISR=10E%TI=I%CI=I%II=I%SS
OS:=S%TS=A)OPS(O1=M53CNW8ST11%O2=M53CNW8ST11%O3=M53CNW8NNT11%O4=M53CNW8ST11
OS:%O5=M53CNW8ST11%O6=M53CST11)WIN(W1=2000%W2=2000%W3=2000%W4=2000%W5=2000%
OS:W6=2000)ECN(R=Y%DF=Y%T=80%W=2000%O=M53CNW8NNS%CC=Y%Q=)T1(R=Y%DF=Y%T=80%S
OS:=O%A=S+%F=AS%RD=0%Q=)T2(R=Y%DF=Y%T=80%W=0%S=Z%A=S%F=AR%O=%RD=0%Q=)T3(R=Y
OS:%DF=Y%T=80%W=0%S=Z%A=O%F=AR%O=%RD=0%Q=)T4(R=Y%DF=Y%T=80%W=0%S=A%A=O%F=R%
OS:O=%RD=0%Q=)T5(R=Y%DF=Y%T=80%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)T6(R=Y%DF=Y%T=8
OS:0%W=0%S=A%A=O%F=R%O=%RD=0%Q=)T7(R=Y%DF=Y%T=80%W=0%S=Z%A=S+%F=AR%O=%RD=0%
OS:Q=)U1(R=Y%DF=N%T=80%IPL=164%UN=0%RIPL=G%RID=G%RIPCK=G%RUCK=G%RUD=G)IE(R=
OS:Y%DFI=N%T=80%CD=Z)

Network Distance: 2 hops
Service Info: OSs: Windows, Windows Server 2008 R2 - 2012; CPE: cpe:/o:microsoft:windows

Host script results:
| smb-security-mode: 
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required
|_clock-skew: mean: -1s, deviation: 0s, median: -1s
| smb2-time: 
|   date: 2023-11-08T02:16:17
|_  start_date: 2023-11-08T02:12:56

TRACEROUTE (using port 23/tcp)
HOP RTT      ADDRESS
1   65.41 ms 10.10.14.1
2   65.46 ms 10.129.63.69

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 131.06 seconds
```

## Port 80

<figure><img src="../.gitbook/assets/image (19).png" alt=""><figcaption></figcaption></figure>

```bash
└─# curl --head -L http://10.129.63.69/index.htm
HTTP/1.1 200 OK
Connection: close
Content-Type: text/html; charset=UTF-8
Content-Length: 33654
Date: Wed, 08 Nov 2023 03:06:13 GMT
Expires: 0
Cache-Control: no-cache
X-Content-Type-Options: nosniff
X-XSS-Protection: 1; mode=block
X-Frame-Options: DENY
Server: PRTG/18.1.37.13946
```

```bash
└─# ftp 10.129.63.69
Connected to 10.129.63.69.
220 Microsoft FTP Service
Name (10.129.63.69:root): anonymous
331 Anonymous access allowed, send identity (e-mail name) as password.
Password: 
230 User logged in.
Remote system type is Windows_NT.
ftp> cd users
250 CWD command successful.
ftp> cd public
250 CWD command successful.
ftp> get user.txt
local: user.txt remote: user.txt
229 Entering Extended Passive Mode (|||50563|)
150 Opening ASCII mode data connection.
100% |********************************************************************************|    34        0.52 KiB/s    00:00 ETA
226 Transfer complete.
34 bytes received in 00:00 (0.52 KiB/s)
ftp> ls
229 Entering Extended Passive Mode (|||50564|)
150 Opening ASCII mode data connection.
02-02-19  11:18PM       <DIR>          Desktop
02-03-19  07:05AM       <DIR>          Documents
07-16-16  08:18AM       <DIR>          Downloads
07-16-16  08:18AM       <DIR>          Music
07-16-16  08:18AM       <DIR>          Pictures
11-07-23  09:13PM                   34 user.txt
07-16-16  08:18AM       <DIR>          Videos
226 Transfer complete.
ftp> quit
221 Goodbye.
                                                                                                                             
┌──(root㉿RM-MSMGK03)-[~/Downloads/HTB/netmon]
└─# cat user.txt 
a9d8b9faf9e7ec515c2234d2eea2defb
```
