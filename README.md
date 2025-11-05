# httpd_protection

**httpd_protection** is a lightweight configuration management and hardening toolkit for Apache (httpd) servers, integrating **fail2ban** rules and optional health monitoring.

It is designed for easy deployment, synchronization, and maintenance from a single GitHub repository.

---

## 📁 Repository Layout

```
httpd_protection/
├── README.md
├── hardening              # Main Perl management script
├── files/
│   ├── fail2ban/
│   │   ├── filter.d/
│   │   │   ├── apache-fcgi-probe.conf
│   │   │   ├── apache-sni-emptyhost.conf
│   │   │   └── apache-us3-queryspam.conf
│   │   └── jail.d/
│   │       └── 99-hardening.local
│   ├── httpd/
│   │   └── conf.d/
│   │       └── hardening.conf
│   └── sbin/
│       ├── apache-health
│       └── list_bans
└── roles/                 # (Reserved for future Ansible automation)
```

---

## ⚙️ The `hardening` Management Script

`hardening` automates installation, synchronization, and health monitoring of system-level configuration files.

### Usage

```
hardening --help
hardening --install
hardening --health-monitor on|off
hardening --status
hardening --update-repo
hardening --update-system
```

---

### 🧩 Commands

#### `--install`
- Installs all configuration files and helper scripts from the repo to system locations.
- Refuses to overwrite existing files.
- Ensures **fail2ban** is installed and running (`dnf`, `yum`, or `apt-get` supported).
- Creates required directories automatically.

#### `--health-monitor on|off`
- Enables or disables a cron job that periodically runs `apache-health` every 5 minutes.
- The cron file: `/etc/cron.d/httpd_protection_health`

#### `--status`
- Checks if installed and reports differences (by checksum) between repo and system files.
- Also prints concise **fail2ban** status (`systemctl is-active`, `fail2ban-client status`).

#### `--update-repo`
- Pulls updated configuration files **from system → repo**, keeping your repository in sync.

#### `--update-system`
- Pushes updated configuration files **from repo → system**.
- Makes timestamped backups of changed system files.
- Automatically restarts **fail2ban** if jail/filter files were updated.
- Prints an exact `systemctl reload httpd` command if `hardening.conf` was changed.

---

### 🛠️ Example Installation

```bash
# Clone repo (under admin account, e.g. /opt)
cd /opt
git clone https://github.com/yourorg/httpd_protection.git
cd httpd_protection

# Install configurations
sudo ./hardening --install

# Enable health monitor
sudo ./hardening --health-monitor on
```

---

### 🧾 Example Output

```
[OK] Created directory: /etc/fail2ban/filter.d
[OK] Installed: /etc/fail2ban/filter.d/apache-sni-emptyhost.conf
[OK] Installed: /etc/httpd/conf.d/hardening.conf
[OK] Installed: /usr/sbin/apache-health
[NOTE] Installation complete. Consider enabling the health monitor with:
  hardening --health-monitor on
```

---

### 🔄 Updating the System

```bash
sudo ./hardening --update-system
```

If `hardening.conf` changed, you’ll see:

```
[NOTE] httpd hardening.conf was updated. Reload httpd to apply changes. Run:
systemctl reload httpd || systemctl restart httpd
```

---

### 🧹 Updating the Repository

```bash
sudo ./hardening --update-repo
```

Copies modified system configs back into your local repo (useful for saving production adjustments).

---

### 🔍 Checking System Status

```bash
sudo ./hardening --status
```

Shows file differences and a concise **fail2ban** summary, for example:

```
[OK] All installed files match the repo versions.
--- fail2ban status ---
service: active=active, enabled=enabled
client: Server replied: pong
Status
|- Number of jail:      3
`- Jail list:           apache-fcgi-probe, apache-sni-emptyhost, apache-us3-queryspam
```

---

## 🧑‍💻 Future Work
- Ansible role integration under `roles/`
- Extended health checks (TLS cert expiry, file integrity)

---

## 🪪 License
MIT License — free for academic, research, and production use.
# httpd_protection
