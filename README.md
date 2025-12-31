# Acces-Point-Raspap
Raspberry Pi Wi-Fi setup hotspot: creates an AP when not connected, lets users configure Wi-Fi from a phone, then disables the hotspot.

1- Create the hotspot connection
```
sudo nmcli con add type wifi ifname wlan0 con-name setup-hotspot autoconnect no ssid "Connection Name"
```
```
sudo nmcli con modify hotspot 802-11-wireless.mode ap
```
```
sudo nmcli con modify hotspot 802-11-wireless.band bg 802-11-wireless.channel 6
```
```
sudo nmcli con modify hotspot wifi-sec.key-mgmt wpa-psk wifi-sec.psk "Password"
```
```
sudo nmcli con modify hotspot ipv4.method shared ipv6.method ignore
```
2- Add the script
```
sudo nano /usr/local/sbin/wifi-setup-mode.sh
```
```
#!/usr/bin/env bash
set -euo pipefail

IFACE="wlan0"
HOTSPOT="hotspot" //Change this if u also changed the name

ACTIVE="$(nmcli -t -f NAME,DEVICE con show --active | awk -F: -v d="$IFACE" '$2==d{print $1; exit}')"

if [[ -n "${ACTIVE:-}" && "$ACTIVE" != "$HOTSPOT" ]]; then
  nmcli con down "$HOTSPOT" >/dev/null 2>&1 || true
  exit 0
fi

if [[ "${ACTIVE:-}" == "$HOTSPOT" ]]; then
  exit 0
fi

nmcli con up "$HOTSPOT"
```

3- Make it executable
```
sudo chmod +x /usr/local/sbin/wifi-setup-mode.sh
```
4- Create the systemd service
```
sudo nano /etc/systemd/system/wifi-setup-mode.service
```

```
[Unit]
Description=Enable setup hotspot when Wi-Fi is not connected
After=NetworkManager.service
Wants=NetworkManager.service

[Service]
Type=oneshot
ExecStart=/usr/local/sbin/wifi-setup-mode.sh

5-Create the systemd timer

sudo nano /etc/systemd/system/wifi-setup-mode.timer
[Unit]
Description=Periodically ensure setup hotspot state

[Timer]
OnBootSec=10
OnUnitActiveSec=15
Unit=wifi-setup-mode.service

[Install]
WantedBy=timers.target
```

6-Enable ans start
```
sudo systemctl daemon-reload
```
```
sudo systemctl enable --now wifi-setup-mode.timer
```

EXTRA:

Get status:
```
systemctl status wifi-setup-mode.timer --no-pager
```
```
journalctl -u wifi-setup-mode.service -n 50 --no-pager
```
Change to static IP:
```
sudo nmcli con mod "setup-hotspot" ipv4.method manual
```
```
sudo nmcli con mod "setup-hotspot" ipv4.addresses yourIP
```
```
sudo nmcli con mod "setup-hotspot" ipv4.gateway YourGateway
```
```
sudo nmcli con mod "setup-hotspot" ipv4.dns "1.1.1.1 8.8.8.8"
```
sudo nmcli con mod "setup-hotspot" ipv6.method ignore

