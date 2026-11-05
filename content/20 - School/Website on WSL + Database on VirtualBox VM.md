---
title: Website on WSL + Database on VirtualBox VM
tags:
  - web-development
  - home-lab
  - wsl
  - virtualbox
  - apache
  - php
  - mysql
  - networking
---

# 🖥️ Website on WSL + Database on a VirtualBox VM

Beginner guide. Ubuntu on both sides. Apache + PHP + MySQL.
*Source: `WSL_Website_VM_Database_Guide.pdf` (Downloads), Nov 2026.*

---

## 1. The Big Picture

### Architecture Flow
`Windows Browser` --[HTTP]--> `WSL Ubuntu (Apache + PHP)` --[SQL :3306]--> `VirtualBox Ubuntu VM (MySQL database)`

Your code lives in WSL. Your data lives in the VM.

### Component Breakdown

| Part | Where it lives | Job |
| :--- | :--- | :--- |
| **Browser** | Windows | Opens your site |
| **Web server** | WSL Ubuntu | Apache runs your PHP code |
| **Database** | VirtualBox Ubuntu VM | MySQL stores the data |

The web server and database talk through the network. WSL connects to the VM using the VM's IP address and port 3306. That's the whole trick.

> **Assumption:** "Oracle Virtual Machine" = Oracle VirtualBox. If something else (like Oracle Cloud) was meant, adapt accordingly.

---

## 2. Use Ubuntu, not Kali

Both exist on WSL and VirtualBox. For web dev, pick Ubuntu on both.

| Feature | Ubuntu | Kali |
| :--- | :--- | :--- |
| **Made for** | General use, servers, dev | Hacking / security testing |
| **Tutorials online** | Almost every web guide | Very few for web dev |
| **Extra stuff** | Clean, lightweight | Hundreds of tools you don't need |
| **Commands** | `apt` | `apt` (same, so this guide still works) |

Keep Kali for the cybersecurity stuff. Mixing them up just makes debugging harder.

---

## 3. Part A: Set up the database (VirtualBox VM)

Do this inside the Ubuntu VM and VirtualBox settings.

### A1. Set the VM network to Bridged
1. Keep the VM powered off.
2. In VirtualBox: `select VM` → `Settings` → `Network` → `Adapter 1`.
3. Set **Attached to:** `Bridged Adapter`.
4. Set **Name:** pick the active Wi-Fi or Ethernet adapter.
5. Click **OK**, then start the VM.

*Bridged gives the VM its own IP on the network, so WSL can reach it directly.*

> ⚠️ **School Wi-Fi warning:** Some networks block Bridged mode. If it fails, use the [[#NAT fallback (if Bridged is blocked)|NAT fallback]] in Section 6.

### A2. Get the VM's IP address
Inside the VM terminal, run:
```bash
hostname -I
```
Write down the first number (e.g., `192.168.1.50`). Use that IP everywhere below.

### A3. Install MySQL
```bash
sudo apt update
sudo apt install mysql-server -y
sudo systemctl enable --now mysql
sudo systemctl status mysql
```
`active (running)` should appear. Press `q` to exit.

### A4. Create a database and a user
Open the MySQL shell, then paste these lines:
```sql
sudo mysql

CREATE DATABASE mysite_db;
CREATE USER 'webuser'@'%' IDENTIFIED BY 'StrongPass123!';
GRANT ALL PRIVILEGES ON mysite_db.* TO 'webuser'@'%';
FLUSH PRIVILEGES;
EXIT;
```
Change the password to a personal one. The `%` means *"this user can connect from any IP"* — fine for learning. **Never use `root` for the application.**

### A5. Allow connections from outside the VM
By default MySQL only listens to the VM itself. To change that:
```bash
sudo nano /etc/mysql/mysql.conf.d/mysqld.cnf
```
Find `bind-address = 127.0.0.1` and change it to `bind-address = 0.0.0.0`. Save and exit: `Ctrl+O`, `Enter`, `Ctrl+X`. Then restart and verify the port binding:
```bash
sudo systemctl restart mysql
sudo ss -tlnp | grep 3306
```
`0.0.0.0:3306` should appear. If it says `127.0.0.1:3306`, the edit didn't save.

### A6. Firewall (only if it's on)
```bash
sudo ufw status
sudo ufw allow 3306/tcp
```
Ubuntu's firewall is off by default. If status says `inactive`, skip the second command.

---

## 4. Part B: Set up the web server (WSL Ubuntu)

Switch to the Ubuntu on WSL terminal.

### B1. Update and install Apache + PHP
```bash
sudo apt update && sudo apt upgrade -y
sudo apt install apache2 php libapache2-mod-php php-mysql \
    php-mbstring php-xml php-curl unzip -y
```
Check the PHP version: `php -v`

### B2. Start Apache
```bash
sudo service apache2 start
```
On Windows, open `http://localhost` in a browser — the Apache default page should appear.

> **Remember:** WSL doesn't auto-start services. Every time WSL is reopened, run `sudo service apache2 start` again.

### B3. Make the first page
Give yourself ownership of the web folder, then replace the default page:
```bash
sudo chown -R $USER:$USER /var/www/html
cd /var/www/html
rm index.html
echo "<h1>Hello from WSL</h1>" > index.php
```
Refresh `http://localhost` — the heading should appear.

*Edit files with VS Code: run `code .` inside the folder (requires the WSL extension). Keep projects inside Linux folders like `/var/www/html` or `~/`, never in `/mnt/c` — far slower there.*

---

## 5. Part C: Connect the website to the database

### C1. Test the connection from WSL first
Before writing PHP, verify WSL can reach the VM (replace the IP with the real VM IP):
```bash
sudo apt install mysql-client -y
mysql -h 192.168.1.50 -u webuser -p mysite_db
```
A `mysql>` prompt means the network side works. Type `EXIT;` to leave.

### C2. Add some test data (in the VM)
```sql
sudo mysql
USE mysite_db;
CREATE TABLE notes (id INT AUTO_INCREMENT PRIMARY KEY, text VARCHAR(100));
INSERT INTO notes (text) VALUES ('Hello from the VM');
EXIT;
```

### C3. Connect with PHP
Back in WSL, create `/var/www/html/db.php`:
```php
<?php
$host = "192.168.1.50";   // your VM IP
$db   = "mysite_db";
$user = "webuser";
$pass = "StrongPass123!";

try {
    $pdo = new PDO("mysql:host=$host;dbname=$db;charset=utf8mb4", $user, $pass);
    $pdo->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);
    echo "Connected!<br>";
    $rows = $pdo->query("SELECT * FROM notes")->fetchAll(PDO::FETCH_ASSOC);
    foreach ($rows as $r) {
        echo htmlspecialchars($r["text"]) . "<br>";
    }
} catch (PDOException $e) {
    echo "Failed: " . $e->getMessage();
}
```
Open `http://localhost/db.php` on Windows. **Connected!** and **Hello from the VM** should appear — the website is reading from the VM's database.

### C4. Using Laravel instead?
Same idea. Point `.env` to the VM:
```env
DB_CONNECTION=mysql
DB_HOST=192.168.1.50
DB_PORT=3306
DB_DATABASE=mysite_db
DB_USERNAME=webuser
DB_PASSWORD=StrongPass123!
```
Then run:
```bash
php artisan migrate
php artisan serve --host=0.0.0.0 --port=8000
```
Open `http://localhost:8000` on Windows. Laravel 11/12 need PHP 8.2+. Check with `php -v`. Ubuntu 24.04 has it; Ubuntu 22.04 doesn't by default.

---

## 6. When it breaks

| Problem | Likely cause | Fix |
| :--- | :--- | :--- |
| **Can't ping the VM** | Wrong adapter, or network blocks Bridged | Re-pick the adapter in VirtualBox. Still stuck? NAT fallback below. |
| **Connection refused (3306)** | MySQL not running, or bind-address still `127.0.0.1` | Redo A5. `sudo ss -tlnp \| grep 3306` must show `0.0.0.0`. |
| **Access denied for user** | Wrong password, or user not created with `%` | Redo A4. Check the exact password. |
| **Unknown database** | Typo in DB name | Run `SHOW DATABASES;` in the VM and compare. |
| **could not find driver** | `php-mysql` missing | `sudo apt install php-mysql` then `sudo service apache2 restart`. |
| **localhost won't load** | Apache stopped | `sudo service apache2 start`. |
| **Worked yesterday, fails now** | VM got a new IP | Run `hostname -I` in the VM and update the code/config. |

### NAT fallback (if Bridged is blocked)
1. `VirtualBox` → `VM Settings` → `Network` → `Adapter 1`.
2. Set to **NAT** → **Advanced** → **Port Forwarding**, and add a rule:
   * **Host Port:** `3307`
   * **Guest Port:** `3306`
   * *(Leave IP fields blank.)*
3. In WSL, find the Windows host IP:
   ```bash
   ip route show | grep default
   ```
4. Use the IP after `via` as `DB_HOST`, with port `3307`. If it still fails, allow port 3307 in Windows Defender Firewall.

---

## 7. Safety Basics
* Use `webuser`, never `root`, in the code.
* Don't push passwords to GitHub. Use `.env` and add it to `.gitignore`.
* Bridged mode on public or school Wi-Fi exposes the database port to others on that network. Use NAT there, or stop MySQL (`sudo systemctl stop mysql`) when done.
* Later, replace `%` with the specific WSL IP so only WSL can connect.

---

## 8. Daily Start Routine
1. **VirtualBox:** Start the Ubuntu VM.
2. **VM terminal:** `sudo systemctl status mysql` (should be running).
3. **WSL terminal:** `sudo service apache2 start`.
4. **Windows browser:** Open `http://localhost`.

---

## 🔗 Connections & Related Notes
* **Local dev & deployment stacks:** [[Zero to Live URL (Elvin Ray)]] & [[Build client ready website fast with Claude (Jlo Edios)]] — this guide is the on-PC counterpart to the cloud/Vercel deployment stacks from those events.
* **Parent hub:** [[School & Computer Science (Map of Content)]]
* **Advanced Database coursework:** the MySQL concepts here (CREATE USER, GRANT PRIVILEGES, binding/listening) connect to `[[Midterm Review]]` in the DB course.
* **Return Home:** [[Hi, I'm Jm|Vault Home]]
