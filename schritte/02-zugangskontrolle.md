# Modul 2 — Zugangskontrolle

**Ziel:** Nur du kommst rein – und zwar auf einem sicheren Weg.

**Gefahrlevel:** 🟡 mittel (hier ändert sich dein Zugang).

**Wichtigste Regel:** Alte SSH‑Verbindung offen lassen. Alles Neue erst in einem zweiten Fenster testen.

---

## Teil A: Admin‑Benutzer statt Root

### 1) Admin anlegen

```bash
# erstellt den Benutzer "admin"
sudo adduser admin

# gibt ihm sudo-Rechte (damit er Admin-Aufgaben darf)
sudo usermod -aG sudo admin
```

### 2) SSH‑Key für admin hinterlegen

```bash
# legt das SSH-Verzeichnis für den admin an
sudo mkdir -p /home/admin/.ssh
sudo chmod 700 /home/admin/.ssh

# übernimmt deinen bestehenden SSH-Schlüssel (damit du dich ohne Passwort anmelden kannst)
sudo cp /root/.ssh/authorized_keys /home/admin/.ssh/ 2>/dev/null

# setzt korrekte Rechte (sonst blockt SSH die Datei)
sudo chown -R admin:admin /home/admin/.ssh
sudo chmod 600 /home/admin/.ssh/authorized_keys
```

### 3) Test (neues Terminal!)

```bash
# Test: kannst du dich als admin einloggen?
ssh admin@SERVER_IP

# Test: funktioniert sudo?
sudo whoami
```

Erwartet: `root`.

---

## Teil B: SSH härten (Port, Root aus, Passwörter aus)

### 4) Backup der SSH‑Config

```bash
# Backup der SSH-Konfiguration (falls wir zurück müssen)
sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.backup
```

### 5) Ubuntu 24.04+: Check ob `ssh.socket` aktiv ist

```bash
# prüft, ob Ubuntu die SSH-Ports über systemd steuert (Ubuntu 24.04+)
systemctl is-active ssh.socket
```

Wenn `active`, Port-Änderung via systemd:

```bash
# legt eine systemd-Override-Datei an, damit SSH auf Port 2222 hört (und nicht zusätzlich auf 22)
sudo mkdir -p /etc/systemd/system/ssh.socket.d
sudo tee /etc/systemd/system/ssh.socket.d/override.conf << 'EOF'
[Socket]
ListenStream=
ListenStream=0.0.0.0:2222
ListenStream=[::]:2222
EOF
```

### 6) SSH‑Einstellungen setzen

```bash
# setzt SSH auf Port 2222, verbietet Root-Login, verbietet Passwörter (nur Key)
sudo tee -a /etc/ssh/sshd_config << 'EOF'

# ── Bot Server Sicherheitssystem ──
Port 2222
PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes
MaxAuthTries 3
LoginGraceTime 20
ClientAliveInterval 300
ClientAliveCountMax 2
AllowUsers admin
X11Forwarding no
PermitEmptyPasswords no
EOF
```

### 7) Neustart + Test

```bash
# lädt systemd neu (falls wir socket/override geändert haben)
sudo systemctl daemon-reload

# startet SSH neu (damit die neuen Einstellungen aktiv werden)
sudo systemctl restart sshd
```

Jetzt im neuen Terminal testen:

```bash
# Test: neuer Port + neuer Benutzer
ssh -p 2222 admin@SERVER_IP
```

---

## Teil C: VS Code/SSH‑Profil anpassen

Im lokalen SSH‑Config‑File so (Beispiel):

```
Host mein-server
  HostName DEINE_SERVER_IP
  User admin
  Port 2222
  IdentityFile ~/.ssh/id_ed25519
```

---

## Ergebnis-Check (kurz)

- Login als `admin` klappt ✅
- Port 22 wird nicht mehr genutzt ✅
- Root-Login aus ✅
- Passwort-Login aus ✅
