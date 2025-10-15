# Windows AD Lab — curtis.org (home lab)

**Goal:** Build a lab Windows Active Directory environment and demonstrate domain services: AD DS, DNS, DHCP, client domain join, user/profile creation and Group Policy basics.


## Table of Contents
- [Environment](#environment)  
- [High-level steps performed](#high-level-steps-performed)  
- [Key skills demonstrated](#key-skills-demonstrated)  
- [Artifacts & Evidence](#artifacts--evidence)  


## Environment
- Hypervisor: Proxmox (3-node cluster)  
- VMs: Windows Server 2025 (Domain Controller), Windows 11 Pro (Client), Ubuntu (containers: Pi-Hole, Heimdall, Portainer), TrueNAS, OPNsense router  
- Lab focus: Domain & AD exercises

## High-level steps performed
1. Deployed Windows Server VM on Proxmox and installed AD DS role.  
2. Promoted server to Domain Controller `curtis.org` and created forward lookup zone.  
3. Verified DNS SRV records and configured DNS to resolve AD clients.  
4. Joined Windows 11 client to domain; created domain user profile and basic GPOs (wallpaper, lock screen).  
5. Documented troubleshooting: DNS registration issues (Netlogon/DNS bindings), Group Policy application, and time/Kerberos issues.

## Key skills demonstrated
- Active Directory (AD DS), AD Forest/domain design  
- DNS for AD (zones, SRV records, testing with nslookup)  
- Domain join process & troubleshooting (Netlogon, SRV records)  
- Group Policy (User vs Computer policies, GPO linking, testing with gpresult)  
- PowerShell for AD management (AD module, `ipconfig /registerdns`, netlogon restart)  
- Virtualization (Proxmox VM sizing, networking), basic firewall/DHCP knowledge (OPNsense)


## Artifacts & Evidence
All artifacts are in the repo under `projects/windows-ad-lab/`

- `diagram.png` — sanitized network diagram (topology & IPs)  
- `checklists/join-checklist.md` — step-by-step domain-join checklist (copyable)   
- `screenshots/`
- Other screenshots in that folder (03, 05, 06, 08, 09, 10) show DNS settings, GPO linking, gpresult outputs and Server Manager AD DS role.




