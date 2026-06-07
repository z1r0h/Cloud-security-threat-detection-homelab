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
graph TB
    %% 1. 定义不同角色的颜色主题 (类)
    classDef internet fill:#e0e0e0,stroke:#9e9e9e,stroke-width:2px,color:#000
    classDef iap fill:#fbbc04,stroke:#f29900,stroke-width:2px,color:#000
    classDef attacker fill:#fce8e6,stroke:#ea4335,stroke-width:2px,color:#b31412
    classDef windows fill:#e8f0fe,stroke:#1a73e8,stroke-width:2px,color:#0d47a1
    classDef splunk fill:#e6f4ea,stroke:#34a853,stroke-width:2px,color:#0b8043

    %% 2. 外部网络与入口
    Internet(("🌐 Internet")):::internet
    IAP["🔒 Google IAP Tunnel<br>35.235.240.0/20"]:::iap

    Internet --> IAP

    %% 3. GCP VPC 内部架构
    subgraph GCP ["☁️ GCP — security-lab-vpc (10.0.10.0/24)"]
        direction TB
        
        Kali["🐉 kali-linux-attacker-vm<br>10.0.10.3<br>Kali Linux"]:::attacker

        %% 将 Windows 机器归类到 AD 环境中，逻辑更紧凑
        subgraph AD ["🏢 Active Directory Environment"]
            direction LR
            WinDC["🖥️ win-dc<br>10.0.10.10<br>AD Domain Controller"]:::windows
            WinClient["💻 win-client<br>10.0.10.20<br>Employee Workstation"]:::windows
        end

        Splunk["📊 splunk-server<br>10.0.10.50<br>Splunk Enterprise"]:::splunk
    end

    %% 4. 设置 Subgraph (背景框) 的样式
    style GCP fill:#f8f9fa,stroke:#4285f4,stroke-width:2px,stroke-dasharray: 5 5
    style AD fill:#ffffff,stroke:#8ab4f8,stroke-width:1px

    %% 5. IAP 远程访问连接 (使用虚线 -.-> 表示管理连接)
    IAP -.->|VNC| Kali
    IAP -.->|RDP :3389| WinDC
    IAP -.->|RDP :3389| WinClient
    IAP -.->|HTTP :8000| Splunk

    %% 6. 内部正常业务流量 (使用常规双向线 <-->)
    WinClient <-->|Join Domain| WinDC

    %% 7. 日志转发流量 (使用虚线 -.-> 表示后台数据流)
    WinDC -.->|Sysmon logs :9997| Splunk
    WinClient -.->|Sysmon logs :9997| Splunk

    %% 8. 攻击流量 (使用加粗线条 ==> 突出显示)
    Kali ==>|Attack Traffic| WinDC
    Kali ==>|Attack Traffic| WinClient
```

## 📜 Author

**Jeremy Lim**  
Cybersecurity Enthusiast | SOC Analyst (aspiring)  
[LinkedIn](https://www.linkedin.com/in/jeremy-lzh/) · [GitHub](https://github.com/z1r0h)
