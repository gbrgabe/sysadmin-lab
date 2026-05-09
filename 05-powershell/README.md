# 05 - PowerShell Reference

Quick reference for PowerShell commands used in this lab for server administration, Active Directory management and help desk tasks.

---

## Active Directory

### User Management

```powershell
# Create user
New-ADUser -Name "Name" -SamAccountName username -Enabled $true

# Reset password
Set-ADAccountPassword -Identity username -NewPassword (ConvertTo-SecureString "Password123!" -AsPlainText -Force)

# Unlock account
Unlock-ADAccount -Identity username

# Disable account
Disable-ADAccount -Identity username

# Enable account
Enable-ADAccount -Identity username

# List all users
Get-ADUser -Filter * | Select Name, SamAccountName, Enabled

# Find a specific user
Get-ADUser -Identity username

# Remove user
Remove-ADUser -Identity username
```

### Groups

```powershell
# Add user to group
Add-ADGroupMember -Identity "GRP-Finance" -Members username

# Remove user from group
Remove-ADGroupMember -Identity "GRP-Finance" -Members username

# List group members
Get-ADGroupMember -Identity "GRP-Finance"
```

### Computers

```powershell
# List all computers
Get-ADComputer -Filter * | Select Name

# Find a specific computer
Get-ADComputer -Identity PC01
```

### Domain Controllers

```powershell
# List Domain Controllers
Get-ADDomainController

# Resolve a DNS name
Resolve-DnsName office.lab

# Get DNS server info
Get-DnsServer
```

---

## Server Administration

### Services

```powershell
# Check service status
Get-Service ntds

# Start a service
Start-Service mysql

# Stop a service
Stop-Service mysql

# Restart a service
Restart-Service mysql

# List all running services
Get-Service | Where-Object {$_.Status -eq "Running"}
```

### Network

```powershell
# Check IP configuration
ipconfig /all

# Set static IP
New-NetIPAddress -InterfaceAlias "Ethernet" -IPAddress 10.0.0.1 -PrefixLength 24

# Set DNS server
Set-DnsClientServerAddress -InterfaceAlias "Ethernet" -ServerAddresses 10.0.0.1

# List network adapters
Get-NetAdapter

# Test connectivity
ping 10.0.0.1
Test-Connection -ComputerName 10.0.0.1
```

### System

```powershell
# Check Windows version
Get-ComputerInfo | Select WindowsProductName, WindowsVersion

# Rename computer
Rename-Computer -NewName "DC01" -Restart

# Check disk space
Get-PSDrive -PSProvider FileSystem

# List installed features
Get-WindowsFeature | Where-Object {$_.InstallState -eq "Installed"}

# Install a Windows feature
Install-WindowsFeature -Name "feature-name" -IncludeManagementTools
```

---

## Help Desk Tasks

```powershell
# Check if a computer is reachable
Test-Connection -ComputerName 10.0.0.5 -Count 2

# Clear DNS cache
Clear-DnsClientCache

# Check DNS cache
Get-DnsClientCache

# Check CPU usage (top 10 processes)
Get-Process | Sort-Object CPU -Descending | Select -First 10

# Check memory usage
Get-CimInstance Win32_OperatingSystem | Select FreePhysicalMemory, TotalVisibleMemorySize

# Force Group Policy update
gpupdate /force

# Check logged on user
query user

# Check running processes
Get-Process

# Kill a process
Stop-Process -Name "processname"

# Check event logs (last 10 errors)
Get-EventLog -LogName System -EntryType Error -Newest 10

# Check login events (last 10)
Get-EventLog -LogName Security -InstanceId 4624 -Newest 10

# Check last boot time
(Get-CimInstance Win32_OperatingSystem).LastBootUpTime

# Check system uptime
(Get-Date) - (Get-CimInstance Win32_OperatingSystem).LastBootUpTime
```

---

## RSAT Tools

```powershell
# Install Active Directory management tools
Add-WindowsCapability -Online -Name Rsat.ActiveDirectory.DS-LDS.Tools~~~~0.0.1.0
```
