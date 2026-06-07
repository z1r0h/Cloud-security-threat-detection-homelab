# 🛡️ Splunk Threat Detection Lab

A home lab built on GCP to simulate real attacks and practice threat detection using Splunk SIEM.

---

## 🖥️ Lab Architecture

```mermaid
graph TD
    Internet((🌐 Internet))
    IAP["🔒 Google IAP Tunnel\n35.235.240.0/20"]

    Internet --> IAP

    IAP -->|VNC| Kali
    IAP -->|RDP :3389| WinDC
    IAP -->|RDP :3389| WinClient
    IAP -->|HTTP :8000| Splunk

    subgraph GCP ["☁️ GCP — security-lab-vpc (10.0.10.0/24)"]
        WinDC["🖥️ win-dc · 10.0.10.10\nAD Domain Controller\nWindows Server 2022"]
        WinClient["💻 win-client · 10.0.10.20\nEmployee Workstation\nWindows Server 2022"]
        Splunk["📊 splunk-server · 10.0.10.50\nSplunk Enterprise\nUbuntu 22.04"]
        Kali["🐉 kali-linux-attacker-vm · 10.0.10.3\nAttack Machine\nKali Linux"]
    end

    WinClient -->|Join Domain| WinDC
    WinDC -->|Sysmon logs :9997| Splunk
    WinClient -->|Sysmon logs :9997| Splunk
    Kali -->|Attack Traffic| WinDC
    Kali -->|Attack Traffic| WinClient
```

All machines are in a private VPC (`security-lab-vpc`) with no public exposure.  
Access is via **Google IAP Tunnel** only.

---

## ⚙️ Tech Stack

| Component | Tool |
|---|---|
| Cloud | Google Cloud Platform (GCP) |
| SIEM | Splunk Enterprise |
| Log Collection | Sysmon + Splunk Universal Forwarder |
| AD Environment | Windows Server 2022 Active Directory |
| Attack Machine | Kali Linux (GUI via VNC) |
| Secure Access | Google Identity-Aware Proxy (IAP) |

---

## 📁 Repo Structure

```
├── docs/setup/          # Step-by-step lab setup guides
├── attacks/             # Attack scenarios (commands + screenshots)
├── detection/           # Splunk SPL queries and alert rules
├── scripts/             # Helper scripts (VM recovery, etc.)
└── assets/              # Architecture diagrams
```

---

## 🔴 Attack Scenarios (Coming Soon)

| # | Scenario | MITRE Technique | Status |
|---|---|---|---|
| 01 | Credential Dumping (Mimikatz) | T1003 | 🔜 |
| 02 | Pass-the-Hash Lateral Movement | T1550.002 | 🔜 |
| 03 | Persistence via Scheduled Task | T1053.005 | 🔜 |
| 04 | C2 Beacon Simulation | T1071 | 🔜 |
| 05 | Brute Force RDP | T1110.001 | 🔜 |

---

## 🔵 Detection (Coming Soon)

Each attack will have a matching Splunk SPL query and alert rule documented in `detection/`.

---

## 🚀 Setup

See [`docs/setup/`](docs/setup/) for full step-by-step instructions.

**Quick overview:**
1. Create VPC + firewall rules (IAP + internal traffic)
2. Spin up 4 VMs (win-dc, win-client, splunk-server, kali-attacker)
3. Install Splunk on Ubuntu, configure port 9997 receiver
4. Promote win-dc to AD Domain Controller (`200teamok.local`)
5. Join win-client to domain
6. Install Sysmon + Splunk Universal Forwarder on both Windows VMs
7. Snapshot all VMs as clean baseline images

---

## 🔄 VM Recovery

One-command restore to clean baseline:

```bash
# Restore win-client
gcloud compute instances delete win-client --zone=asia-southeast1-a --quiet && \
gcloud compute instances create win-client --source-machine-image=winclient-clean --zone=asia-southeast1-a
```

See [`scripts/recovery/`](scripts/recovery/) for all VMs.

---

## ⚠️ Disclaimer

This lab is for **educational purposes only**.  
All credentials shown in setup docs are lab-only and should never be used in production.

---
```mermaid
---
config:
  layout: dagre
  theme: dark
  look: neo
  fontFamily: '''Merriweather Variable'', serif'
---
flowchart TB
 subgraph AD["🏢 Active Directory Environment"]
    direction TB
        WinDC["🖥️ win-dc<br>10.0.10.10<br>AD Domain Controller"]
        WinClient["💻 win-client<br>10.0.10.20<br>Employee Workstation"]
  end
 subgraph GCP["☁️ GCP — security-lab-vpc (10.0.10.0/24)"]
    direction LR
        AD
        Kali["🐉 kali-linux-attacker-vm<br>10.0.10.3<br>Kali Linux"]
        Splunk["📊 splunk-server<br>10.0.10.50<br>Splunk Enterprise"]
  end
    Internet(("🌐 Internet")) --> IAP["🔒 Google IAP Tunnel<br>35.235.240.0/20"]
    IAP -. VNC .-> Kali
    IAP -. RDP :3389 .-> WinDC & WinClient
    IAP -. HTTP :8000 .-> Splunk
    WinClient <-- Join Domain --> WinDC
    WinDC -. Sysmon logs :9997 .-> Splunk
    WinClient -. Sysmon logs :9997 .-> Splunk
    Kali == Attack Traffic ==> WinDC & WinClient

     WinDC:::windows
     WinClient:::windows
     Kali:::attacker
     Splunk:::splunk
     Internet:::internet
     IAP:::iap
    classDef internet fill:#f0f0f0,stroke:#9e9e9e,stroke-width:2px,color:#000
    classDef iap fill:#fff8e1,stroke:#fbc02d,stroke-width:2px,color:#333
    classDef windows fill:#e8f0fe,stroke:#1a73e8,stroke-width:2px,color:#0d47a1
    classDef attacker fill:#fdecea,stroke:#d93025,stroke-width:2.5px,color:#8b0000
    classDef splunk fill:#e6f4ea,stroke:#0f9d58,stroke-width:2px,color:#0b8043
    classDef network fill:#f8f9fa,stroke:#4285f4,stroke-width:1.5px,stroke-dasharray: 5 5
    style AD fill:#eef4ff,stroke:#5a9bfc,stroke-width:1px
    style GCP fill:#fff8e1,stroke:#2962FF,stroke-width:1.5px,stroke-dasharray: 5 5
    linkStyle 1 stroke:#000000
```

## 📜 Author

**Jeremy Lim**  
Cybersecurity Enthusiast | SOC Analyst (aspiring)  
[LinkedIn](https://www.linkedin.com/in/jeremy-lzh/) · [GitHub](https://github.com/z1r0h)
