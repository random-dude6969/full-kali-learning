# Kismet Cheatsheet

## Quick Start

```bash
# Install
sudo apt update && sudo apt install kismet

# Start with interface
sudo kismet -c wlan0

# Open web UI
# https://localhost:2501
```

## Essential Commands

| Command | Description |
|---------|-------------|
| `sudo kismet -c wlan0` | Start with default interface |
| `sudo kismet -c wlan0:channel=6` | Lock to channel 6 |
| `sudo kismet -c wlan0:channels=1,6,11` | Scan specific channels |
| `sudo kismet -c wlan0:hop=10` | Fast channel hopping (10/sec) |
| `sudo kismet -c wlan0 --no-gui` | Headless/daemon mode |
| `sudo kismet -c wlan0 --daemonize` | Run as background daemon |
| `sudo kismet -c wlan0 --use-gpsd` | Enable GPS |
| `sudo kismet -c wlan0 -w capture` | Log with custom prefix |
| `sudo kismet -c wlan0 --log-types kismet,pcap` | Control log types |
| `sudo kismet --server-host 0.0.0.0` | Bind to all interfaces (remote access) |

## Datasource Syntax

```
source=interface:name=Label:channel=X:hop=Y
```

| Parameter | Example | Description |
|-----------|---------|-------------|
| `name` | `name=Office` | Human-readable name |
| `channel` | `channel=6` | Lock channel |
| `channels` | `channels=1,6,11` | Scan list |
| `hop` | `hop=5` | Hop rate (channels/sec) |
| `uuid` | `uuid=abc123` | Source UUID |

```bash
# Multiple sources
sudo kismet -c wlan0:name=2.4G -c wlan1:name=5G

# Named source on specific channels
sudo kismet -c wlan0:name=Monitor:channels=1,6,11
```

## Log Types

| Type | Format | Use |
|------|--------|-----|
| `kismet` | SQLite DB | Primary analysis with kismet_db |
| `pcap` | PCAP | Wireshark/tshark analysis |
| `netxml` | XML | Legacy compatibility |
| `netjson` | JSON | Scripting/integration |
| `gps` | GPS log | Geographic mapping |

```bash
# kismet_db queries
kismet_db -r capture.kismetdb -l devices
kismet_db -r capture.kismetdb -l devices -o devices.csv
kismet_db -r capture.kismetdb -p output.pcap
kismet_db -r capture.kismetdb -l alerts
```

## GPS Setup

```bash
# Start GPSD
sudo gpsd /dev/ttyUSB0 -F /var/run/gpsd.sock

# Test GPS
gpspipe -w -n 10

# Start Kismet with GPS
sudo kismet -c wlan0 --use-gpsd
```

## Web UI

| URL | Description |
|-----|-------------|
| `https://localhost:2501` | Main UI |
| `https://localhost:2501/devices` | Device list |
| `https://localhost:2501/alerts` | Alert list |
| `https://localhost:2501/map` | GPS map |

## REST API Examples

```bash
# Get devices
curl -sk https://localhost:2501/devices/views/all/devices.json

# Get alerts
curl -sk https://localhost:2501/alerts/all_alerts.json

# Get access points
curl -sk https://localhost:2501/devices/views/accesspoints/devices.json

# Get status
curl -sk https://localhost:2501/system/status.json
```

## Common Alerts

| Alert | Meaning |
|-------|---------|
| `DOT11_DEAUTHDOT11` | Deauth flood attack |
| `DOT11_DISCON` | Disconnection flood |
| `DOT11_EAPOL` | EAPOL/authentication attack |
| `PROBENORESP` | Probe request storm |
| `NETWRAP` | Network wrap attack |

## Troubleshooting

```bash
# Interface not found
sudo ip link set wlan0 down
sudo iw wlan0 set type monitor
sudo ip link set wlan0 up

# Permission denied
sudo kismet -c wlan0

# SSL errors (delete and regenerate)
sudo rm /etc/kismet/kismet_httpd-ssl.*
sudo systemctl restart kismet

# GPS not working
sudo gpsd /dev/ttyUSB0 -F /var/run/gpsd.sock
sudo kismet -c wlan0 --use-gpsd

# High CPU — reduce hop rate
sudo kismet -c wlan0:hop=2
```
