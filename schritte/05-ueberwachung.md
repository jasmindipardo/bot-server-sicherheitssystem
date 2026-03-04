# Modul 5 — Überwachung

**Ziel:** Du merkst früh, wenn etwas aus dem Ruder läuft (Speicher, Festplatte, Log‑Wachstum).

**Gefahrlevel:** 🟢 niedrig.

---

## 1) 15‑Minuten‑Gesundheitscheck installieren

```bash
# erstellt ein kleines Prüf-Skript (RAM/Swap/Disk)
sudo tee /usr/local/bin/sicherheitscheck.sh << 'SCRIPT'
#!/bin/bash
export PATH="/usr/sbin:/usr/bin:/sbin:/bin:$PATH"

RAM_TOTAL=$(free -m | awk '/^Mem:/{print $2}')
RAM_USED=$(free -m | awk '/^Mem:/{print $3}')
RAM_PCT=$((RAM_USED * 100 / RAM_TOTAL))

SWAP_TOTAL=$(free -m | awk '/^Swap:/{print $2}')
SWAP_USED=$(free -m | awk '/^Swap:/{print $3}')
if [ "$SWAP_TOTAL" -gt 0 ]; then
  SWAP_PCT=$((SWAP_USED * 100 / SWAP_TOTAL))
else
  SWAP_PCT=0
fi

DISK_PCT=$(df / | tail -1 | awk '{print $5}' | sed 's/%//')

ALERT=""
if [ "$RAM_PCT" -gt 80 ]; then ALERT="RAM ${RAM_PCT}%"; fi
if [ "$SWAP_PCT" -gt 50 ]; then ALERT="$ALERT SWAP ${SWAP_PCT}%"; fi
if [ "$DISK_PCT" -gt 85 ]; then ALERT="$ALERT DISK ${DISK_PCT}%"; fi

if [ -n "$ALERT" ]; then
  echo "WARNUNG:$ALERT"
  exit 1
else
  echo "OK: RAM ${RAM_PCT}% | SWAP ${SWAP_PCT}% | DISK ${DISK_PCT}%"
  exit 0
fi
SCRIPT

# macht das Skript ausführbar
sudo chmod +x /usr/local/bin/sicherheitscheck.sh
```

Cronjob alle 15 Minuten:

```bash
# richtet einen Cronjob ein, der alle 15 Minuten prüft (bei Warnung wird ins Systemlog geschrieben)
sudo tee /etc/cron.d/sicherheitscheck << 'EOF'
*/15 * * * * root /usr/local/bin/sicherheitscheck.sh > /dev/null 2>&1 || logger -t sicherheitscheck "$(/usr/local/bin/sicherheitscheck.sh 2>&1)"
EOF
```

---

## 2) Log-Größe begrenzen (damit die Platte nicht vollläuft)

```bash
# begrenzt die Größe der systemd-Logs (sonst kann die Festplatte volllaufen)
sudo mkdir -p /etc/systemd/journald.conf.d
sudo tee /etc/systemd/journald.conf.d/groesse.conf << 'EOF'
[Journal]
SystemMaxUse=500M
RuntimeMaxUse=100M
EOF

# übernimmt die neuen Log-Grenzen
sudo systemctl restart systemd-journald
```

---

## 3) Login-Protokoll

```bash
# schreibt bei jedem Login eine Zeile ins Log (wer, wann, von welcher IP)
sudo tee /etc/profile.d/login-protokoll.sh << 'EOF'
#!/bin/bash
echo "$(date '+%Y-%m-%d %H:%M') — Login: $(whoami) von ${SSH_CLIENT%% *}" >> /var/log/server-logins.log
EOF
# aktiviert das Login-Skript
sudo chmod +x /etc/profile.d/login-protokoll.sh
```

Rotation:

```bash
# sorgt dafür, dass das Login-Log nicht endlos wächst
sudo tee /etc/logrotate.d/sicherheitssystem << 'EOF'
/var/log/server-logins.log {
  weekly
  rotate 12
  compress
  missingok
  notifempty
}
EOF
```

---

## Kurztest

```bash
# Test: Skript einmal manuell ausführen
/usr/local/bin/sicherheitscheck.sh

# Test: Cronjob-Datei ist da
cat /etc/cron.d/sicherheitscheck

# Info: wie groß sind die systemd-Logs aktuell?
journalctl --disk-usage
```

---

## Ergebnis-Check (kurz)

- Script vorhanden ✅
- Cronjob vorhanden ✅
- journald begrenzt ✅
- Login-Log schreibt ✅
