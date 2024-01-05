## **Active Directory Pentest Types**

Given the massive attack surface that Active Directory presents, and the fact that I only have an hour or so to present, **the objective** of the material here is to keep the information **within the scope of the PJPT and PNPT**. I could never even begin to cover the entire attack surface.

### **External**

- Emulates a threat that attempts to penetrate and own the Active Directory domain via a public facing service, such as web mail, VPN, web applications, etc.
- Usually involves a heavy OSINT campaign to pre-emptively collect usernames and passwords
    - Often found in historical breach dumps
    - Employees may re-use passwords, so having a password for an employee for another service may be helpful
- Also, involves heavy enumeration of the target organization's current and historical DNS records
    - SecurityTrails has an excellent tool for current and archived DNS records
    - Plot out the company's externally facing resources
- If there are login points on the externally facing resources, it's worth a shot at spraying some credentials of any usernames and/or passwords discovered during the OSINT phase

### **Internal**

- Emulates a threat that has breached an organization and is already working from the inside
    - Could be via physical breach where a rogue device is planted on the network
    - Could be via a phishing campaign where backdoor is placed on an employee device
- The penetration test could be conducted as a greybox pentest, where a credential is provided
    - Or, as a black box penetration test where the breach has occurred, but no credentials are yet known
    - Greybox would allow for more immediate enumeration of the domain
    - Blackbox would require the tester take additional steps to attempt to acquire credentials (and attempt to crack them)

## **Post-Compromise Cycle**

![[Pasted image 20240104191929.png]]
%%  %%_**Note:** "Mission Accomplished" is based on the parameters of the penetration test_
