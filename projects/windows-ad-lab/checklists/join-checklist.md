# Join checklist — client → curtis.org

**Preconditions**
- DC IP: 192.168.1.166 (example)
- Client running Windows 11 Pro
- DC DNS service running and zone curtis.org exists

## On the DC (run as Administrator)
1. Confirm DNS service:
   - `Get-Service dns` → Status: Running
   - DNS Manager → curtis.org zone exists → A record points to 192.168.1.166
2. Force registration:
   - `ipconfig /flushdns`
   - `ipconfig /registerdns`
   - `net stop netlogon && net start netlogon`
3. Confirm SRV records exist in DNS Manager under `_msdcs` and `_tcp` (e.g. _ldap._tcp).

## On the client (Windows 11 Pro)
1. Set DNS to the DC only:
   - Settings → Network & internet → Adapter → IPv4 → Preferred DNS: `192.168.1.166`
2. Flush and test:
   - `ipconfig /flushdns`
   - `nslookup curtis.org` → should return 192.168.1.166
   - `nslookup -type=SRV _ldap._tcp.curtis.org` → should list SRV pointing to DC
3. Test ports:
   - PowerShell: `Test-NetConnection -ComputerName 192.168.1.166 -Port 389`
   - Ensure TcpTestSucceeded : True
4. Join domain:
   - GUI: Settings → Accounts → Access work or school → Join a domain → enter `curtis.org`
   - Or PowerShell (Admin): `Add-Computer -DomainName "curtis.org" -Credential (Get-Credential) -Restart`
5. After reboot, log in using domain user: `curtis\lab_user1`. Verify policies:
   - `gpupdate /force`
   - `gpresult /r` or `gpresult /h C:\temp\gpo.html`

## Troubleshooting quick hits
- If nslookup returns server but times out: ensure DNS service listening on 0.0.0.0:53 (run `netstat -an | findstr ":53"` on DC) and firewall allows UDP/TCP 53.  
- If SRV missing: re-run netlogon restart and `ipconfig /registerdns`.  
- If Kerberos/Time errors: sync clocks (`w32tm /resync`).

