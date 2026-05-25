---
title: Sequel Challenge Report
date: 2025-03-16
draft: false
tags:
  - publish
lastmod: 2026-05-25T14:43:14.497Z
---
### **Challenge Details**

**Category:** SQL\
**Description:** Sequel is a Linux machine that introduces a vulnerable MySQL service misconfigured to allow access without a password. The machine showcases how to enumerate and interact with the database through SQL queries to extract critical information.\
**Given Files/Link:** app.hackthebox.com/machines/Sequel?sort\_by=created\_at\&sort\_type=desc

## 1. Summary

Sequel is a Linux machine on HackTheBox that demonstrates the dangers of database misconfigurations. The core vulnerability lies in a MariaDB/MySQL service that is exposed to the network and allows the `root` user to authenticate without a password. By connecting to the service remotely and performing standard SQL enumeration, an attacker can effortlessly extract sensitive information, including the system flag.

## 2. Reconnaissance

The first step is to scan the target IP (`10.129.7.83`) to identify open ports and running services. We use Nmap with default scripts and version detection.

**Command:**

```
nmap -sC -sV 10.129.7.83
```

**Results:**

* **Port:** 3306/tcp is open.
* **Service:** MySQL
* **Version:** `5.5.5-10.3.27-MariaDB-0+deb10u1`\
  ![](Pasted%20image%2020260525223402.png)

The scan confirms that a database server is running and directly accessible over the network.

## 3. Exploitation

With port 3306 open, the next step is to attempt interaction with the database. A common, highly critical misconfiguration is leaving the default `root` user account without a password.\
We can attempt to log in remotely using the MySQL command-line client:

**Command:**

```
mysql -u root -p -h 10.129.7.83 --skip-ssl
```

![2026-05-22\_220440 (1).png](/ob/2026-05-22_220440%20\(1\).png)

When prompted for the password, simply pressing **Enter** grants us access. The server drops us directly into the `MariaDB [(none)]>` prompt. This confirms the vulnerability: unauthenticated/passwordless root access to the database.

## 4. Persistence

In the context of this specific introductory challenge, gaining standard system-level persistence (like SSH keys, cron jobs, or reverse shells) is not required, as the primary objective is housed entirely within the database.

However, in a real-world scenario, an attacker with root database privileges might establish persistence by creating a backdoor database user with full privileges, or by using `SELECT ... INTO OUTFILE` to write a web shell to the server's filesystem (if a web server is concurrently running).

## 5. Getting the Flag

Now that we have root access to the database, we need to enumerate the environment to find the flag.

**Step 1: List all databases**

```
SHOW DATABASES;
```

![2026-05-22\_220534.png](/ob/2026-05-22_220534.png)

The output reveals a custom database named **htb** alongside the standard system databases (`information_schema`, `mysql`, `performance_schema`).

**Step 2: Select the target database**

```
USE htb;
```

**Step 3: Enumerate the tables within the database**

```
SHOW TABLES;
```

![2026-05-22\_220840.png](/ob/2026-05-22_220840.png)

This query reveals two tables of interest: **config** and **users**.

**Step 4: Dump the table contents** Since "config" often contains valuable environment variables or flags in CTF challenges, we query it first.

```
This query reveals two tables of interest: **config** and **users**.
```

![2026-05-22\_221028.png](/ob/2026-05-22_221028.png)

**Result:** The table outputs several configuration variables (timeout, security, auto\_logon, max\_size). In the 5th row, we successfully locate our target:

* **Name:** flag
* **Value:** `7b4bec00d1a39e3dd4e021ec3d915da8`
