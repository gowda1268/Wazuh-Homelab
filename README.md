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







🚀 Phase 1: Distributed Installation Lifecycle
1. Central Manager Provisioning

The central orchestration node was stood up on an Ubuntu Server environment using the automated enterprise installation architecture pipeline:

curl -sO [https://packages.wazuh.com/4.x/wazuh-install.sh](https://packages.wazuh.com/4.x/wazuh-install.sh) && sudo bash wazuh-install.sh -a


2. Linux Endpoint Enrollment (Linux001)

The separate Ubuntu Server endpoint agent was configured to securely register and bind its telemetry stream to the manager's listening socket:
# Download and install the official Wazuh repository GPG key
wget -qO - [https://packages.wazuh.com/key/GPG-KEY-WAZUH](https://packages.wazuh.com/key/GPG-KEY-WAZUH) | gpg --dearmor -o /usr/share/keyrings/wazuh.gpg

# Add the repository list entry
echo "deb [signed-by=/usr/share/keyrings/wazuh.gpg] [https://packages.wazuh.com/4.x/apt/](https://packages.wazuh.com/4.x/apt/) stable main" | sudo tee /etc/apt/sources.list.d/wazuh.list

# Update package lists and install the agent pointed to the Manager IP
sudo apt-get update && WAZUH_MANAGER='192.168.172.65' sudo apt-get install wazuh-agent

# Enable and start the background daemon service
sudo systemctl daemon-reload
sudo systemctl enable wazuh-agent
sudo systemctl start wazuh-agent




🛑 Phase 2: Infrastructure Engineering & Storage Resolution
1. Diagnosing Storage Watermark Freezes (OSError: [Errno 28])

During infrastructure initialization, the wazuh-manager.service failed to start. Forcing the background process into the foreground (sudo /var/ossec/bin/wazuh-apid -f) revealed a fatal exception: OSError: [Errno 28] No space left on device.

Running df -h confirmed that the root logical volume was pinned at 100% capacity. This storage ceiling hit the Wazuh Indexer's (OpenSearch) protective Flood-Stage Disk Watermark (95%), locking down all database write operations and triggering infinite boot loops.
2. Live LVM Volume Expansion

To resolve this issue gracefully without data loss or service degradation, Linux Logical Volume Manager (LVM) utilities were leveraged to dynamically allocate virtual hardware storage on-the-fly without requiring a system restart:


# Evaluate available physical group boundaries to confirm unallocated sector space
sudo vgs

# Extend the underlying logical file allocation to absorb 100% of newly exposed space
sudo lvextend -l +100%FREE -r /dev/mapper/ubuntu--vg-ubuntu--lv


⚡ Phase 3: Detection Engineering & Security Scenarios



Scenario 1: Automated SSH Brute-Force & Active Response Containment

An aggressive SSH brute-force simulation loop was launched from the Windows Host machine terminal targeting the Ubuntu Linux Agent node:

# Executed from Windows Host PowerShell (192.168.172.146)
for ($i=1; $i -le 10; $i++) { ssh fakeuser@192.168.172.11 }

🛠️ Active Response Rule Optimization

To prevent service crashes, /var/ossec/etc/ossec.conf on the Wazuh Manager was optimized to handle local containment blocks dynamically:

<active-response>
  <command>firewall-drop</command>
  <location>local</location>
  <level>5</level>
</active-response>


The pipeline successfully identified the high-frequency failure pattern originating from the Windows host and automatically dropped the attacking connections. Running iptables -L INPUT -v -n on the agent verified live firewall bans dropping traffic from 192.168.172.146.

📊 Ingestion & Detection Verification

The screenshot below displays the Wazuh Dashboard capturing the transition from individual low-severity authentication warnings to an aggregated, high-severity Level 10 alert. This demonstrates Wazuh's correlation engine recognizing the rapid-fire nature of the network brute-force attack originating from the Windows host.



Scenario 2: Host Privilege Escalation (Sudo Abuse)

To validate internal insider-threat tracking capabilities, an intentional privilege escalation attempt was simulated directly on the agent terminal by purposefully forcing high-severity administrative authorization failures via the sudo boundary:

sudo -k && sudo ls /root
# (Deliberately inputted incorrect credentials 3 consecutive times)

📊 Administrative Log Audit

The screenshot below shows the raw telemetry ingested from the agent's system logs. Wazuh successfully parsed the failed administrative tracking event, highlighted the precise user account (x2) executing the command, raised it to a Level 10 alert severity, and accurately mapped the event behavior to MITRE ATT&CK Technique T1548.003.





Scenario 3: Local Account Tampering (File Integrity Monitoring)

To verify real-time monitoring of critical files, real-time tracking attributes were appended to the <syscheck> section inside the agent's ossec.conf:


<directories realtime="yes" report_changes="yes">/etc</directories>
An adversary simulation was executed on the agent by forcing an unauthorized local account creation:
sudo useradd hacker_test

📊 FIM Real-Time Alert Analytics

The screenshot below captures the real-time File Integrity Monitoring dashboard the moment the unauthorized local user account was created. It maps out the exact timestamps and highlights modifications made directly to the core security files (/etc/passwd and /etc/shadow).




Scenario 4: Continuous Vulnerability Assessment & System Hardening

To identify software weaknesses, the Vulnerability Detector was activated within the manager’s global configuration file:

```xml
<vulnerability-detection>
  <enabled>yes</enabled>
  <index-status>yes</index-status>
  <feed-update-interval>60m</feed-update-interval>
</vulnerability-detection>



📊 Vulnerability Inventory & Compliance Card

The screenshots below provide the final state of the audited endpoint. The first highlights the automated vulnerability inventory index breaking down unpatched host packages by CVE severity. The second shows the Security Configuration Assessment (SCA) final compliance scorecard reflecting successful operating system hardening against CIS standards.

Additionally, Wazuh executed continuous Center for Internet Security (CIS) benchmarks against the Ubuntu agent endpoint to evaluate compliance metrics. To demonstrate remediation engineering, the system was hardened manually by editing /etc/ssh/sshd_config, setting PermitRootLogin no, and executing sudo systemctl restart ssh. The subsequent audit reflected an immediate increase in compliance scores.


🧹 Phase 4: Database Maintenance & Baseline Cleanliness

To maintain resource-conscious lab operations and safely flush heavy simulation streams before establishing pristine baseline tracking vectors, index administrative cleanups were invoked directly via the index API structures:


curl -k -u "admin":"bxU*ZqDkate.UXM59rhJ0ZdvM.itbYI7" -X DELETE "https://localhost:9200/wazuh-alerts-*"






