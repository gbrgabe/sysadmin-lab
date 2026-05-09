# 01 - Infrastructure & VirtualBox Setup

This section covers the full setup of the lab environment, from downloading the ISOs to having both VMs configured and joined to the domain.

All downloads from official sources: VirtualBox from virtualbox.org and both ISOs from Microsoft's official website.

---

## Step 1 - Resource Planning

Before creating any VM, I planned the resource allocation based on the host machine specs (16GB RAM, 1TB SSD + 1TB HDD).

| Machine | Role | CPU | RAM | Disk |
|---|---|---|---|---|
| DC01 | Domain Controller | 2 cores | 2.5GB | 100GB dynamic |
| PC01 | Client | 2 cores | 7GB | 130GB dynamic |

> 📝 Note: I used dynamic disks so the files on the host only grow as needed, instead of allocating the full size upfront.

---

## Step 2 - Create VM for Windows Server 2022 Core

- Name: WS22
- RAM: 2.5GB
- CPU: 2 cores
- Disk: 100GB dynamic
- Adapter 1: Internal Network (adlab)
- Adapter 2: NAT

> 📝 Note: I configured two network adapters on the server. Adapter 1 for the Internal Network used in the lab, and Adapter 2 as NAT for internet access during initial setup and updates.

---

## Step 3 - Create VM for Windows 11

- Name: PC01
- RAM: 7GB
- CPU: 2 cores
- Disk: 130GB dynamic
- Network: NAT (temporary, for initial installation only)
- Secure Boot: enabled

> 📝 Note: PC01 started on NAT for the Windows installation and driver updates. Without internet access during this stage, drivers fail to update properly. Once Windows was installed and updated, I switched the network adapter back to Internal Network (adlab), the same network as DC01.

---

## Step 4 - Deploy Windows Server 2022 Core

Installed Windows Server 2022 without GUI (Core).

### Identify network interfaces

Since the server has two adapters (Internal Network and NAT), I first identified which interface was which:

```powershell
Get-NetAdapter
```

### Configure static IP

```powershell
New-NetIPAddress -InterfaceAlias "Ethernet" -IPAddress 10.0.0.1 -PrefixLength 24
```

### Configure DNS server

```powershell
Set-DnsClientServerAddress -InterfaceAlias "Ethernet" -ServerAddresses 10.0.0.1
```

> ⚠️ Setting the DNS via PowerShell never worked consistently, the server kept ignoring it and returning a different preferred DNS. The fix was to always set the DNS through SConfig (option 8 - Network Settings) after setting the static IP via PowerShell. After that, `ipconfig /all` returns the correct values.

### Install AD DS

```powershell
Install-WindowsFeature AD-Domain-Services -IncludeManagementTools
```

### Create new forest

```powershell
Install-ADDSForest -DomainName "office.lab" -InstallDns
```

### Verify everything is working

```powershell
ipconfig /all
Get-ADDomain
Get-Service ntds
nslookup office.lab
nslookup 10.0.0.1
```

> 📝 Note: `Get-Service ntds` should return `Running`. If not, AD DS did not install correctly.

---

## Step 5 - Join PC01 to the Domain

On PC01, after switching to Internal Network (adlab):

### Configure static IP

```powershell
New-NetIPAddress -InterfaceAlias "Ethernet" -IPAddress 10.0.0.15 -PrefixLength 24
```

### Point DNS to the Domain Controller

```powershell
Set-DnsClientServerAddress -InterfaceAlias "Ethernet" -ServerAddresses 10.0.0.1
```

### Join the domain

```
sysdm.cpl
```
click Change, and enter office.lab as the domain.
### Verify

```powershell
ping 10.0.0.1
nslookup office.lab
```

### Confirm domain login

After rebooting, log in with:

```
administrator@office.lab
```

Using the server password. If the login succeeds, PC01 is confirmed on the domain.

---

## Lab Ready

At this point both VMs were up, DC01 was running as a Domain Controller and PC01 was joined to the office.lab domain.
