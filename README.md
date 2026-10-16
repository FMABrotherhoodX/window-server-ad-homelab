# Windows Server Active Directory Home Lab

## Project Overview
Built a simulated enterprise IT environment using VMware Workstation Pro 
and Windows Server 2022. Configured Active Directory from scratch, including 
domain controller setup, organizational unit design, and user and group 
management across geographic and departmental structures.

## Tools Used
- VMware Workstation Pro
- Windows Server 2022 Evaluation
- Active Directory Domain Services (AD DS)
- Windows 11 Enterprise
---

## Active Directory Setup & Organizational Structure

Set up a domain controller from scratch on Windows Server 2022 and built 
an organizational unit structure that mirrors how real enterprise environments 
organize their Active Directory.

### Screenshot 1 — OU Tree Structure
![OU Tree Structure](screenshots/ad-setup/01-ou-tree-structure.png)

The full organizational unit hierarchy built under the MH.Local domain. 
Geographic OUs (USA, Europe, Asia) each contain three sub-OUs — 
Computer, Server, and User — to separate different object types by department.

### Screenshot 2 — User OU: Groups and User Accounts
![User OU](screenshots/ad-setup/02-user-ou-groups.png)

Inside the USA User OU, showing the IT-Users security group, DL-ITAdmins 
distribution group, and three user accounts. Security groups are used for 
assigning permissions, while distribution groups are used for email lists.

### Screenshot 3 — Computer OU: Security Group and Placeholder PCs
![Computer OU](screenshots/ad-setup/03-computer-ou-group-pcs.png)

Inside the USA Computer OU showing the IT-Computer security group alongside 
placeholder computer objects PC01, PC02, and PC03 representing workstations 
assigned to the IT department.

### Screenshot 4 — Server OU: Security Group and Placeholder Servers
![Server OU](screenshots/ad-setup/04-server-ou-group-servers.png)

Inside the USA Server OU showing the IT-Server security group alongside 
placeholder server objects SRV01, SRV02, and SRV03 representing servers 
assigned to the IT department.

### Screenshot 5 — IT-Computer Group Members Tab
![Members Tab](screenshots/ad-setup/05-it-computer-members-tab.png)

IT-Computer group properties showing PC01, PC02, and PC03 confirmed as 
members of the security group. This proves the computer objects are not 
just sitting in the OU but are actively assigned to the correct group.

### What I Learned
Group names in Active Directory must be unique across the entire domain, 
not just within an OU. When creating matching IT groups across the User, 
Computer, and Server OUs, using a naming convention like IT-Users, 
IT-Computer, and IT-Server avoided conflicts and made the structure 
immediately readable. Security groups handle permissions and access while 
distribution groups handle email lists — understanding that distinction 
was key to setting up the right group type for each purpose.

---

## Group Policy Management

Created and configured six Group Policy Objects (GPOs) in the Group Policy Management Console (GPMC) to enforce security settings and manage user environments across the domain.

### Screenshot 1 — GPMC Overview
![GPMC Overview](screenshots/group-policy/06-gpmc-all-gpos-overview.png)

All six GPOs listed under the MH.Local domain in the Group Policy Management Console, including the bonus Account Lockout policy.

### Screenshot 2 — Password Policy Settings
![Password Policy](screenshots/group-policy/07-gpo-password-policy-settings.png)

Password policy configured with a 12-character minimum length, complexity requirements enabled, a 90-day maximum password age, and a 30-day minimum age. Applied under Computer Configuration since the computer enforces login requirements.

### Screenshot 3 — Drive Mapping Configuration
![Drive Mapping](screenshots/group-policy/08-gpo-drive-mapping-config.png)

Network drive mapped to E: pointing to \\servername\folder. Configured under User Configuration Preferences since users can modify mapped drives if needed.

### Screenshot 4 — Desktop Wallpaper Policy
![Desktop Wallpaper](screenshots/group-policy/09-gpo-desktop-wallpaper-settings.png)

Desktop wallpaper policy enabled under User Configuration Policies to enforce a consistent wallpaper across all users. Set as a Policy rather than a Preference so users cannot change it.

### Screenshot 5 — Restrict Control Panel
![Restrict Control Panel](screenshots/group-policy/10-gpo-restrict-control-panel.png)

Control panel access fully prohibited for all users. Configured under User Configuration Policies with Prohibit access to Control Panel and PC settings set to Enabled.

### Screenshot 6 — Disable USB Storage
![Disable USB Storage](screenshots/group-policy/11-gpo-disable-usb-storage.png)

All removable storage classes are denied access at the computer level. Configured under Computer Configuration Policies to prevent users from using any USB storage devices.

### Screenshot 7 — Account Lockout Policy (Bonus)
![Account Lockout](screenshots/group-policy/12-gpo-account-lockout-policy.png)

Account lockout configured with a threshold of 5 invalid logon attempts, a 30-minute lockout duration, and a 30-minute reset counter. This prevents brute force attacks on user accounts.

### What I Learned
Group Policy has two configuration types — Computer Configuration applies
settings to the machine regardless of who logs in, while User Configuration
applies settings to the user wherever they log in. Policies are enforced
and cannot be changed by users, while Preferences set defaults that users
can modify. Choosing the right combination of configuration type and setting
type is the key decision when creating any GPO in a real enterprise environment.

---

## GPO Testing & Computer Domain Join

Created a Windows 11 Enterprise VM, joined it to the MH.Local domain, applied Group Policy Objects to the client machine, and verified they were working. Ran into a real-world troubleshooting scenario when domain admin credentials were rejected — resolved it by promoting a domain user to Domain Admin through ADUC.

### Screenshot 1 — Server Static IP Configuration
![Server Static IP](screenshots/gpo-testing-domain-join/13-server-static-ip-configured.png)

Server configured with a static IP address. DNS set to the loopback address 127.0.0.1 pointing to itself as the domain controller, with Google DNS 8.8.8.8 as the secondary. Static IP is best practice for servers so the address does not change on reboot.

### Screenshot 2 — Client DNS Pointing to Server
![Client DNS](screenshots/gpo-testing-domain-join/14-client-dns-pointing-to-server.png)

Windows 11 Enterprise client configured to use the server's static IP as its primary DNS server. This allows the client to communicate with the domain controller and is required before joining the domain.

### Screenshot 3 — Client Joined to Domain
![Domain Join](screenshots/gpo-testing-domain-join/15-client-joined-to-domain.png)

System Properties on the Windows 11 client showing Comp01 successfully joined to the MH.Local domain. Full computer name reads Comp01.MH.Local confirming the domain join.

### Screenshot 4 — Computer Moved to Correct OU
![Computer in OU](screenshots/gpo-testing-domain-join/16-aduc-computer-moved-to-ou.png)

After joining the domain the computer automatically landed in the default Computers container. Moved it manually to USA > Computer OU in ADUC so the correct GPOs would apply to it. Best practice is to always move newly joined computers to their appropriate OU.

### Screenshot 5 — GPOs Linked to OUs
![GPOs Linked](screenshots/gpo-testing-domain-join/17-gpmc-gpos-linked-to-ous.png)

GPMC showing Restrict Control Panel linked to MH.Local/USA/User with Link Enabled confirmed. Computer OU has Disable USB Devices and Password Policy linked. User OU has Desktop Wallpaper, Drive Mapping, and Restrict Control Panel linked.

### Screenshot 6 — gpupdate /force
![gpupdate force](screenshots/gpo-testing-domain-join/18-gpupdate-force-complete.png)

Ran gpupdate /force on the client to apply GPO settings immediately instead of waiting the default 90 minute refresh cycle. Both Computer Policy and User Policy updated successfully.

### Screenshot 7 — GPO Blocking Control Panel
![Control Panel Blocked](screenshots/gpo-testing-domain-join/19-gpo-control-panel-blocked.png)

Attempted to open Control Panel on the client after forcing the GPO update. Restrictions error message confirms the policy applied successfully — access cancelled and user directed to contact system administrator.

### What I Learned
When a computer joins a domain it lands in the default Computers container, not the custom OUs. Moving it to the correct OU is a required step before GPOs will apply to it. GPOs do not apply instantly — they refresh every 90 minutes by default. In a real environment gpupdate /force is used to push policy changes immediately. Running into credential issues with the built-in Administrator account was also a useful real-world scenario — promoting a domain user to Domain Admin through ADUC is the practical fix when the built-in account password is unknown.

---

## Upcoming
- File Services and Network Sharing
