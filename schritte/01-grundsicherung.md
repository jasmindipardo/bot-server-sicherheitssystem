# Modul 1 — Grundsicherung

**Ziel:** Du hast eine Rückfall-Option (Snapshot) und ein aktuelles System.

**Gefahrlevel:** 🟢 niedrig (hier sperrst du dich nicht aus).

---

## Stopp, bevor du weitermachst

Wenn du **keinen Snapshot** hast: nicht starten. Erst Snapshot, dann weitermachen.

---

## 1) Snapshot beim Hoster (macht der User im Browser)

- **Hetzner:** Cloud Console → Server → Snapshots → Snapshot erstellen
- **Netcup:** SCP → Server → Snapshot
- **Contabo:** Control Panel → Snapshots

Warte, bis der User schreibt: „Snapshot ist fertig“.

---

## 2) Kurz-Check: Ist genug Platz auf der Festplatte?

```bash
# zeigt, wie voll die System-Festplatte ist (Ziel: unter 80%)
df -h /
```

Wenn über **80%** belegt, erst aufräumen:

```bash
# verkleinert System-Logs (können sonst mehrere GB belegen)
sudo journalctl --vacuum-size=500M

# entfernt alte/unnötige Pakete (macht Platz)
sudo apt autoremove -y

# leert den lokalen Paket-Cache (macht Platz)
sudo apt clean
```

---

## 3) Kleine Sicherung wichtiger Dateien

```bash
# legt einen Backup-Ordner an (damit wir im Notfall vergleichen/zurückbauen können)
sudo mkdir -p /root/sicherheitssystem-backup

# sichert die SSH-Konfiguration (wichtigster Zugang!)
sudo cp /etc/ssh/sshd_config /root/sicherheitssystem-backup/

# sichert Basis-Systemdateien (falls vorhanden)
sudo cp /etc/fstab /root/sicherheitssystem-backup/ 2>/dev/null
sudo cp /etc/sysctl.conf /root/sicherheitssystem-backup/ 2>/dev/null
```

---

## 4) System aktualisieren

```bash
# holt aktuelle Paketlisten
sudo apt update

# installiert Updates (Sicherheitsupdates + Bugfixes)
sudo apt upgrade -y

# räumt alte Pakete weg
sudo apt autoremove -y
```

---

## 5) Automatische Sicherheitsupdates aktivieren

```bash
# installiert automatischen Update-Dienst
sudo apt install -y unattended-upgrades

# aktiviert automatische Sicherheitsupdates (Dialog, aber ohne viel Technik)
sudo dpkg-reconfigure -plow unattended-upgrades

# prüft, ob der Dienst läuft
sudo systemctl is-active unattended-upgrades
```

Erwartet: `active`.

---

## Ergebnis-Check (kurz)

- Snapshot: ✅
- Disk < 80%: ✅
- Updates installiert: ✅
- Auto-Updates aktiv: ✅
