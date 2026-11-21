# Active Directory (AD) Home Lab

## Goal
Build a small Windows domain from scratch, join a client to it, enforce a password and lockout policy, and practice a common help desk and security task: investigating and fixing a locked account.

## Environment
| Machine | Role | OS | IP address |
|---|---|---|---|
| DomainController01 | Domain controller (AD DS, DNS) | Windows Server 2022 Standard Evaluation | 192.168.10.10 |
| CLIENT01 | Domain-joined workstation | Windows 11 Enterprise Evaluation | 192.168.10.20 |

- **Hypervisor:** VirtualBox
- **Domain:** lab.local
- **Network:** both VMs (virtual machines) share a private Internal Network named `labnet` (192.168.10.0/24), separate from my home network. A second NAT (Network Address Translation) adapter gives each VM internet access.

## Tools Used
VirtualBox, Windows Server 2022, Windows 11 Enterprise, Active Directory Users and Computers, Group Policy Management, Event Viewer

## Steps Taken

### 1. Built the server and set a static IP
Installed Windows Server 2022 (Desktop Experience) in VirtualBox, then set a static IP of 192.168.10.10 on the internal adapter, with DNS (Domain Name System) pointed at itself.

![Server Manager after install](screenshots/01-server-installed.png)
![Static IP configuration](screenshots/02-static-ip.png)

### 2. Created the domain
Installed the AD DS (Active Directory Domain Services) role and promoted the server to a domain controller for a new forest, `lab.local`.

![Domain controller ready](screenshots/03-domain-controller.png)

### 3. Created OUs, users, and groups
Created two OUs (organizational units), IT and Sales, each with one user and one security group, and added each user to their group.

![IT OU](screenshots/04-ous-users-groups-it.png)
![Sales OU](screenshots/04-ous-users-groups-sales.png)
![IT-Staff members](screenshots/05-it-staff-members.png)

### 4. Set password and lockout policies
Edited the Default Domain Policy in Group Policy Management:
- Minimum password length: 12 characters, complexity required
- Account lockout: 3 invalid attempts, locked for 15 minutes

![Password policy](screenshots/06-password-policy.png)
![Lockout policy](screenshots/07-lockout-policy.png)
![net accounts output](screenshots/08-net-accounts.png)

### 5. Built the client and joined the domain
Created a Windows 11 Enterprise VM with a static IP of 192.168.10.20 and DNS pointed at the domain controller. Confirmed connectivity, joined it to `lab.local`, and signed in as a domain user.

![Connectivity test](screenshots/11-ping-test.png)
![Signed in as a domain user](screenshots/09-client-joined.png)
![CLIENT01 in Active Directory](screenshots/10-client-in-ad.png)

### 6. Locked out an account, investigated, and fixed it
- Entered 3 wrong passwords for a test user to trigger the lockout policy.
- Found the lockout in the Security log on the domain controller (Event ID 4740, "a user account was locked out"), which showed the account and the source computer.
- Unlocked the account, reset the password, and forced a change at next logon.
- Confirmed the user could sign in again.

![Account locked out](screenshots/12-account-locked.png)
![Event ID 4740](screenshots/13-event-4740.png)
![Account tab, locked](screenshots/14-account-tab-locked.png)
![Sign-in after reset](screenshots/15-password-reset-signin.png)

## Problems and Fixes
- **Static IP set on the wrong adapter.** On the client, Windows numbered the network adapters in the opposite order from the server, so my static address landed on the NAT adapter. The client couldn't reach the server (ping returned "Destination host unreachable"). I identified the internal adapter by its 169.254.x.x address, set the NAT adapter back to automatic, and applied the static IP to the correct one. The ping and DNS lookup then worked.
- **Computer name length.** I named the server `DomainController01`, which is longer than 15 characters, so Windows shortens its older NetBIOS (Network Basic Input/Output System) name. It didn't affect the domain.

## What I Learned
- How a domain controller, DNS, and a domain-joined client depend on each other.
- Why the client has to use the domain controller for DNS.
- How a lockout policy works and where to find lockout events (Event ID 4740) in the Security log.
- The standard steps for unlocking an account and resetting a password.

## Next Steps
Add a Wazuh SIEM (Security Information and Event Management) server and connect this Windows client to it, so failed logons and lockouts like these generate alerts.
