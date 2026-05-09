# 03 - Active Directory

Configuration of Active Directory Domain Services including users, groups, Organizational Units and Group Policy Objects.

This is mostly a personal reference for AD commands I use regularly. Will expand with more context as the lab evolves.

---

## Users

### Create a new user

```powershell
New-ADUser -Name "name" `
-SamAccountName name `
-UserPrincipalName name@office.lab `
-AccountPassword (ConvertTo-SecureString "Password123!" -AsPlainText -Force) `
-Enabled $true
```

### List users and computers

```powershell
Get-ADUser -Filter * | Select Name, SamAccountName
Get-ADComputer -Filter * | Select Name
```

### Remove a user

```powershell
Remove-ADUser -Identity "username"
```

---

## Organizational Units

### Create OUs

```powershell
New-ADOrganizationalUnit -Name "Accounts" -Path "DC=office,DC=lab"
New-ADOrganizationalUnit -Name "Machines" -Path "DC=office,DC=lab"
New-ADOrganizationalUnit -Name "Groups" -Path "DC=office,DC=lab"
```

### Verify OUs were created

```powershell
Get-ADOrganizationalUnit -Filter *
```

### Rename an OU

```powershell
Rename-ADObject -Identity "OU=Accounts,DC=office,DC=lab" -NewName "Users"
```

### Remove an OU

```powershell
Remove-ADOrganizationalUnit -Identity "OU=Accounts,DC=office,DC=lab" -Recursive
```

---

## Groups

### Create a group inside the Groups OU

```powershell
New-ADGroup -Name "GRP-Finance" -GroupScope Global -Path "OU=Groups,DC=office,DC=lab"
```

### Rename a group

```powershell
Rename-ADObject -Identity "CN=GRP-Finance,OU=Groups,DC=office,DC=lab" -NewName "GRP-Finances"
```

### Remove a group

```powershell
Remove-ADGroup -Identity "GRP-Finance"
```

---

## Move Objects to OUs

### Move a user to an OU

```powershell
Get-ADUser name | Move-ADObject -TargetPath "OU=Accounts,DC=office,DC=lab"
```

### Move a computer to an OU

```powershell
Get-ADComputer name | Move-ADObject -TargetPath "OU=Machines,DC=office,DC=lab"
```

---

## Group Members

### Add user to a group

```powershell
Add-ADGroupMember -Identity "GRP-Finance" -Members username
```

### Remove user from a group

```powershell
Remove-ADGroupMember -Identity "GRP-Finance" -Members "username"
```

---

## Group Policy Objects (GPO)

> 📝 Note: GPO configuration is still being tested. Will update once confirmed working on PC01.

### Create a GPO

```powershell
New-GPO -Name "Restrict Control Panel - IT Only"
```

### Link GPO to an OU

```powershell
New-GPLink -Name "Restrict Control Panel - IT Only" -Target "OU=Accounts,DC=office,DC=lab"
```

### Configure the policy

```powershell
Set-GPRegistryValue -Name "Restrict Control Panel - IT Only" `
-Key "HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer" `
-ValueName "NoControlPanel" -Type DWord -Value 1
```

### Apply GPO to a specific group only

```powershell
# Remove Authenticated Users
Set-GPPermission -Name "Restrict Control Panel - IT Only" `
-TargetName "Authenticated Users" -TargetType Group -PermissionLevel None

# Add the target group
Set-GPPermission -Name "Restrict Control Panel - IT Only" `
-TargetName "GRP-IT" -TargetType Group -PermissionLevel GpoApply
```

### Rename a GPO

```powershell
Rename-GPO -Name "Restrict Control Panel - IT Only" -NewName "Restrict Control Panel - IT Only"
```

### Remove GPO link

```powershell
Remove-GPLink -Name "Restrict Control Panel - IT Only" -Target "OU=Accounts,DC=office,DC=lab"
```

### Remove GPO

```powershell
Remove-GPO -Name "Restrict Control Panel - IT Only"
```

### Verify GPOs

```powershell
Get-GPO -All | Select DisplayName
```
