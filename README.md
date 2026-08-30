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
<img width="732" height="511" alt="Image" src="https://github.com/user-attachments/assets/706d68cb-de2b-46fa-af22-0f3d8d8455ba" />

---

### Phase 2: Ingestion & Telemetry Engineering (Blue Team Action)
To ensure the Windows event logs were readable by the Splunk core engine, I customized the endpoint's ingestion configurations. By setting `renderXml = false` in the forwarder's properties, Windows raw data was transmitted as structured key-value text pairs, allowing Splunk to natively extract authentication headers.

*Below is the verification of the raw Event Code 4625 logs landing securely in the SIEM index:*
<img width="1137" height="570" alt="Screenshot 2026-08-28 220833" src="https://github.com/user-attachments/assets/31dd97d5-da09-41cb-86bb-1e8ef4a784fa" />


---

### Phase 3: Analytical Development (Detection Engineering)
I authored an optimized, interview-proof SPL query designed to aggregate raw authentication telemetry while actively suppressing alert noise. 

<img width="610" height="162" alt="Screenshot 2026-08-28 212648" src="https://github.com/user-attachments/assets/58591228-786f-4f1f-a3fa-4b3599804d19" />


#### **Query Breakdown:**
1. **`index="main" EventCode=4625`**: Pulls Windows Security events tracking explicit logon failures.
2. **`| rename...`**: Standardizes unindexed system tags into clean, human-readable data properties (`sc_ip` and `Account_Name`).
3. **`| stats count by...`**: Groups multi-dimensional fields together to immediately correlate exactly *who* was targeted right next to *where* the attack came from.
4. **`| where count > 10`**: Establishes a strict behavioral threshold. It ignores standard employee typos but flags automated automation tools (like Hydra) the moment they exceed 10 rapid attempts.

*Below is the visual confirmation of the optimized SPL search yielding clear metadata fields:*
<img width="1058" height="513" alt="Screenshot 2026-08-28 220909" src="https://github.com/user-attachments/assets/fe6e7810-d33f-49f9-b214-726c81466a87" />


---

## Operational Deliverables (SOC Operations)

### 1. Custom Security Operations Center Dashboard
I saved the optimized SPL query as a persistent visual tracking panel using Splunk's Classic Dashboard studio. This workspace gives internal tier-1 analysts immediate situational visibility over active authentication threats across the domain infrastructure.

*Below is the live Enterprise SOC Analyst Workspace displaying the attack metrics:*
<img width="1363" height="453" alt="Screenshot 2026-08-28 215941" src="https://github.com/user-attachments/assets/668abd75-994c-4bb9-b59e-da55e0a17b7e" />


### 2. Automated Trigger Alert Queue
To ensure proactive network defense when analysts are away from the dashboard console, I configured the threshold code to run as a **Scheduled Alert Rule** utilizing background cron intervals (`*/5 * * * *`). The moment the Kali machine spiked the count past the threshold of 10, the engine automatically populated the global triage queue.

*Below is the step-by-step alert configuration rule profile:*
<img width="823" height="541" alt="Screenshot 2026-08-28 211923" src="https://github.com/user-attachments/assets/b826d89b-2a92-4711-a8fd-a30c22baf76c" />


*Below is the resulting high-severity incident generating inside the active SOC queue:*
<img width="1350" height="176" alt="Screenshot 2026-08-28 215752" src="https://github.com/user-attachments/assets/29e20b11-1c87-4fc9-8a5c-c23e9fc2fd08" />


---

## Key Technical Takeaways & Skills Validated
* **SIEM Engineering:** Deploying log forwarders, configuring data ingestion metrics, and managing local asset index spaces.
* **Data Normalization:** Writing custom SPL to map unstructured authentication payloads into organized data frameworks.
* **Defensive Threshold Automation:** Designing specific mathematical rules to reduce false-positive metrics and limit alert fatigue in live SOC environments.
* **Troubleshooting & System Administration:** Optimizing underlying VM storage performance thresholds (`limits.conf`) to handle intensive query loads under hardware constraints.
