# Domain join checklist — curtis.org

Purpose
Quick checklist to join a Windows client to the curtis.org domain and validate the join. Keep this at-hand when repeating the lab.

Prerequisites
- Domain Controller (DC) up and reachable (example IP: 192.168.1.166).
- DNS running on the DC (AD DS + DNS installed).
- Client and DC on same L2 network or routable network path.
- Local admin on the client and Domain Admin credentials for the join.

Steps

1) Basic network checks (on client)
   - Ensure client can ping the DC:
     `ping 192.168.1.166`
   - Confirm the client’s DNS is set to the DC IP (important):
     - Windows: Settings → Network → Adapter → IPv4 → DNS = `192.168.1.166`
     - Or run: `ipconfig /all` and check “DNS Servers”

2) Verify AD DNS registration (optional but fast)
   - From the client:
     `nslookup -type=SRV _ldap._tcp.curtis.org`
   - Expected: SRV record(s) pointing to the DC hostname and IP.

3) Server-side quick checks (on DC)
   - Open DNS Manager → Forward Lookup Zones → `curtis.org` → confirm host (A) records and _msdcs entries exist.
   - DNS Server Properties → Interfaces: confirm DNS is listening on the DC IP (not only loopback).

4) If SRV/records are missing
   - On DC: `net stop netlogon` then `net start netlogon`
   - On DC: `ipconfig /registerdns`
   - Wait 30–60s and re-check `nslookup -type=SRV ...` from client.

5) Join the domain (pick one)

   GUI:
   - Settings → Accounts → Access work or school → Join a domain → enter `curtis.org`
   - Provide domain admin credentials when prompted and accept restart.

   PowerShell (as local admin on client):
   ```powershell
   $cred = Get-Credential    # enter DOMAIN\Administrator
   Add-Computer -DomainName curtis.org -Credential $cred -Restart
   
6 ) Post-join validation (after restart)
    On client: sign in using a domain account (use curtis\youruser).
    From client run:
    whoami (should show curtis\<username>)
    nltest /dsgetdc:curtis.org (discovers DC)
    nslookup curtis.org (resolves DC IP)
    gpupdate /force then gpresult /r to confirm GPOs

7 ) Common fixes if join fails
    Ensure client DNS points only to DC (not to external DNS). Fix and retry join.
    If credentials rejected: ensure clock skew is < 5 minutes between client and DC (fix with w32tm /resync).
    If SRV not found: restart Netlogon and register DNS (steps in #4).
    Check Event Viewer (System and Directory Service logs) on the DC and client for detailed errors.

 8 ) Optional: remove a computer (cleanup)
     From ADUC on DC: Computers → right-click machine → Delete
     On client if still reachable: Remove-Computer -UnjoinDomaincredential (Get-Credential) -Restart   

    
