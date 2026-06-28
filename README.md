# osTicket Setup on Azure — Prerequisites and Installation

## Overview

In this lab, I set up osTicket, an open-source help desk ticketing system, on a Windows 11 virtual machine in Azure. The main goal wasn't just installing osTicket itself — it was building and configuring the full supporting stack around it.

Most of the work came from getting IIS, PHP, and MySQL to correctly work together. IIS doesn't natively run PHP, so I had to manually configure FastCGI, wire PHP into IIS, and make sure database connectivity worked before osTicket would even load. Once the environment was stable, the osTicket installation itself was straightforward.

## Environment

- Azure subscription
- Windows 11 VM
- IIS (Internet Information Services) with CGI/FastCGI support
- PHP 8.2 (Non-Thread Safe / x64)
- MySQL 8.0
- HeidiSQL (database management tool)
- osTicket v1.18.x

## Diagram

`[add diagram later]`

---

## Task 1: Provision the Virtual Machine

First, I created a Windows 11 virtual machine in Azure inside the **rg-osticket-lab** resource group. This VM would host the entire osTicket environment.

After deployment finished, I copied the public IP address from the VM overview page so I could connect to it remotely.

<img width="137" height="92" alt="image" src="https://github.com/user-attachments/assets/03c0750e-bc0e-4023-88ff-3be5f363ed2c" />

---

## Task 2: Connect via Remote Desktop

Next, I connected to the VM using Remote Desktop Connection and the public IP.

Once inside the VM, I opened a browser and downloaded all required components:

- IIS URL Rewrite Module
- PHP 8.2 (Non-Thread Safe, x64)
- Visual C++ Redistributable
- MySQL 8.0 Installer
- osTicket package

I extracted everything into a working folder on the desktop and cleaned up the zip files afterward to keep the environment organized.

<img width="663" height="266" alt="image" src="https://github.com/user-attachments/assets/b0a01bd0-0035-498e-bab1-0039f2bbd2c8" />

---

## Task 3: Enable IIS with CGI

To support PHP, I enabled IIS with CGI support.

I went to **Windows Features → Turn Windows features on or off**, then enabled:

- Internet Information Services
- World Wide Web Services → Application Development Features → CGI

CGI is required because IIS needs it to pass PHP requests to FastCGI.



---

## Task 4: Install URL Rewrite Module

I installed the IIS URL Rewrite Module, which osTicket uses for cleaner URL handling.

I skipped installing PHP Manager because it does not reliably support PHP 8.x. Instead, I planned to configure PHP manually using IIS Handler Mappings later.

`[ADD SCREENSHOT HERE]`

---

## Task 5: Set Up PHP

I created a folder at `C:\PHP` and extracted the PHP 8.2 (NTS x64) files into it.

Then I installed the Visual C++ Redistributable since PHP depends on it to run properly.

Inside the PHP folder, I copied `php.ini-production` and renamed it to `php.ini`. This file is required because PHP doesn't come preconfigured. I later enabled required extensions inside it.

![Uploading image.png…]()


---

## Task 6: Install MySQL

I installed MySQL 8.0 using the MySQL Installer and selected the **Server Only** option since I only needed the database engine.

During setup, I:

- Used default standalone configuration
- Enabled strong password authentication
- Set a root password for database access

`[ADD SCREENSHOT HERE]`

---

## Task 7: Wire PHP into IIS

Instead of using PHP Manager (which doesn't support PHP 8 well), I manually connected PHP to IIS.

In IIS Manager, I went to **Handler Mappings** and added a module mapping:

| Setting | Value |
|---|---|
| Request path | `*.php` |
| Module | FastCgiModule |
| Executable | `C:\PHP\php-cgi.exe` |
| Name | FastCGI |

I also added `index.php` to the default document list so IIS would correctly load PHP index pages.

After that, I restarted IIS to apply the changes.

`[ADD SCREENSHOT HERE]`

---

## Task 8: Install osTicket Files

I extracted the osTicket package and copied the `upload` folder into:

```
C:\inetpub\wwwroot
```

Then I renamed it to `osTicket`.

After restarting IIS, I navigated to the site in the browser using:

```
http://localhost/osTicket
```

`[ADD SCREENSHOT HERE]`

---

## Task 9: Enable Required PHP Extensions

When loading the osTicket installer, I noticed missing PHP extensions.

To fix this, I edited `C:\PHP\php.ini` and enabled:

- `php_imap.dll`
- `php_intl.dll`
- `php_opcache.dll`

After saving the file, I restarted IIS and refreshed the installer page. All required extensions were now enabled.

`[ADD SCREENSHOT HERE]`

---

## Task 10: Configure ost-config.php

I navigated to:

```
C:\inetpub\wwwroot\osTicket\include\
```

and renamed `ost-sampleconfig.php` to `ost-config.php`.

Then I updated file permissions:

- Disabled inheritance
- Removed existing permissions
- Granted Everyone full control

This allowed the installer to write configuration data during setup.

`[ADD SCREENSHOT HERE]`

---

## Task 11: Set Up the Database with HeidiSQL

I installed HeidiSQL and connected it to the MySQL server using:

| Setting | Value |
|---|---|
| Username | root |
| Password | *(the one set during MySQL installation)* |

After connecting, I created a new database named:

```
osTicket
```

This database would store all ticketing system data.

`[ADD SCREENSHOT HERE]`

---

## Task 12: Finish osTicket Installation

Back in the browser, I completed the osTicket setup by entering:

| Setting | Value |
|---|---|
| MySQL Database | osTicket |
| MySQL Username | root |
| MySQL Password | *(root password)* |

After clicking **Install**, osTicket successfully completed setup and was ready to use.

`[ADD SCREENSHOT HERE]`

---

## Key Takeaways

- The hardest part of this lab wasn't osTicket — it was getting IIS, PHP, and MySQL to properly communicate.
- Modern PHP on IIS requires manual configuration using FastCGI instead of older tools like PHP Manager.
- Most installation issues came from missing PHP extensions or incorrect IIS handler setup.
- Once the stack was correctly configured, osTicket installed without issues.
- HeidiSQL made database setup much easier compared to using MySQL Workbench.


