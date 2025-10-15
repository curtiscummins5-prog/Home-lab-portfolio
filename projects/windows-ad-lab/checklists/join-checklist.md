# Domain join checklist — curtis.org

Purpose: Quick lab notes to join a Windows client to the curtis.org domain and validate the join.

Prereqs
- Domain Controller (DC) up and reachable (example IP: 192.168.1.166)
- DNS role installed on the DC (AD DS + DNS)
- Client and DC on same L2 or routable network
- Local admin on the client and domain admin credentials

1) Basic network checks (on client)
- Ping the DC:
  ping 192.168.1.166
- Verify DNS is set to the DC (Settings → Network → Adapter → IPv4) or:
  ipconfig /all

2) Verify AD DNS registration (optional)
- From the client:
  nslookup -type=SRV _ldap._tcp.curtis.org
- Expected: SRV records pointing to the DC hostname/IP

3) Server-side quick checks (on the DC)
- DNS Manager → Forward Lookup Zones → curtis.org — confirm A records and _msdcs entries
- DNS Manager → Server → Properties → Interfaces — confirm DNS listens on DC IP (not only loopback)
- Optional command on DC:
  netstat -an | findstr ":53"

4) If SRV/records are missing (run on DC)
- Run:
  net stop netlogon
  net start netlogon
  ipconfig /registerdns
- Wait 30–60s, then re-check from client:
  nslookup -type=SRV _ldap._tcp.curtis.org

5) Join the domain
- GUI: Settings → Accounts → Access work or school → Join a domain → enter curtis.org → provide domain admin credentials → restart
- PowerShell (run as local admin on client):
  $cred = Get-Credential   # enter curtis\Administrator
  Add-Computer -DomainName curtis.org -Credential $cred -Restart

6) Post-join validation (after restart, on the client)
- Run:
  whoami
  whoami /fqdn
  nltest /dsgetdc:curtis.org
  nslookup curtis.org
  nslookup -type=SRV _ldap._tcp.curtis.org
  gpupdate /force
  gpresult /r
  # optional HTML report:
  gpresult /h C:\Temp\gpresult.html
  w32tm /query /status

7) Common fixes if join fails
- Ensure client DNS points only to the DC. See and set:
  Get-DnsClientServerAddress
  Set-DnsClientServerAddress -InterfaceAlias "<INTERFACE>" -ServerAddresses "192.168.1.166"
- Fix clock skew:
  w32tm /resync
  w32tm /query /status
- Re-run Netlogon and DNS registration on DC (see step 4)
  net stop netlogon
  net start netlogon
  ipconfig /registerdns
- Check Event Viewer on both client and DC for detailed errors
- Temporary firewall debug (remove rules after test):
  New-NetFirewallRule -DisplayName "Allow LDAP TCP 389" -Direction Inbound -Protocol TCP -LocalPort 389 -Action Allow -Profile Domain
  New-NetFirewallRule -DisplayName "Allow DNS UDP 53" -Direction Inbound -Protocol UDP -LocalPort 53 -Action Allow -Profile Domain
  # Remove after test:
  # Remove-NetFirewallRule -DisplayName "Allow LDAP TCP 389"
  # Remove-NetFirewallRule -DisplayName "Allow DNS UDP 53"

8) Optional: cleanup / unjoin
- From ADUC (GUI): Computers → right-click machine → Delete
- From client (unjoin):
  $cred = Get-Credential
  Remove-Computer -UnjoinDomainCredential $cred -PassThru -Verbose -Restart
- From DC / AD PowerShell:
  Remove-ADComputer -Identity "<COMPUTER_NAME>"
  # or disable instead of delete:
  Disable-ADAccount -Identity "<COMPUTER_NAME>"



    
