# Enterprise Multi-OS Wazuh SIEM Lab: Automated Active Response & Privilege Escalation Auditing

## 📌 Project Overview
This project documents the end-to-end engineering, deployment, and optimization of an enterprise-grade Wazuh SIEM/XDR telemetry pipeline across a distributed network. The laboratory environment features a central Wazuh Manager orchestration node built on Ubuntu Server, capturing and analyzing endpoint security data from a dedicated Ubuntu Server Agent node. 

The entire detection array was managed, monitored, and stress-tested using a physical Windows 11 host system acting as both the primary administrative Control Center and the external network attacker profile.

Beyond basic installation, this case study details advanced infrastructure troubleshooting—specifically resolving critical Java-backend storage watermark blocks via live Linux Logical Volume Manager (LVM) partition scaling. Finally, the pipeline was validated by engineering a real-time automated Active Response firewall block against automated network attacks and tracking high-severity insider threat privilege escalation attempts.

---

## 🛠️ Core Skills & Technologies Demonstrated
* **SIEM/XDR Architecture:** Distributed deployment of Wazuh Manager, Wazuh Indexer core database, and Wazuh Dashboard analytics.
* **SOAR & Active Response Automation:** Configuring automatic host containment parameters via `iptables` rulesets pushed dynamically from the manager core.
* **Linux Systems Engineering:** Live LVM partition extensions, advanced Systemd service debugging, process tracking, and storage ceiling mitigation.
* **Endpoint Detection Engineering:** Auditing high-frequency SSH authentication brute-force vectors, internal shell command analysis (`journald` parsing), and MITRE ATT&CK framework mapping.

---

## 🏗️ Lab Architecture & Network Topology
The environment utilizes a routeable Bridged Network topology to ensure transparent network packet delivery and accurate host source-IP tracking across the logging pipeline.
```text

       [ Windows 11 Host ]  ─────── (Manages UI Browser & Runs Attacks)
      (IP: 192.168.172.146)
                │
                │ (Launches SSH Brute-Force Loop)
                ▼
  [ Ubuntu Linux Agent (Linux001) ] 
     (Agent IP: 192.168.172.11)
                │
                │ (Ships Encrypted Telemetry via Port 1514)
                ▼
   [ Ubuntu Wazuh Manager (x) ]
    (Manager IP: 192.168.172.65)
