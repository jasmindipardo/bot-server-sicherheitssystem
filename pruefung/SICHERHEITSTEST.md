# 🔒 Sicherheitstest (am Ende ausführen)

Dieser Test ist dein „Alles‑auf‑einen‑Blick“‑Check.

Wichtig: Wir prüfen nicht nur Dateien, sondern **was wirklich aktiv läuft**.

---

## Ein Block — einmal laufen lassen

```bash
export PATH="/usr/sbin:/usr/bin:/sbin:/bin:$PATH"

echo "════════════════════════════════════════════"
echo "  BOT SERVER SICHERHEITSSYSTEM — CHECK"
echo "════════════════════════════════════════════"

echo "\n[SSH]"
ss -tlnp | grep sshd
sshd -T 2>/dev/null | grep -E "^port |^permitrootlogin |^passwordauthentication |^allowusers " || true

echo "\n[FIREWALL]"
sudo ufw status | grep -v "(v6)"

echo "\n[FAIL2BAN]"
systemctl is-active fail2ban
sudo fail2ban-client status 2>/dev/null

echo "\n[ÖFFENTLICHE DIENSTE] (sollte leer sein außer SSH/Webserver)"
ss -tlnp | grep "0.0.0.0" | grep -vE "127\.0\.0|sshd|caddy|nginx|tailscale|systemd-resolve"

echo "\n[KERNEL-WERTE]"
echo "ASLR: $(sysctl -n kernel.randomize_va_space 2>/dev/null) (soll 2)"
echo "dmesg: $(sysctl -n kernel.dmesg_restrict 2>/dev/null) (soll 1)"
echo "kptr: $(sysctl -n kernel.kptr_restrict 2>/dev/null) (soll 1)"
echo "syncookies: $(sysctl -n net.ipv4.tcp_syncookies 2>/dev/null) (soll 1)"
echo "redirects: $(sysctl -n net.ipv4.conf.all.accept_redirects 2>/dev/null) (soll 0)"
echo "swappiness: $(sysctl -n vm.swappiness 2>/dev/null) (soll 10)"
echo "ipv6: $(sysctl -n net.ipv6.conf.all.disable_ipv6 2>/dev/null) (soll 1)"

echo "\n[SWAP]"
swapon --show
free -h | grep Swap

echo "\n[RAM-LIMITS]"
for svc in $(systemctl list-units --type=service --state=running | grep openclaw | awk '{print $1}'); do
  echo "Service: $svc"
  systemctl show "$svc" 2>/dev/null | grep -E "MemoryMax=|MemoryHigh=" | head -2
done

echo "\n[MONITORING]"
if [ -f /usr/local/bin/sicherheitscheck.sh ]; then
  /usr/local/bin/sicherheitscheck.sh
else
  echo "(kein sicherheitscheck.sh gefunden)"
fi
cat /etc/cron.d/sicherheitscheck 2>/dev/null | head -2 || echo "(kein cronjob gefunden)"

echo "\n[AUTO-UPDATES]"
systemctl is-active unattended-upgrades 2>/dev/null || echo "(nicht aktiv)"

echo "\n[DISK]"
df -h / | awk 'NR==2{print $3" belegt / "$2" gesamt ("$5")"}'

echo "\n════════════════════════════════════════════"
echo "  CHECK FERTIG"
echo "════════════════════════════════════════════"
```

---

## Was „gut“ aussieht (Kurzfassung)

- SSH läuft **nicht** auf Port 22
- Root-Login aus, Passwort-Login aus
- Firewall aktiv, nur SSH + ggf. 80/443
- Fail2Ban aktiv (sshd + recidive)
- Keine komischen öffentlichen Ports
- Swap vorhanden
- RAM-Limits gesetzt
- Monitoring läuft
