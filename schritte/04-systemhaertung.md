# Modul 4 — Systemhärtung

**Ziel:** Weniger Angriffsfläche + mehr Stabilität.

**Gefahrlevel:** 🟢 niedrig, **außer** RAM‑Limits (Neustart nötig).

---

## Block 1: Alles schließen, was nicht gebraucht wird

### 1) Welche Ports hören nach außen?

```bash
# zeigt alle Dienste, die auf Ports "lauschen" (LISTEN)
sudo ss -tlnp | grep LISTEN
```

Alles auf `0.0.0.0` ist grundsätzlich von außen erreichbar.

### 2) Klassiker entfernen (wenn vorhanden)

```bash
# entfernt/stoppt CUPS (Druckdienst) — auf Servern fast nie nötig
sudo snap remove cups 2>/dev/null
sudo systemctl stop cups 2>/dev/null
sudo systemctl disable cups 2>/dev/null
```

### 3) Abschluss-Scan

```bash
# zeigt nur Dienste, die wirklich öffentlich erreichbar wären (0.0.0.0)
# erwartet: möglichst leer (außer sshd + ggf. webserver)
sudo ss -tlnp | grep "0.0.0.0" | grep -vE "sshd|caddy|nginx|tailscale|systemd-resolve"
```

---

## Block 2: Betriebssystem-Einstellungen verschärfen (sysctl)

### 4) Datei schreiben

```bash
# schreibt Kernel/Netzwerk-Sicherheitswerte (sysctl)
sudo tee /etc/sysctl.d/90-sicherheitssystem.conf << 'EOF'
# Bot Server Sicherheitssystem — Hardening

net.ipv4.conf.all.accept_redirects = 0
net.ipv4.conf.default.accept_redirects = 0
net.ipv6.conf.all.accept_redirects = 0
net.ipv6.conf.default.accept_redirects = 0
net.ipv4.conf.all.send_redirects = 0
net.ipv4.conf.default.send_redirects = 0
net.ipv4.conf.all.accept_source_route = 0
net.ipv4.conf.default.accept_source_route = 0
net.ipv4.tcp_syncookies = 1
net.ipv4.conf.all.log_martians = 1
net.ipv4.conf.default.log_martians = 1
net.ipv4.icmp_echo_ignore_broadcasts = 1
net.ipv4.icmp_ignore_bogus_error_responses = 1

kernel.randomize_va_space = 2
kernel.dmesg_restrict = 1
kernel.kptr_restrict = 1
kernel.yama.ptrace_scope = 2

vm.swappiness = 10
vm.overcommit_memory = 0
fs.suid_dumpable = 0

net.ipv6.conf.all.disable_ipv6 = 1
net.ipv6.conf.default.disable_ipv6 = 1
net.ipv6.conf.lo.disable_ipv6 = 1
EOF

# lädt alle sysctl-Dateien neu (macht die Werte sofort aktiv)
sudo sysctl --system
```

### 5) Dafür sorgen, dass es nach einem Neustart auch wirklich gilt

```bash
# sorgt dafür, dass die sysctl-Werte nach jedem Neustart wieder angewendet werden
sudo tee /etc/systemd/system/sicherheitssystem-boot.service << 'EOF'
[Unit]
Description=Sicherheitssystem — sysctl nach Boot anwenden
After=network.target

[Service]
Type=oneshot
ExecStart=/sbin/sysctl --system

[Install]
WantedBy=multi-user.target
EOF

# aktiviert Autostart für den Boot-Service
sudo systemctl enable sicherheitssystem-boot.service
```

---

## Block 3: Notfall-Speicher (Swap)

### 6) Prüfen

```bash
# zeigt, ob Swap schon aktiv ist + wie viel RAM/Swap vorhanden ist
swapon --show
free -h
```

Wenn kein Swap da ist:

```bash
# legt eine 4GB Swap-Datei an (Notfall-Puffer)
sudo fallocate -l 4G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile

# sorgt dafür, dass Swap nach Neustart wieder aktiv wird
echo "/swapfile none swap sw 0 0" | sudo tee -a /etc/fstab
```

---

## Block 4: RAM-Grenzen für OpenClaw

⚠️ Das kann einen kurzen Neustart des Dienstes auslösen.

### 7) Limits setzen

```bash
# findet den laufenden OpenClaw-Service-Namen
SERVICE_NAME=$(systemctl list-units --type=service | grep openclaw | awk '{print $1}' | head -1)

# legt systemd-Override an, um den RAM-Verbrauch zu begrenzen
sudo mkdir -p /etc/systemd/system/${SERVICE_NAME}.d
sudo tee /etc/systemd/system/${SERVICE_NAME}.d/limits.conf << 'EOF'
[Service]
MemoryMax=6G
MemoryHigh=5G
MemorySwapMax=512M
NoNewPrivileges=yes
EOF
```

### 8) Reload + Restart

```bash
# liest die neue Override-Datei ein
sudo systemctl daemon-reload

# startet OpenClaw neu (kurz offline)
sudo systemctl restart ${SERVICE_NAME}

# prüft ob der Dienst wieder läuft
sudo systemctl is-active ${SERVICE_NAME}
```

---

## Ergebnis-Check (kurz)

- Keine komischen öffentlichen Ports ✅
- sysctl aktiv + Boot‑Service aktiv ✅
- Swap vorhanden ✅
- MemoryMax/MemoryHigh gesetzt ✅
