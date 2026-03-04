# Modul 3 — Netzwerk

**Ziel:** Von außen ist nur offen, was wirklich offen sein muss.

**Gefahrlevel:** 🟡 mittel (Firewall kann Dinge blocken).

---

## Teil A: Firewall (UFW)

### 1) Status ansehen

```bash
# zeigt Firewall-Status + Regeln
sudo ufw status verbose
```

### 2) Standardregeln setzen

```bash
# blockt alles, was von außen rein will (Standard: zu)
sudo ufw default deny incoming

# erlaubt ausgehende Verbindungen (Updates, APIs, etc.)
sudo ufw default allow outgoing
```

### 3) Erlauben: SSH + (optional) Web

```bash
sudo ufw allow 2222/tcp comment "SSH"
sudo ufw allow 80/tcp comment "HTTP"   # nur wenn Webserver läuft
sudo ufw allow 443/tcp comment "HTTPS" # nur wenn Webserver läuft
```

### 4) Aktivieren

```bash
# aktiviert die Firewall
sudo ufw enable
```

### 5) Alten Standard-Port 22 entfernen

```bash
# entfernt den alten Standard-SSH-Port 22 (falls noch vorhanden)
sudo ufw delete allow 22/tcp 2>/dev/null
sudo ufw delete allow 22 2>/dev/null

# zeigt Regeln inkl. Nummern (hilft beim Debugging)
sudo ufw status numbered
```

---

## Teil B: Schutz gegen Login-Angriffe (Fail2Ban)

### 6) Installieren + aktivieren

```bash
# installiert Fail2Ban (sperrt IPs bei auffälligen Login-Versuchen)
sudo apt install -y fail2ban

# startet es sofort + aktiviert Autostart
sudo systemctl enable --now fail2ban
```

### 7) Konfiguration (inkl. Wiederholungstäter)

```bash
# schreibt die Fail2Ban-Regeln (sshd sperrt schnell, recidive sperrt Wiederholungstäter lange)
sudo tee /etc/fail2ban/jail.local << 'EOF'
[DEFAULT]
bantime = 3600
findtime = 600
maxretry = 3

[sshd]
enabled = true
port = 2222
filter = sshd
logpath = %(sshd_log)s
backend = %(sshd_backend)s

[recidive]
enabled = true
logpath = /var/log/fail2ban.log
banaction = %(banaction_allports)s
bantime = 604800
findtime = 86400
maxretry = 5
EOF
```

### 8) Neustart + Kurzcheck

```bash
# lädt neue Konfiguration
sudo systemctl restart fail2ban
sleep 2

# Übersicht: welche Schutzregeln (Jails) aktiv sind?
sudo fail2ban-client status

# Details speziell für SSH
sudo fail2ban-client status sshd
```

---

## Ergebnis-Check (kurz)

- UFW: active ✅
- Nur SSH + ggf. 80/443 erlaubt ✅
- Fail2Ban läuft ✅
- Jails: sshd + recidive ✅
