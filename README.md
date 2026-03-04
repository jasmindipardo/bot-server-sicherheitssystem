# 🔒 Bot Server Sicherheitssystem

Du willst, dass dein KI‑Assistent auf einem Server läuft – aber bitte so, dass du nachts ruhig schlafen kannst.

Dieses Repository ist eine **leicht verständliche Sicherheits-Routine**: Dein Assistent arbeitet sie Schritt für Schritt ab. Du musst nur „okay, weiter“ sagen.

---

## Wie du es benutzt (copy & paste)

Schick deinem Assistenten diese Nachricht:

```
Bitte sichere meinen Server ab.
Nutze dafür das Bot Server Sicherheitssystem: https://github.com/jasmindipardo/bot-server-sicherheitssystem

Wichtig:
- Erkläre mir vor JEDEM Schritt kurz, was du gleich machst (ohne Fachchinesisch).
- Warte jedes Mal auf mein OK.
- Modul 6 (VPN) lassen wir erstmal aus.
```

---

## Die 6 Module (übersichtlich – aber vollständig)

1) **Grundsicherung**
- Snapshot/Backup, Festplatte prüfen, Updates, automatische Sicherheitsupdates

2) **Zugangskontrolle**
- Admin‑Benutzer statt Root, SSH sicher machen (Port, Keys, keine Passwörter)

3) **Netzwerk**
- Firewall „nur das Nötigste“, plus automatischer Schutz gegen Login‑Angriffe

4) **Systemhärtung**
- unnötige Dienste schließen, Betriebssystem absichern, Swap + RAM‑Grenzen

5) **Überwachung**
- 15‑Minuten‑Check (RAM/Swap/Disk), Log‑Begrenzung, Login‑Protokoll

6) **VPN‑Tunnel (optional)**
- SSH unsichtbar machen (nur noch über VPN)

Zum Schluss: **Sicherheitstest** (ein Block, der alles auf einmal prüft).

---

## Bevor du startest

- Erstelle beim Hosting-Anbieter einen **Snapshot** (Hetzner/Netcup/…)
- Du brauchst **SSH-Zugang** (z.B. via VS Code)
- Plane **30–45 Minuten** ein

---

## Mini-Regeln für deinen Assistenten

- Nie „blind“ ausführen: erst erklären, dann auf OK warten.
- Bei SSH-Änderungen: alte Verbindung offen lassen, neue separat testen.
- Wenn ein Neustart nötig ist: vorher ankündigen.

---

## Dateien (klassische Ansicht)

Arbeite die Dateien **von oben nach unten** durch:

```text
bot-server-sicherheitssystem/
├── README.md                      # Start hier
├── LICENSE                        # Lizenz (CC BY-NC)
├── schritte/
│   ├── 01-grundsicherung.md       # Backup + Updates
│   ├── 02-zugangskontrolle.md     # Admin + SSH absichern
│   ├── 03-netzwerk.md             # Firewall + Fail2Ban
│   ├── 04-systemhaertung.md       # Dienste, Kernel, Swap, RAM-Limits
│   ├── 05-ueberwachung.md         # 15-Min-Check + Logs
│   └── 06-vpn.md                  # Optional: VPN-Tunnel
└── pruefung/
    └── SICHERHEITSTEST.md         # Abschluss: alles prüfen
```

---

## Lizenz

**CC BY-NC 4.0** (nicht kommerziell nutzbar).

---

Ein Projekt von **Jasmin Di Pardo** — Glücklich & gebucht Mentoring für Kreativ‑Freelancer.
