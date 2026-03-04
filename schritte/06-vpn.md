# Modul 6 — VPN-Tunnel (optional)

**Ziel:** SSH ist nicht mehr „öffentlich sichtbar“.

**Gefahrlevel:** 🔴 fortgeschritten (kann dich aussperren, wenn du es falsch machst).

Wenn du unsicher bist: Dieses Modul einfach weglassen.

---

## 1) Tailscale auf dem Server

```bash
# installiert Tailscale (VPN)
curl -fsSL https://tailscale.com/install.sh | sh

# verbindet den Server mit deinem Tailscale-Account (öffnet Login-Link)
sudo tailscale up
```

Danach bekommst du einen Link zur Anmeldung.

---

## 2) Tailscale auf deinem Computer

Download: https://tailscale.com/download

Mit dem gleichen Account anmelden.

---

## 3) Test: private IP

```bash
# zeigt die private VPN-IP (100.x.x.x)
tailscale ip -4
```

Jetzt per SSH über die 100.x.x.x IP testen:

```bash
# Test: SSH über VPN-IP (statt öffentliche IP)
ssh -p 2222 admin@100.X.X.X
```

---

## 4) (Optional) SSH nur noch über VPN erlauben

⚠️ Erst machen, wenn der Test oben klappt.

```bash
# entfernt öffentliches SSH
sudo ufw delete allow 2222/tcp

# erlaubt SSH nur noch über das Tailscale-Interface
sudo ufw allow in on tailscale0 to any port 2222 proto tcp comment "SSH nur via VPN"
```

---

## Ergebnis-Check (kurz)

- SSH via Tailscale-IP klappt ✅
- Optional: 2222 nicht mehr öffentlich ✅
