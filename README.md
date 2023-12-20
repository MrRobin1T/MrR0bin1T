# Authority

## IP Address

10.129.87.197

## NMAP Results

```
└─# nmap -sC -sV -p- -T4 -A -Pn 10.129.87.197
Starting Nmap 7.94 ( https://nmap.org ) at 2023-10-13 22:52 EDT
Nmap scan report for 10.129.87.197
Host is up (0.068s latency).
Not shown: 65509 closed tcp ports (reset)
PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Simple DNS Plus
80/tcp    open  http          Microsoft IIS httpd 10.0
|_http-server-header: Microsoft-IIS/10.0
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-title: IIS Windows Server
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2023-10-14 06:53:48Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: authority.htb, Site: Default-First-Site-Name)
| ssl-cert: Subject: 
| Subject Alternative Name: othername: UPN::AUTHORITY$@htb.corp, DNS:authority.htb.corp, DNS:htb.corp, DNS:HTB
| Not valid before: 2022-08-09T23:03:21
|_Not valid after:  2024-08-09T23:13:21
|_ssl-date: 2023-10-14T06:54:59+00:00; +4h00m00s from scanner time.
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: authority.htb, Site: Default-First-Site-Name)
|_ssl-date: 2023-10-14T06:54:58+00:00; +3h59m59s from scanner time.
| ssl-cert: Subject: 
| Subject Alternative Name: othername: UPN::AUTHORITY$@htb.corp, DNS:authority.htb.corp, DNS:htb.corp, DNS:HTB
| Not valid before: 2022-08-09T23:03:21
|_Not valid after:  2024-08-09T23:13:21
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
8443/tcp  open  ssl/https-alt
|_ssl-date: TLS randomness does not represent time
|_http-title: Site doesn't have a title (text/html;charset=ISO-8859-1).
| ssl-cert: Subject: commonName=172.16.2.118
| Not valid before: 2023-10-12T06:49:26
|_Not valid after:  2025-10-13T18:27:50
| fingerprint-strings: 
|   FourOhFourRequest, GetRequest: 
|     HTTP/1.1 200 
|     Content-Type: text/html;charset=ISO-8859-1
|     Content-Length: 82
|     Date: Sat, 14 Oct 2023 06:53:54 GMT
|     Connection: close
|     <html><head><meta http-equiv="refresh" content="0;URL='/pwm'"/></head></html>
|   HTTPOptions: 
|     HTTP/1.1 200 
|     Allow: GET, HEAD, POST, OPTIONS
|     Content-Length: 0
|     Date: Sat, 14 Oct 2023 06:53:54 GMT
|     Connection: close
|   RTSPRequest: 
|     HTTP/1.1 400 
|     Content-Type: text/html;charset=utf-8
|     Content-Language: en
|     Content-Length: 1936
|     Date: Sat, 14 Oct 2023 06:54:00 GMT
|     Connection: close
|     <!doctype html><html lang="en"><head><title>HTTP Status 400 
|     Request</title><style type="text/css">body {font-family:Tahoma,Arial,sans-serif;} h1, h2, h3, b {color:white;background-color:#525D76;} h1 {font-size:22px;} h2 {font-size:16px;} h3 {font-size:14px;} p {font-size:12px;} a {color:black;} .line {height:1px;background-color:#525D76;border:none;}</style></head><body><h1>HTTP Status 400 
|_    Request</h1><hr class="line" /><p><b>Type</b> Exception Report</p><p><b>Message</b> Invalid character found in the HTTP protocol [RTSP&#47;1.00x0d0x0a0x0d0x0a...]</p><p><b>Description</b> The server cannot or will not process the request due to something that is perceived to be a client error (e.g., malformed request syntax, invalid
9389/tcp  open  mc-nmf        .NET Message Framing
47001/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
49664/tcp open  msrpc         Microsoft Windows RPC
49665/tcp open  msrpc         Microsoft Windows RPC
49666/tcp open  msrpc         Microsoft Windows RPC
49667/tcp open  msrpc         Microsoft Windows RPC
49671/tcp open  msrpc         Microsoft Windows RPC
49674/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49675/tcp open  msrpc         Microsoft Windows RPC
49679/tcp open  msrpc         Microsoft Windows RPC
49682/tcp open  msrpc         Microsoft Windows RPC
49691/tcp open  msrpc         Microsoft Windows RPC
49698/tcp open  msrpc         Microsoft Windows RPC
49709/tcp open  msrpc         Microsoft Windows RPC
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port8443-TCP:V=7.94%T=SSL%I=7%D=10/13%Time=652A02C3%P=aarch64-unknown-l
SF:inux-gnu%r(GetRequest,DB,"HTTP/1\.1\x20200\x20\r\nContent-Type:\x20text
SF:/html;charset=ISO-8859-1\r\nContent-Length:\x2082\r\nDate:\x20Sat,\x201
SF:4\x20Oct\x202023\x2006:53:54\x20GMT\r\nConnection:\x20close\r\n\r\n\n\n
SF:\n\n\n<html><head><meta\x20http-equiv=\"refresh\"\x20content=\"0;URL='/
SF:pwm'\"/></head></html>")%r(HTTPOptions,7D,"HTTP/1\.1\x20200\x20\r\nAllo
SF:w:\x20GET,\x20HEAD,\x20POST,\x20OPTIONS\r\nContent-Length:\x200\r\nDate
SF::\x20Sat,\x2014\x20Oct\x202023\x2006:53:54\x20GMT\r\nConnection:\x20clo
SF:se\r\n\r\n")%r(FourOhFourRequest,DB,"HTTP/1\.1\x20200\x20\r\nContent-Ty
SF:pe:\x20text/html;charset=ISO-8859-1\r\nContent-Length:\x2082\r\nDate:\x
SF:20Sat,\x2014\x20Oct\x202023\x2006:53:54\x20GMT\r\nConnection:\x20close\
SF:r\n\r\n\n\n\n\n\n<html><head><meta\x20http-equiv=\"refresh\"\x20content
SF:=\"0;URL='/pwm'\"/></head></html>")%r(RTSPRequest,82C,"HTTP/1\.1\x20400
SF:\x20\r\nContent-Type:\x20text/html;charset=utf-8\r\nContent-Language:\x
SF:20en\r\nContent-Length:\x201936\r\nDate:\x20Sat,\x2014\x20Oct\x202023\x
SF:2006:54:00\x20GMT\r\nConnection:\x20close\r\n\r\n<!doctype\x20html><htm
SF:l\x20lang=\"en\"><head><title>HTTP\x20Status\x20400\x20\xe2\x80\x93\x20
SF:Bad\x20Request</title><style\x20type=\"text/css\">body\x20{font-family:
SF:Tahoma,Arial,sans-serif;}\x20h1,\x20h2,\x20h3,\x20b\x20{color:white;bac
SF:kground-color:#525D76;}\x20h1\x20{font-size:22px;}\x20h2\x20{font-size:
SF:16px;}\x20h3\x20{font-size:14px;}\x20p\x20{font-size:12px;}\x20a\x20{co
SF:lor:black;}\x20\.line\x20{height:1px;background-color:#525D76;border:no
SF:ne;}</style></head><body><h1>HTTP\x20Status\x20400\x20\xe2\x80\x93\x20B
SF:ad\x20Request</h1><hr\x20class=\"line\"\x20/><p><b>Type</b>\x20Exceptio
SF:n\x20Report</p><p><b>Message</b>\x20Invalid\x20character\x20found\x20in
SF:\x20the\x20HTTP\x20protocol\x20\[RTSP&#47;1\.00x0d0x0a0x0d0x0a\.\.\.\]<
SF:/p><p><b>Description</b>\x20The\x20server\x20cannot\x20or\x20will\x20no
SF:t\x20process\x20the\x20request\x20due\x20to\x20something\x20that\x20is\
SF:x20perceived\x20to\x20be\x20a\x20client\x20error\x20\(e\.g\.,\x20malfor
SF:med\x20request\x20syntax,\x20invalid\x20");
No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).
TCP/IP fingerprint:
OS:SCAN(V=7.94%E=4%D=10/13%OT=53%CT=1%CU=36238%PV=Y%DS=2%DC=T%G=Y%TM=652A03
OS:03%P=aarch64-unknown-linux-gnu)SEQ(SP=FE%GCD=1%ISR=105%TI=I%CI=I%II=I%SS
OS:=S%TS=U)OPS(O1=M53CNW8NNS%O2=M53CNW8NNS%O3=M53CNW8%O4=M53CNW8NNS%O5=M53C
OS:NW8NNS%O6=M53CNNS)WIN(W1=FFFF%W2=FFFF%W3=FFFF%W4=FFFF%W5=FFFF%W6=FF70)EC
OS:N(R=Y%DF=Y%T=80%W=FFFF%O=M53CNW8NNS%CC=Y%Q=)T1(R=Y%DF=Y%T=80%S=O%A=S+%F=
OS:AS%RD=0%Q=)T2(R=Y%DF=Y%T=80%W=0%S=Z%A=S%F=AR%O=%RD=0%Q=)T3(R=Y%DF=Y%T=80
OS:%W=0%S=Z%A=O%F=AR%O=%RD=0%Q=)T4(R=Y%DF=Y%T=80%W=0%S=A%A=O%F=R%O=%RD=0%Q=
OS:)T5(R=Y%DF=Y%T=80%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)T6(R=Y%DF=Y%T=80%W=0%S=A%
OS:A=O%F=R%O=%RD=0%Q=)T7(R=Y%DF=Y%T=80%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)U1(R=Y%
OS:DF=N%T=80%IPL=164%UN=0%RIPL=G%RID=G%RIPCK=G%RUCK=G%RUD=G)IE(R=Y%DFI=N%T=
OS:80%CD=Z)

Network Distance: 2 hops
Service Info: Host: AUTHORITY; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required
|_clock-skew: mean: 3h59m59s, deviation: 0s, median: 3h59m58s
| smb2-time: 
|   date: 2023-10-14T06:54:50
|_  start_date: N/A

TRACEROUTE (using port 256/tcp)
HOP RTT      ADDRESS
1   68.06 ms 10.10.14.1
2   69.59 ms 10.129.87.197

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 155.30 seconds
```



