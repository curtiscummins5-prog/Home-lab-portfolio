# Windows AD Lab — curtis.org (home lab)

**Goal:** Build a lab Windows Active Directory environment and demonstrate domain services: AD DS, DNS, DHCP, client domain join, user/profile creation and Group Policy basics.

**Environment**
- Hypervisor: Proxmox (3-node cluster)
- VMs: Windows Server 2025 (Domain Controller), Windows 11 Pro (Client), Ubuntu (containers: Pi-Hole, Heimdall, Portainer), TrueNAS, OPNsense router
- Lab focus: Domain & AD exercises

## High-level steps performed
1. Deployed Windows Server VM on Proxmox and installed AD DS role.  
2. Promoted server to Domain Controller `curtis.org` and created forward lookup zone.  
3. Verified DNS SRV records and configured DNS to resolve AD clients.  
4. Joined Windows 11 client to domain; created domain user profile and basic GPOs (wallpaper, lock screen).  
5. Documented troubleshooting: DNS registration issues (Netlogon/DNS bindings), Group Policy application, and time/kerberos issues.

## Key skills demonstrated
- Active Directory (AD DS), AD Forest/domain design  
- DNS for AD (zones, SRV records, testing with nslookup)  
- Domain join process & troubleshooting (Netlogon, SRV records)  
- Group Policy (User vs Computer policies, GPO link, testing with gpresult)  
- PowerShell for AD management (AD module, ipconfig /registerdns, netlogon restart)  
- Virtualization (Proxmox VM sizing, networking), basic firewall/DHCP knowledge (OPNsense)

## Artifacts
- `diagram.png` — sanitized network diagram
- `checklists/join-checklist.md` — step-by-step join checklist
- `scripts/` — PowerShell snippets (no credentials)
- `screenshots/` — DNS Manager, ADUC, gpresult outputs (sanitized)

## How to reproduce (short)
1. Create Windows Server VM (3 vCPU / 8–12 GB RAM / 80 GB disk).  
2. Install AD DS and DNS roles, promote to DC for `curtis.org`.  
3. Ensure server DNS points to itself and has forward lookup zone `curtis.org`.  
4. On client, set DNS to DC IP, run `ipconfig /registerdns`, `net stop netlogon && net start netlogon` on server if SRV missing.  
5. Join client: `Add-Computer -DomainName curtis.org -Credential (Get-Credential) -Restart`.

## Learnings / notes
- The killer issues: missing SRV records and DNS binding to loopback only; fix by verifying DNS Interfaces and restarting Netlogon.  
- GPO timing: some policies require logoff/logon or gpupdate /force; lock screen (computer) policies require reboot.
