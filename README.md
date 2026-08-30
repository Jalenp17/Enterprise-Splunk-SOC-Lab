# Enterprise Blue Team Lab: Active Directory Telemetry Ingestion & Splunk SIEM Detection

## Project Overview
This project demonstrates the design, deployment, and optimization of a centralized log management pipeline using **Splunk Enterprise** to detect and triage automated credential attacks within an Active Directory environment. 

I simulated real-world cyber threat vectors from an isolated attack platform, engineered custom endpoint forwarding rules, and authored high-fidelity Search Processing Language (SPL) threshold analytics to isolate malicious activity from normal human authentication errors.

## Core Lab Architecture
* **SIEM Platform:** Splunk Enterprise hosted on an Ubuntu Server virtual machine.
* **Target Infrastructure:** Windows Server 2022 configured as an Active Directory Domain Controller.
* **Attack Platform:** Kali Linux 2026.
* **Log Ingestion Pipeline:** Splunk Universal Forwarder deployed on the target Windows system, handling text-based Windows Security event logs mapped directly into the default `main` index.

---

## Technical Phase Execution

### Phase 1: Threat Emulation (Red Team Action)
Using a custom user list (`users.txt`) and a targeted password list (`passwork.txt`) compiled on the Kali Linux Desktop, I launched a single-threaded automated brute-force attack against the Windows Domain Controller over the Remote Desktop Protocol (RDP / Port 3389).

```bash
# Executing the automated credential-guessing attack from the Desktop folder
cd ~/Desktop
hydra -L users.txt -P passwork.txt rdp://[TARGET_WINDOWS_IP] -t 1 -V -f
```

*Below is the execution window capturing the high-velocity credential attack vector:*
![Kali Linux Attack](screenshots/kali_attack.png)

---

### Phase 2: Ingestion & Telemetry Engineering (Blue Team Action)
To ensure the Windows event logs were readable by the Splunk core engine, I customized the endpoint's ingestion configurations. By setting `renderXml = false` in the forwarder's properties, Windows raw data was transmitted as structured key-value text pairs, allowing Splunk to natively extract authentication headers.

*Below is the verification of the raw Event Code 4625 logs landing securely in the SIEM index:*
![Raw Event Evidence](screenshots/raw_events.png)

---

### Phase 3: Analytical Development (Detection Engineering)
I authored an optimized, interview-proof SPL query designed to aggregate raw authentication telemetry while actively suppressing alert noise. 

```spl
index="main" EventCode=4625
| rename Source_Network_Address AS sc_ip, 
| stats count by Account_Name, sc_ip
| where count > 10
```

#### **Query Breakdown:**
1. **`index="main" EventCode=4625`**: Pulls Windows Security events tracking explicit logon failures.
2. **`| rename...`**: Standardizes unindexed system tags into clean, human-readable data properties (`sc_ip` and `Account_Name`).
3. **`| stats count by...`**: Groups multi-dimensional fields together to immediately correlate exactly *who* was targeted right next to *where* the attack came from.
4. **`| where count > 10`**: Establishes a strict behavioral threshold. It ignores standard employee typos but flags automated automation tools (like Hydra) the moment they exceed 10 rapid attempts.

*Below is the visual confirmation of the optimized SPL search yielding clear metadata fields:*
![SPL Query In Action](screenshots/spl_query.png)

---

## Operational Deliverables (SOC Operations)

### 1. Custom Security Operations Center Dashboard
I saved the optimized SPL query as a persistent visual tracking panel using Splunk's Classic Dashboard studio. This workspace gives internal tier-1 analysts immediate situational visibility over active authentication threats across the domain infrastructure.

*Below is the live Enterprise SOC Analyst Workspace displaying the attack metrics:*
![Splunk SOC Dashboard](screenshots/splunk_dashboard.png)

### 2. Automated Trigger Alert Queue
To ensure proactive network defense when analysts are away from the dashboard console, I configured the threshold code to run as a **Scheduled Alert Rule** utilizing background cron intervals (`*/5 * * * *`). The moment the Kali machine spiked the count past the threshold of 10, the engine automatically populated the global triage queue.

*Below is the step-by-step alert configuration rule profile:*
![Alert Setup Process](screenshots/alert_setup.png)

*Below is the resulting high-severity incident generating inside the active SOC queue:*
![Splunk Triggered Alerts Queue](screenshots/splunk_alerts.png)

---

## Key Technical Takeaways & Skills Validated
* **SIEM Engineering:** Deploying log forwarders, configuring data ingestion metrics, and managing local asset index spaces.
* **Data Normalization:** Writing custom SPL to map unstructured authentication payloads into organized data frameworks.
* **Defensive Threshold Automation:** Designing specific mathematical rules to reduce false-positive metrics and limit alert fatigue in live SOC environments.
* **Troubleshooting & System Administration:** Optimizing underlying VM storage performance thresholds (`limits.conf`) to handle intensive query loads under hardware constraints.
