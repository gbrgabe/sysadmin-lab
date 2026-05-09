# 04 - GLPI Ticketing System

Installation and configuration of GLPI 10.0.15 on Windows Server 2022 Core using IIS, PHP 8.3 and MariaDB.

Before starting, decide which version of GLPI you'll be installing, as this determines which PHP version you need. I went with GLPI 10.0.15 as it's the most stable version for learning. I learned the hard way that PHP 8.5 is not compatible, so if you're using older GLPI versions, go with PHP 8.3 from the start.

---

## Stack

| Tool | Version |
|---|---|
| GLPI | 10.0.15 |
| PHP | 8.3 |
| MariaDB | Latest |
| IIS | Windows Feature |
| Chocolatey | Latest |

---

## Step 1 - Install IIS and CGI

```powershell
Install-WindowsFeature -Name Web-Server, Web-Mgmt-Tools, Web-Asp-Net45, Web-CGI
```

---

## Step 2 - Install Chocolatey, PHP, MariaDB and 7-Zip

Because I'm implementing GLPI on Windows Server Core, I started by installing Chocolatey to be able to download everything from the command line. On a Core server without GUI, this is much easier than dealing with long Invoke-WebRequest URLs.

```powershell
# Install Chocolatey
Set-ExecutionPolicy Bypass -Scope Process -Force
[System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072
iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))

# Update Path
$env:Path = [System.Environment]::GetEnvironmentVariable("Path","Machine") + ";" + [System.Environment]::GetEnvironmentVariable("Path","User")

# Install PHP, MariaDB and 7-Zip
# 7-Zip is needed to extract the .tgz and .tar files in Step 4 - Windows Server Core has no native tool for this.
choco install php mariadb 7zip -y
# Note: If you're installing GLPI 10.0.15, install PHP 8.3 directly to avoid version issues:
# choco install php --version=8.3 mariadb 7zip -y

# Update Path again
$env:Path = [System.Environment]::GetEnvironmentVariable("Path","Machine") + ";" + [System.Environment]::GetEnvironmentVariable("Path","User")
```

> ⚠️ Chocolatey installs the latest PHP version by default. GLPI 10.0.15 is not compatible with PHP 8.5+. After installation, remove PHP 8.5 and install PHP 8.3 manually.

```powershell
choco uninstall php -y
choco install php --version=8.3 -y
```

---

## Step 3 - Configure php.ini

> ⚠️ Do this before opening the browser. Skipping this step causes a database timeout error during GLPI installation.

This was probably what took most of my time. Avoiding this error will save you a HEADACHE. Trust me, sort this out first. 
I only figured this out after deleting everything and running into the same error again, so I'm adding it here as Step 3. 
After starting the GLPI wizard, I got a timeout error specifically when selecting the database. The browser then started returning 500 errors.
The installation failed midway but the database had already been partially created, 
so I had to delete it, fix php.ini and start the wizard again.

```
notepad C:\tools\php83\php.ini
```

Set the following values:

```ini
# Timeout & Memory
max_execution_time = 600
memory_limit = 512M

# Session
session.cookie_httponly = On

# Path
extension_dir = "C:\tools\php83\ext"

# Error reporting (useful during setup)
display_errors = On
display_startup_errors = On
log_errors = On
error_reporting = E_ALL
```

Enable the following extensions (remove the `;` at the beginning of each line):

```ini
extension=curl
extension=exif
extension=gd
extension=intl
extension=mbstring
extension=mysqli
extension=openssl
extension=zip
extension=sodium
extension=fileinfo
extension=ldap
extension=bz2
```
```
# Access MySQL to delete and recreate the database if needed.
# That's how I troubleshot after getting the timeout error
& "C:\Program Files\MariaDB 12.2\bin\mysql.exe" -u root

DROP DATABASE glpidb;
CREATE DATABASE glpidb CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```

---

## Step 4 - Download and Extract GLPI

GLPI is not available on Chocolatey, so I downloaded it directly from the official GitHub release using Invoke-WebRequest.

```powershell
cd C:\inetpub\wwwroot
Invoke-WebRequest -Uri "https://github.com/glpi-project/glpi/releases/download/10.0.15/glpi-10.0.15.tgz" -OutFile "glpi.tgz"

# Extract
7z x glpi.tgz
7z x glpi.tar

# Grant IIS permissions
icacls "C:\inetpub\wwwroot\glpi" /grant "IIS_IUSRS:(OI)(CI)F" /T
icacls "C:\inetpub\wwwroot\glpi" /grant "IUSR:(OI)(CI)F" /T
```

> 📝 Note: Technically IIS_IUSRS should be enough, but I ran both just to be safe and it worked.

---

## Step 5 - Configure IIS default document

Before the timeout issue, I kept facing 403 errors trying to access the GLPI URL. This tells IIS to recognize `index.php` as the default document, and this was the fix.

```powershell
Add-WebConfigurationProperty -filter /system.webServer/defaultDocument/files -name "." -value @{value='index.php'} -PSPath 'IIS:\'
```

---

## Step 5.1 - Configure FastCGI for PHP

```powershell
# Configure FastCGI handler for PHP
Import-Module WebAdministration
New-WebHandler -Name "PHP" -Path "*.php" -Verb "*" -Modules "FastCgiModule" -ScriptProcessor "C:\tools\php83\php-cgi.exe"

Add-WebConfigurationProperty -pspath 'MACHINE/WEBROOT/APPHOST' -filter "system.webServer/handlers" -name "." -value @{name='PHP_via_FastCGI';path='*.php';verb='*';modules='FastCgiModule';scriptProcessor='C:\tools\php83\php-cgi.exe';resourceType='Either'}
```

> 📝 Note: I'm not 100% sure which of these two commands is strictly necessary. When I redo this lab from scratch I'll update this section with what's actually required.

---

## Step 6 - Initialize MariaDB and create GLPI database

```powershell
# Check the actual service name first
Get-Service | findstr "maria mysql"

# Start MySQL service
Start-Service mysql

# Create database and user
mysql -u root -e "CREATE DATABASE glpidb;"
mysql -u root -e "CREATE USER 'glpiuser'@'localhost' IDENTIFIED BY 'Senha123!';"
mysql -u root -e "GRANT ALL PRIVILEGES ON glpidb.* TO 'glpiuser'@'localhost';"
mysql -u root -e "FLUSH PRIVILEGES;"
```

> 📝 Note: Even though I installed MariaDB via Chocolatey, the service runs as `mysql`. I used `Get-Service` to confirm the actual service name before trying to start it.

---

## Step 7 - GLPI Web Installation

Access from PC01 browser:

```
http://10.0.0.1/glpi/index.php
```

Follow the installation wizard and use the database credentials created in Step 6.

---

## Lessons Learned

- GLPI 10.0.15 requires PHP 8.3, versions above are not compatible
- php.ini must be configured before the browser installation, timeout errors will occur otherwise
- `extension_dir` path must match the actual PHP 8.3 installation directory
- 403 errors on the GLPI URL are fixed by configuring the IIS default document to recognize `index.php`
- If the installation times out and the browser starts returning 500 errors, delete the database and recreate it before running the wizard again
- Even though MariaDB is installed via Chocolatey, the service runs as `mysql`, use `Get-Service | findstr "maria mysql"` to confirm before trying to start it
