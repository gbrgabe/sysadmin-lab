# Sysadmin Lab - Windows 🖥️

A hands-on home lab built to simulate a real small corporate environment using VirtualBox.
This repository documents the full setup, configurations, and everything I learned along the way.

---

## Lab Architecture

| Machine | Role | OS | IP | RAM | CPU |
|---|---|---|---|---|---|
| DC01 | Domain Controller | Windows Server 2022 Core | 10.0.0.1 | 2.5GB | 2 |
| PC01 | Client | Windows 11 | 10.0.0.5 | 7GB | 2 |

- **Domain:** office.lab
- **Network:** Internal Network (adlab) - Intel PRO/1000 MT Desktop
- **DNS:** 10.0.0.1

---

## What was implemented

- [01 - Infrastructure & VirtualBox setup](01-infrastructure/)
- [02 - Network Services - Static IP, DNS Forwarders, Internal Network](02-networking/)
- [03 - Active Directory - AD DS, OUs, Users, Groups, GPOs](03-active-directory/)
- [04 - GLPI Ticketing System - MariaDB, IIS](04-glpi/)
- [05 - PowerShell - Headless server management](05-powershell/)
- 06 - Linux Lab - coming soon

---

## Why I built this

Without hands-on experience it would be nearly impossible to land a job, especially aiming for sysadmin roles.
So I built this lab from scratch and documented everything here.

---

## Environment

| Tool | Version |
|---|---|
| VirtualBox | 7.2.6 |
| Windows Server | 2022 Core |
| Windows | 11 |
| GLPI | 10.0.15 |
| MariaDB | Latest |
| PHP | 8.3 |
| Chocolatey | Latest |
