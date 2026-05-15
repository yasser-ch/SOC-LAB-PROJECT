# 🛡️ SOC Lab Project — Automated Security Operations Center

> A distributed Next-Generation SOC lab built by a 5-member team, each running their component on a separate machine linked via OpenVPN. This repository documents the **NDR Probe + Wazuh SIEM** component.

---

## 📸 Screenshots

### Global Architecture

<img width="1408" height="768" alt="image (2)" src="https://github.com/user-attachments/assets/81b9928f-228d-4ef7-b65e-a3922ee787b0" />


### Wazuh Dashboard — NDR-Probe Events

<img width="1354" height="691" alt="WhatsApp Image 2026-05-13 at 12 45 33 AM" src="https://github.com/user-attachments/assets/80e933f0-9c64-4770-b772-220fa3edfb61" />

<img width="605" height="483" alt="Image1" src="https://github.com/user-attachments/assets/6322beed-f71a-4c5a-a43f-18e18f91bb31" />

### Arkime — Live Session Capture

<img width="1356" height="603" alt="WhatsApp Image 2026-05-13 at 12 02 56 AM" src="https://github.com/user-attachments/assets/8c0ff910-8cce-4994-9b8f-1f0135651891" />


### Zeek — Active Logs

<img width="580" height="405" alt="Image1" src="https://github.com/user-attachments/assets/ab80db8c-de1a-4700-8774-102d689c3cd2" />


---

## 🗺️ Architecture Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                        SOC Lab Network                          │
│                      VPN Subnet 10.0.8.0/24                     │
│                                                                 │
│  ┌─────────────┐     ┌──────────────────┐                       │
│  │    Kali     │────▶│ pfSense+Suricata │                      │
│  │ 10.0.8.6   │     │   10.0.8.1        │                       │
│  │ (Attacker) │     │ (Firewall/NIDS)   │                       │
│  └─────────────┘     └────────┬─────────┘                       │
│                               │ OpenVPN (tun0)                  │
│                               ▼                                 │
│                  ┌────────────────────────┐                     │
│                  │     NDR Probe          │  ◀── THIS REPO      │
│                  │     10.0.8.8           │                     │
│                  │  Zeek + Arkime         │                     │
│                  │  Filebeat + Wazuh Agent│                     │
│                  └────────────┬───────────┘                     │
│                               │ TCP :5044 / :1514               │
│                               ▼                                 │
│                  ┌────────────────────────┐                     │
│                  │     Wazuh SIEM         │                     │
│                  │     10.0.8.5           │                     │
│                  │  Manager+Indexer       │                     │
│                  │  +Dashboard            │                     │
│                  └────────────┬───────────┘                     │
│                               │                                 │
│              ┌────────────────┴────────────────┐                │
│              ▼                                 ▼                │
│  ┌───────────────────┐           ┌─────────────────────────┐    │
│  │   Shuffle SOAR    │           │  TheHive + Cortex + MISP│    │
│  │   10.0.8.4        │           │  10.0.8.3               │    │
│  └───────────────────┘           └─────────────────────────┘    │
└─────────────────────────────────────────────────────────────────┘
```

---

## 👥 Team

| Member | Component | VPN IP |
|--------|-----------|--------|
| **Yasser chettour** (me) | Wazuh SIEM + NDR Probe | `10.0.8.5` / `10.0.8.8` |
| Mounir Merghich | pfSense + Suricata + OpenVPN Server | `10.0.8.1` |
| Malak Belkhou | Shuffle SOAR + DVWA + Windows 10 Client | `10.0.8.4` |
| Landry Dossah | Kali Linux (attacker) + Cowrie Honeypot | `10.0.8.6` / `10.0.8.10` |
| Hiba Sidinou | TheHive + Cortex + MISP | `10.0.8.3` | [@hibasidinou](https://github.com/hibasidinou) |

---

## 🔧 My Components

### VM 1 — Wazuh SIEM (`10.0.8.5`)

| Component | Role | Port |
|-----------|------|------|
| Wazuh Indexer | Storage & indexation (OpenSearch) | `:9200 HTTPS` |
| Wazuh Manager | Correlation engine & rules | `:1514` / `:5044` |
| Filebeat 7.17 | Log transfer Manager → Indexer | `:5044 TCP` |
| Wazuh Dashboard | Visualization & Threat Hunting | `:443 HTTPS` |

### VM 2 — NDR Probe (`10.0.8.8`)

| Component | Role | Details |
|-----------|------|---------|
| Zeek 8.0.5 | Passive network analysis | Captures on `tun0` |
| Arkime 5.4.0 | Full Packet Capture (PCAP) | Web UI `:8005` |
| OpenSearch | Local storage for Arkime | `localhost:9200` |
| Filebeat 7.17 | Ships Zeek logs → Wazuh | TCP `:5044` |
| Wazuh Agent | Agent ID 004 — NDR-Probe | TCP `:1514` |

---

## 📊 Zeek Log Types

| Log File | Content |
|----------|---------|
| `conn.log` | All TCP/UDP connections |
| `dns.log` | DNS queries and responses |
| `http.log` | HTTP requests, URLs, user-agents |
| `ssl.log` | TLS handshakes, certificates |
| `notice.log` | Zeek security alerts |

---

## 📁 Repository Structure

```
SOC-LAB-PROJECT/
│
├── configs/
│   ├── zeek/
│   │   ├── node.cfg          # Zeek interface config (interface=tun0)
│   │   ├── local.zeek        # Loaded Zeek scripts
│   │   └── zeek.service      # systemd autostart unit
│   │
│   ├── arkime/
│   │   └── config.ini        # Arkime capture + viewer config
│   │
│   ├── wazuh/
│   │   └── ossec.conf        # Wazuh agent config (→ 10.0.8.5)
│   │
│   └── filebeat/
│       └── filebeat.yml      # Filebeat → Wazuh Manager :5044
│
├── docs/
│   └── screenshots/          # Place your screenshots here
│
└── README.md
```

---

## ✅ Validation Results

| Verification | Result |
|---|---|
| Zeek captures traffic on `tun0` | ✅ `conn.log`, `dns.log`, `http.log` generated in real time |
| Filebeat ships logs to Wazuh | ✅ `filebeat test output` → `talk to server... OK` (TCP :5044) |
| Wazuh receives NDR events | ✅ **503 events** indexed — filtered on `agent.name: NDR-Probe` |
| Arkime indexes sessions | ✅ **290+ PCAP sessions** — web UI accessible on `:8005` |
| Wazuh Agent active | ✅ Agent ID **004** — `NDR-Probe` — status: `active (running)` |

---

## 🚀 Quick Start — Services

```bash
# Check all services at once
for s in opensearch arkimecapture arkimeviewer wazuh-agent zeek; do
  echo -n "$s: "; systemctl is-active $s
done

# Restart everything
sudo systemctl restart opensearch
sudo systemctl restart arkimecapture arkimeviewer
sudo systemctl restart wazuh-agent
sudo /opt/zeek/bin/zeekctl deploy
```

### Browser Access (via SSH tunnel from your PC)

```bash
# Arkime UI
ssh -L 8005:localhost:8005 ndr@<VM_LOCAL_IP>
# → open http://localhost:8005

# Wazuh Dashboard
ssh -L 8443:10.0.8.5:443 ndr@<VM_LOCAL_IP>
# → open https://localhost:8443

# pfSense UI
ssh -L 8080:10.0.8.1:80 ndr@<VM_LOCAL_IP>
# → open http://localhost:8080
```

---

## 🛠️ Key Technical Details

- **OS:** Ubuntu Server 22.04 LTS (no GUI)
- **Hypervisor:** VMware Workstation
- **Capture interface:** `tun0` (OpenVPN) — all inter-member traffic flows here
- **Zeek logs location:** `/opt/zeek/logs/current/`
- **Arkime storage:** OpenSearch local on NDR VM `localhost:9200`
- **Wazuh Agent ID:** `004` — name: `NDR-Probe`
- **All services autostart on boot:** `opensearch`, `arkimecapture`, `arkimeviewer`, `wazuh-agent`, `zeek`

---

## 🧱 Tech Stack

![Ubuntu](https://img.shields.io/badge/Ubuntu-22.04-E95420?logo=ubuntu&logoColor=white)
![Zeek](https://img.shields.io/badge/Zeek-8.0.5-0078D4)
![Arkime](https://img.shields.io/badge/Arkime-5.4.0-brightgreen)
![Wazuh](https://img.shields.io/badge/Wazuh-4.7.0-blue)
![OpenSearch](https://img.shields.io/badge/OpenSearch-2.x-orange?logo=opensearch)
![Filebeat](https://img.shields.io/badge/Filebeat-7.17-00BFB3?logo=elastic)
![OpenVPN](https://img.shields.io/badge/OpenVPN-2.5-orange)

---

## 📄 License

This project was built for educational purposes as part of a university cybersecurity program.
