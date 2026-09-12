# Active Directory Helpdesk Simulation Lab

## Overview
This project simulates a small company's IT environment using a homelab built in Oracle VirtualBox. It includes a Windows Server 2025 domain controller, Active Directory with 4 departments and 25 simulated employees, and documented helpdesk troubleshooting.

## Environment
- Hypervisor: Oracle VirtualBox
- Domain Controller: Windows Server 2025 Standard (Desktop Experience)
- Client Machines: Windows 11
- Domain Name: copr.local
- Network: Host-only network, 192.168.56.0/24

## Server Roles

First thing I did was set up the actual roles the server needed — Active Directory Domain Services, DNS, DHCP, and Print and Document Services. Installed all of these through Server Manager before doing anything else, since everything after this depends on these being in place.

![Server roles installed](<Server Roles Dashboard3.png>)

## DHCP Setup

Before clients could get an IP address automatically, I had to authorize the DHCP server and set up a scope. I authorized it in the DHCP console, then created a scope covering 192.168.56.100–192.168.56.200 for the client range, with DNS pointing back to the DC (192.168.56.10) so any machine getting an IP would also know where to find Active Directory and name resolution.

![DHCP scope active](<DHCP Scope.png>)

## Domain Promotion

Promoted the server to a domain controller and created a new forest. Meant to name it corp.local but ended up typing it as copr.local by mistake — didn't notice until later when a bunch of PowerShell commands started failing because they were pointed at a domain that didn't actually exist. Rather than redo the whole promotion, I just kept copr.local and adjusted everything else to match it going forward.

## Organizational Units

Set up 5 OUs to organize the company: Sales, IT, Finance, HR, and a Disabled Users OU for anyone who gets offboarded later. Kept it flat — no sub-folders inside each department — just simple and easy to manage for this size of company.

![OU structure](<OU Structure.png>)

## Bulk User Creation

Instead of manually creating 25 employee accounts one at a time, I built a CSV with everyone's name, department, and job title, then wrote a PowerShell script that loops through the CSV and creates each user with New-ADUser, dropping them into the right department OU automatically. Ran it, checked the count afterward — 27 total accounts, which checks out since that's my 25 plus a couple of built-in system accounts every domain starts with.

![CSV file used for bulk creation](<CSV FILE.png>)
![Bulk creation script](<Bulk Membership Confirmed.png>)
![User count verified](<Bulk user Creation.png>)
![Sales users populated](<Sales Users Populated.png>)

## Security Groups

Created one security group per department — SG-Sales, SG-IT, SG-Finance, SG-HR — then used PowerShell to grab everyone in each department's OU and add them to the matching group automatically, instead of adding 25 people one by one. Had to redo the group creation once since they didn't actually save the first time through the GUI, but PowerShell fixed that cleanly.

![Security groups created](<Security Groups Created.png>)
![Group membership confirmed](<Group Membership.png>)

## Shared Folders & Permissions

Created a shared folder for each department and locked down access using NTFS permissions tied to each department's security group, so only the right people can get into their own department's folder.

To actually prove this worked instead of just trusting the config, I tested it using runas /netonly to run a command prompt authenticating as a Sales employee (jsmith) while staying logged in locally as Administrator — this gets around the fact that regular employees can't log directly into a domain controller. From there, I tried opening both folders:

- \\DC01\Sales opened fine, since jsmith is in the SG-Sales group
- \\DC01\Finance was denied, since jsmith isn't in SG-Finance

Confirmed the whole setup works end to end — OU membership, group membership, and folder permissions are all doing their job correctly.

![Access test results](<Verify Department Folder Permissions.png>)
