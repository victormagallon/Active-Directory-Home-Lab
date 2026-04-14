
# Windows Active Directory Home Lab

## Overview
This document covers the Active Directory home lab built following the 
East Charmer YouTube series. The goal was to simulate a basic company 
network by setting up a Windows Server, configuring Active Directory, 
applying Group Policies, and connecting a client computer to the domain. 
The lab was built using VMware Workstation Pro, Windows Server 2022 as 
the server, and Windows 11 Pro as the client machine.

Based on the East Charmer YouTube Series-(https://www.youtube.com/playlist?list=PLAdEnQWAAbfXMY2D4HVZOe-ChfTKmaJfQ)

---

## Tools Used
- VMware Workstation Pro
- Windows Server 2022
- Windows 11 Pro

---

## Steps Completed

### 1. Setting Up the Windows Server Virtual Machine
Windows Server 2022 was installed inside a virtual machine on VMware 
Workstation Pro. Before anything else, the server was given a proper 
name so it would be easily identified on the network. Windows Updates 
were then run and fully completed before moving on, since updates require 
reboots that could interrupt later steps. Remote management was also 
turned on to allow the server to be managed from other machines.

---

### 2. Installing Active Directory Domain Services
The Active Directory Domain Services (AD DS) role was installed via 
Server Manager by adding it as a new role. Once installed, the server 
was promoted to a Domain Controller by creating a new forest with the 
domain name `viccyber.local`. The `.local` ending is standard for lab 
environments to avoid conflicts with real websites. After an automatic 
reboot, the domain name appeared on the login screen, confirming 
everything worked. The presence of tools like Active Directory Users 
and Computers (ADUC) and the Group Policy Management Console (GPMC) 
in the Admin Tools menu confirmed a successful install.

---

### 3. Configuring a Static IP Address
The Domain Controller was assigned a static IP address. This is required 
because all client machines need to reach the server at the same address 
every time — if the IP changes, the network breaks. The static IP was 
set manually in the NIC's TCP/IPv4 settings. The Primary DNS was pointed 
to the server itself, and the Alternate DNS was set to `8.8.8.8` 
(Google) as a backup.

---

### 4. Building the Organizational Structure
Organizational Units (OUs) were created inside Active Directory Users 
and Computers to organize the domain like a real company. Think of OUs 
as folders, each department (IT, Sales, HR, Finance) gets its own 
folder to store its users, computers, and policies. User accounts and 
groups were then created inside each OU. An important distinction was 
learned here: a **Security group** is used to control access to files 
and resources, while a **Distribution group** is only used for email 
lists and cannot assign permissions.

["OUs"](https://github.com/user-attachments/assets/f9fb263b-10f2-44eb-b9fc-d21a2cc0c628)

["Groups"](https://github.com/user-attachments/assets/ee1dad30-ae3a-4ded-bce7-d72cd55a99a0)


---

### 5. Creating and Linking Group Policy Objects
Group Policy Objects (GPOs) were created in the Group Policy Management 
Console to enforce rules across the domain. Policies created and tested 
included restricting the Control Panel, enforcing password rules, mapping 
a shared network drive at login, disabling USB devices, and setting a 
standard desktop wallpaper. Each GPO was linked to its target OU by 
dragging it onto the OU in the GPMC, which activates the policy for 
everyone inside that OU.

---

### 6. Configuring Password Policy
A Password Policy GPO was configured with the following settings:
- Minimum password length: **12 characters**
- Password complexity requirements: **Enabled**
- Password history: **4 passwords remembered**
- Maximum password age: **180 days**

["Password Policy"](https://github.com/user-attachments/assets/b8549525-a398-4ea7-8b1d-3d3a3948662a)

---

### 7. Configuring Account Lockout Policy
An Account Lockout Policy was configured to lock a user account after 
**4 invalid login attempts**. The lockout duration and reset counter 
were both set to **30 minutes**. This was tested by intentionally 
entering wrong passwords on the Windows 11 client until the account 
locked out.

["Lockout Policy"](https://github.com/user-attachments/assets/76f730a6-46e5-4c93-98fb-d109853044bc)


---

### 8. Testing Account Lockout on the Client Machine
After exceeding the invalid login threshold, the domain account was 
successfully locked out on the Windows 11 client. The account was then 
unlocked from the server through ADUC by checking the **"Unlock account"** 
box on the Account tab of the user's properties.

["Account Locked"](https://github.com/user-attachments/assets/d7c1219b-9066-4904-ab45-80b5184724b5)

["Unlocking Account"](https://github.com/user-attachments/assets/ee215f9b-e647-4fb7-a1d7-d439034d1f93)


---

### 9. Configuring Drive Mapping via GPO
A Drive Mapping GPO was created under:
`User Configuration > Preferences > Windows Settings > Drive Maps`

The shared folder on the server was mapped to drive letter **S:** and 
set to update automatically on every login.

---

### 10. Confirming the Mapped Drive on the Client Machine
After logging into the domain on the Windows 11 client and running 
`gpupdate /force`, the mapped network drive (S:) appeared automatically 
under Network Locations in File Explorer, confirming the GPO was applied 
successfully.

["Client Mapped Drive"](https://github.com/user-attachments/assets/bf4e9f3a-e00d-4977-a54c-51003313106e)


---

### 11. Configuring File Server Resource Manager (FSRM)
The File Server Resource Manager (FSRM) role was installed to manage 
the shared folder. A storage quota of **100 MB (Hard limit)** was 
configured on the SHARED folder path using the built-in source template. 
This prevents any single user from filling up the shared drive.

["FSRM"](https://github.com/user-attachments/assets/2571a784-3d57-49d1-aea8-a30212e66925)


---

### 12. Configuring File Screening
File screening was configured on the SHARED folder to actively block 
**Audio and Video Files** and **Image Files** using the SHARED source 
template. This prevents users from storing media files in the shared 
directory.

["Blocked Files"](https://github.com/user-attachments/assets/6c4aebab-2e0d-4bba-809f-6da4ae3c2f16)


---

### 13. Configuring Fine-Grained Password Policies
Two separate password policies were created using the Active Directory 
Administrative Center:
- **AdminPasswordPolicy** — Precedence 1 (stricter rules, applies to admins)
- **UserPasswordPolicy** — Precedence 2 (standard rules, applies to regular users)

Fine-grained policies allow different password rules for different groups 
of users, unlike a standard domain-wide GPO password policy which applies 
to everyone equally.


["Password Policy"](https://github.com/user-attachments/assets/66bb6bd3-fc47-46f4-bcb8-31468ef8757b)


---

### 14. Setting Up the Windows 11 Client Machine
A Windows 11 Pro virtual machine was created to act as a regular employee 
workstation. Windows 11 Home cannot join a domain, so the Pro edition is 
required. During setup, the **"Domain join instead"** option was selected 
to skip Microsoft's account screen. The client's DNS was manually pointed 
to the Domain Controller's static IP so it could find the domain. 
Connectivity was confirmed by running `ping` and `nslookup viccyber.local` 
in the Command Prompt.

---

### 15. Joining the Domain and Testing Policies
The Windows 11 machine was joined to `viccyber.local` via:
**System Properties > Change > Domain**

After entering the domain name and admin credentials, the machine rebooted 
and was successfully logged into using a domain account. On the server, 
the new computer appeared in ADUC and was moved into the correct OU so 
its policies would apply. The command `gpupdate /force` was run on the 
client to immediately apply all GPOs. The Control Panel restriction 
policy was confirmed working when access was blocked on the client machine.

["Domain Joined"](https://github.com/user-attachments/assets/78a51bf1-852a-4f04-b34b-9f3f4820d6c2)





---

### 16. Enabling Access-Based Enumeration (ABE)
Access-Based Enumeration (ABE) was enabled on the server's shared folders. 
Without ABE, users browsing a shared drive can see all folders — even 
ones they cannot open. With ABE on, users can only see folders they 
actually have access to, making shared drives cleaner and more secure. 
ABE was turned on through:
**Server Manager > File and Storage Services > Shares > Share Properties > Settings**

Logged into the Windows 11 client as an HR user. When browsing the 
DeptShares folder, only the HR folder was visible — all other department 
folders were completely hidden, confirming ABE was working correctly.

["Client Machine-ABE enabled, HR Only"](https://github.com/user-attachments/assets/851d5f14-90f5-43f9-8ae6-9f0e572db682)


---

## Conclusion
This home lab covered the core skills used in real enterprise IT 
environments, deploying a Domain Controller, organizing users with OUs, 
enforcing policies with GPOs, and managing file share permissions. These 
are foundational skills directly relevant to careers in IT administration 
and cybersecurity.
