## **The Big Picture**

To change the way you attack (and defend) Active Directory networks, change the way you think about Active Directory.

What I mean by the above is this... 

- _**Focus on the holistic nature of Active Directory as opposed to individual parts**_
- Shift from memorized situational rehearsals to thinking about why the situation exists
- Don't spend so much time thinking about the attacks and what tools to use
- Make it less about the tool and more about the reason the tool was developed

### **The Core of Active Directory**

At its core, Active Directory is an _**Identity and Access Management**_ suite. The _**directory**_ component manages the identities and couples with _**AAA (Authentication, Authorization, and Accounting)**_ protocols and services, as well as policies, to provision and manage access.

- Active Directory is a product designed for businesses
- Businesses hire employees
- These employees have a lifecycle at the business
    - They're hired
    - They need a computer
        - The computer is registered in the directory
        - The computer also now has an identity in the directory
    - The employee needs a credential, a mailbox, maybe a VoIP number
        - The credential is their identity + password
        - The identity is registered in the directory and follows them over the course of their employment
    - They need access to applications and systems
        - Applications are typically run by service principals (service accounts), which are also identities 
        - Groups are organizational units created in Active Directory
        - Groups are then typically assigned to particular applications to manage access
        - The user identity is then typically assigned to the group
    - They may need privileged access
        - A separate identity should be created for privileged tasks
        - This identity should be added to any privileged groups
    - They leave, they're laid off, or they're fired
        - Their identity or identities is/are disabled and eventually deleted from the directory
        - Their group memberships are stripped
        - Their computer is de-registered from the directory

I am repeating this concept of identities so much, cause they are truly the bedrock of Active Directory penetration testing (and defending). Your ability to penetrate an Active Directory network will only be as good as your ability to enumerate **users, computers, service accounts,** and **groups**.  
  
What groups does this user belong to?  
What resources do these groups have access to?  
What file shares can this user access?  
What specific permissions does this service account have?  
  
That's why tools such as Bloodhound are so powerful, because they do the heavy lifting of enumerating these items and figuring out what kinds of access levels they have on the network.

### **Single Sign-On**

If a new application server, a printer, an Exchange server, or anything else is provisioned in the business, DNS records can be created, service accounts can be created, they can integrate into LDAP, use Kerberos authentication. And in order to access these resources, the user's only need to remember one username and password.

From a security standpoint, this is both a blessing and a curse in that if credential is compromised, this credential can be used to sign into any service accessible to that user (e.g. pass the password and pass the hash attacks).   
  
Conversely, if the credential is compromised or the employee is offboarded, you only need to manage one credential.

**General Windows Protocols**

I mention these, because even though **Active Directory Domain Services (AD DS)** runs on Windows Server, it's still Windows at the end of the day, and still has support for these more insecure legacy protocols (which can be disabled, sometimes with consequences)

- **NetBIOS** ― A legacy protocol that allows hostname lookup and data transmission in a Local Area Network (LAN)
- **LLMNR** ― A legacy protocol that allows computers in a LAN to do hostname to IP address lookups in a functionally similar way to DNS
- **NetNTLM** ― Currently at NetNTLMv2, is a challenge-response authentication protocol that builds upon NTLM. Domain-joined computers will fall back to this protocol if Kerberos fails for any reason. Authenticates, does not authorize.  
    

**Some of the Core Active Directory Protocols and Services**

- **DNS** ― Required to manage hostnames and IP addresses of various services accessed by users and computers
- **DHCP** ― Ensures user computers and other registered systems can access the network  
    
- **AD CS** ― Manages certificates issued to users, applications, and computers   
    
- **SMB** ― Allows users to upload, download, and modify directories and files on a central server
- **LDAP/LDAPS** ― Allows users and applications to search and manage records in the Active Directory database  
    
- **Kerberos** ― Authenticates users and provides access to registered services and resources (does not authorize)
- **RPC** ― Facilitates communications between user applications and services
- **Group Policy** ― Centralized controls of user and computer account configurations
- Etc.

 

The key takeaway from this lesson is that for all of the numerous parts that comprise Active Directory, the core functionality is to manage access and share data between users and computers on a single domain or between domains.  
  
Instead of focusing on the attacks and tools for specific services and protocols that make up Active Directory, try thinking about how specific parts help meet the organization goal or identity and access management.