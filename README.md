# Mini SOC Lab

A hands-on Security Operations Center lab focused on Windows endpoint telemetry, SIEM detection engineering, alert correlation, and incident investigation.

The lab uses Kali Linux to generate controlled attack activity against a Windows 11 endpoint. Sysmon and Windows security telemetry are collected by a Wazuh agent and analyzed on a dedicated Wazuh SIEM server.

The project demonstrates the complete SOC workflow:

**Generate Activity → Collect Telemetry → Detect → Correlate → Investigate → Document**

---

## Project Objectives

The primary objectives of this project are to:

- Build a small but realistic SOC monitoring environment.
- Collect endpoint and network telemetry from a Windows system.
- Analyze Sysmon and Windows Firewall events.
- Create custom Wazuh detection and correlation rules.
- Map detections to the MITRE ATT&CK framework.
- Investigate alerts using process, network, registry, and file telemetry.
- Document findings using structured investigation reports.
- Maintain the project as a reproducible cybersecurity portfolio project.

---

## Lab Architecture

```text
                   Mini SOC Lab

        ┌──────────────────────────┐
        │       Kali Linux         │
        │       Attacker VM        │
        │     192.168.70.10        │
        └────────────┬─────────────┘
                     │
                     │ Controlled attack traffic
                     ▼
        ┌──────────────────────────┐
        │       Windows 11         │
        │       Victim VM          │
        │     192.168.70.20        │
        │                          │
        │ Sysmon                   │
        │ Windows Firewall         │
        │ Wazuh Agent              │
        └────────────┬─────────────┘
                     │
                     │ Security telemetry
                     ▼
        ┌──────────────────────────┐
        │      Ubuntu Server       │
        │       Wazuh SIEM         │
        │     192.168.70.30        │
        │                          │
        │ Wazuh Manager            │
        │ Wazuh Indexer            │
        │ Wazuh Dashboard          │
        └──────────────────────────┘
```

The lab uses a VirtualBox Host-only network:

```text
192.168.70.0/24
```

The virtual machines also use NAT interfaces when Internet access is required.

---

## Lab Components

| Component | Role | Address |
|---|---|---|
| Kali Linux | Attacker / test traffic generator | `192.168.70.10` |
| Windows 11 | Monitored victim endpoint | `192.168.70.20` |
| Ubuntu Server | Wazuh SIEM | `192.168.70.30` |
| Sysmon | Endpoint telemetry | Windows |
| Wazuh Agent | Telemetry forwarding | Windows |
| Wazuh Manager | Detection and correlation | Ubuntu |

---

## Telemetry Sources

The Windows endpoint provides multiple telemetry sources:

- Sysmon
- Windows Event Logs
- Windows Defender Firewall logs
- Wazuh File Integrity Monitoring
- Wazuh Security Configuration Assessment

### Verified Sysmon Events

The following Sysmon events were validated during the lab:

| Event ID | Description |
|---:|---|
| 1 | Process Create |
| 3 | Network Connection |
| 11 | File Create |
| 12 | Registry Object Create/Delete |
| 13 | Registry Value Set |
| 22 | DNS Query |

---

# Detection Scenarios

Seven detection and investigation scenarios were implemented.

## 1. TCP Port Scan Detection

Kali Linux performs an Nmap scan against the Windows victim.

Example:

```bash
nmap -Pn -p 135,139,445,5040,49664-49673 --reason 192.168.70.20
```

Windows Firewall records the connection attempts and Wazuh correlates repeated firewall drops from the same source.

**Custom Rule**

- Rule ID: `100100`
- Severity: `10`
- MITRE ATT&CK: `T1046`
- Technique: Network Service Scanning

Investigation:

`docs/investigations/port-scan.md`

Detection:

`detections/wazuh/port_scan_rule.xml`

---

## 2. Windows Discovery Sequence

A sequence of Windows discovery commands is executed:

```text
whoami
net user
ipconfig /all
tasklist
```

Sysmon Event ID 1 records the process creation events.

Individual custom rules detect each discovery command and a correlation rule identifies multiple discovery actions occurring within a short time window.

**Custom Rules**

- `100101` — User discovery
- `100102` — Network configuration discovery
- `100103` — Process discovery
- `100104` — Account discovery
- `100105` — Discovery sequence correlation

**MITRE ATT&CK**

- `T1033` — System Owner/User Discovery
- `T1087` — Account Discovery
- `T1016` — System Network Configuration Discovery
- `T1057` — Process Discovery

Investigation:

`docs/investigations/windows-discovery.md`

Detection:

`detections/wazuh/windows_discovery_rules.xml`

---

## 3. Registry Run Key Persistence

A Windows Registry Run Key is created to simulate persistence:

```text
HKCU\Software\Microsoft\Windows\CurrentVersion\Run
```

Sysmon Event ID 13 records the registry value modification.

The activity passes through the Wazuh Sysmon detection chain before being escalated by a custom rule.

```text
Sysmon Event ID 13
        ↓
Wazuh Rule 61615
        ↓
Wazuh Rule 92300
        ↓
Custom Rule 100106
```

**Custom Rule**

- Rule ID: `100106`
- Severity: `10`
- MITRE ATT&CK: `T1547.001`
- Technique: Registry Run Keys / Startup Folder

Investigation:

`docs/investigations/registry-run-key-persistence.md`

Detection:

`detections/wazuh/registry_run_key_persistence.xml`

---

## 4. Suspicious PowerShell Execution

PowerShell is executed using:

```powershell
powershell.exe -NoProfile -ExecutionPolicy Bypass -Command "Write-Output 'MiniSOC-PowerShell-Test'"
```

Sysmon Event ID 1 records the process creation.

Wazuh built-in rule `92027` identifies PowerShell spawning another PowerShell process. A custom rule increases severity when `ExecutionPolicy Bypass` is present.

```text
Sysmon Event ID 1
        ↓
Wazuh Rule 92027
        ↓
Custom Rule 100107
```

**Custom Rule**

- Rule ID: `100107`
- Severity: `10`
- MITRE ATT&CK: `T1059.001`
- Technique: PowerShell

Investigation:

`docs/investigations/suspicious-powershell-execution.md`

Detection:

`detections/wazuh/suspicious_powershell_execution.xml`

---

## 5. Suspicious File Creation

PowerShell creates an executable-looking file inside the Windows temporary directory:

```powershell
Set-Content -Path "$env:TEMP\MiniSOC-dropper-test.exe" -Value "MiniSOC test file"
```

Sysmon Event ID 11 records the file creation.

Wazuh built-in rule `92213` identifies an executable dropped into a directory commonly used by malware. A custom child rule adds PowerShell execution context.

```text
Sysmon Event ID 11
        ↓
Wazuh Rule 92213
        ↓
Custom Rule 100108
```

**Custom Rule**

- Rule ID: `100108`
- Severity: `12`
- MITRE ATT&CK: `T1105`
- Technique: Ingress Tool Transfer

Investigation:

`docs/investigations/suspicious-file-creation.md`

Detection:

`detections/wazuh/suspicious_file_creation.xml`

---

## 6. Scheduled Task Creation

A scheduled task is created using `schtasks.exe`:

```powershell
schtasks /Create /SC ONLOGON /TN "MiniSOC-TestTask" /TR "cmd.exe /c echo MiniSOC-ScheduledTask > %TEMP%\mini-soc-task.txt" /F
```

Sysmon Event ID 1 captures execution of `schtasks.exe`.

The custom detection rule checks for `schtasks.exe` combined with the `/Create` argument.

**Custom Rule**

- Rule ID: `100109`
- Severity: `10`
- MITRE ATT&CK: `T1053.005`
- Technique: Scheduled Task

Investigation:

`docs/investigations/scheduled-task-creation.md`

Detection:

`detections/wazuh/scheduled_task_creation.xml`

---

## 7. PowerShell Outbound Network Connection

PowerShell initiates an outbound TCP connection:

```powershell
Test-NetConnection 192.168.70.30 -Port 1514
```

Sysmon Event ID 3 records the connection.

Wazuh built-in rule `92101` identifies PowerShell communicating over TCP, while the custom child rule identifies an outbound connection initiated by PowerShell.

```text
Sysmon Event ID 3
        ↓
Wazuh Rule 61605
        ↓
Wazuh Rule 92101
        ↓
Custom Rule 100110
```

**Custom Rule**

- Rule ID: `100110`
- Severity: `10`
- MITRE ATT&CK: `T1095`
- Technique: Non-Application Layer Protocol

Investigation:

`docs/investigations/powershell-outbound-connection.md`

Detection:

`detections/wazuh/powershell_outbound_connection.xml`

---

# MITRE ATT&CK Coverage

| Scenario | Technique | MITRE ID | Tactic |
|---|---|---|---|
| Port Scan | Network Service Scanning | `T1046` | Discovery |
| User Discovery | System Owner/User Discovery | `T1033` | Discovery |
| Account Discovery | Account Discovery | `T1087` | Discovery |
| Network Discovery | System Network Configuration Discovery | `T1016` | Discovery |
| Process Discovery | Process Discovery | `T1057` | Discovery |
| Registry Persistence | Registry Run Keys / Startup Folder | `T1547.001` | Persistence |
| Suspicious PowerShell | PowerShell | `T1059.001` | Execution |
| Suspicious File Creation | Ingress Tool Transfer | `T1105` | Command and Control |
| Scheduled Task | Scheduled Task | `T1053.005` | Execution / Persistence / Privilege Escalation |
| PowerShell Network Activity | Non-Application Layer Protocol | `T1095` | Command and Control |

---

# Detection Engineering

Custom Wazuh rules created during the project:

| Rule ID | Detection | Level |
|---:|---|---:|
| `100100` | TCP port scan correlation | 10 |
| `100101` | `whoami` execution | 5 |
| `100102` | `ipconfig` execution | 5 |
| `100103` | `tasklist` execution | 5 |
| `100104` | Windows account discovery | 5 |
| `100105` | Windows discovery sequence correlation | 10 |
| `100106` | Registry Run Key persistence | 10 |
| `100107` | PowerShell ExecutionPolicy Bypass | 10 |
| `100108` | PowerShell-created executable in Temp | 12 |
| `100109` | Scheduled Task creation | 10 |
| `100110` | PowerShell outbound TCP connection | 10 |

---

# Repository Structure

```text
mini-soc-lab/
│
├── README.md
│
├── detections/
│   └── wazuh/
│       ├── port_scan_rule.xml
│       ├── powershell_outbound_connection.xml
│       ├── registry_run_key_persistence.xml
│       ├── scheduled_task_creation.xml
│       ├── suspicious_file_creation.xml
│       ├── suspicious_powershell_execution.xml
│       └── windows_discovery_rules.xml
│
├── docs/
│   ├── architecture.md
│   ├── lab-setup.md
│   ├── project-checklist.md
│   │
│   └── investigations/
│       ├── port-scan.md
│       ├── powershell-outbound-connection.md
│       ├── registry-run-key-persistence.md
│       ├── scheduled-task-creation.md
│       ├── suspicious-file-creation.md
│       ├── suspicious-powershell-execution.md
│       └── windows-discovery.md
│
└── .gitignore
```

---

# Investigation Workflow

The investigation methodology used throughout the project follows a simplified SOC workflow:

```text
Alert
  ↓
Validate telemetry
  ↓
Identify process / user / host
  ↓
Inspect command line or network activity
  ↓
Review related Sysmon events
  ↓
Map behavior to MITRE ATT&CK
  ↓
Classify event
  ↓
Document findings
```

Lab-generated activity is classified as:

**True Positive — Authorized Simulation**

The behavior is intentionally generated but correctly detected by the monitoring stack.

---

# Skills Demonstrated

This project demonstrates practical experience with:

- Security monitoring
- SIEM administration
- Wazuh
- Sysmon
- Windows Event Logs
- Windows Firewall telemetry
- Detection engineering
- Custom Wazuh rules
- Event correlation
- Process analysis
- Registry analysis
- Network telemetry analysis
- MITRE ATT&CK mapping
- SOC alert triage
- Incident investigation
- Linux administration
- Windows security monitoring
- Git and GitHub documentation

---

# Key Findings

Several important observations were made during the project:

- Raw telemetry and generated alerts should be treated as separate layers during investigation.
- Sysmon Event IDs provide high-value endpoint visibility when combined with process context.
- Built-in Wazuh rules can be extended effectively using custom child rules.
- Detection rule hierarchy is critical when designing Wazuh rules.
- Parent/child relationships such as `92300 → 100106` and `92101 → 100110` can materially affect whether a custom rule fires.
- Command-line context significantly improves process-based detections.
- Correlation rules can detect behavior that individual low-severity events cannot identify alone.
- Network and endpoint telemetry together provide much stronger investigative context.

---

# Limitations

This is a controlled educational lab rather than a production SOC.

Current limitations include:

- Single Windows endpoint.
- Single attacker system.
- No enterprise-scale log volume.
- No threat-intelligence integration.
- No automated incident-response workflow.
- Limited identity and authentication telemetry.
- Controlled attack simulations rather than real malware.

---

# Future Improvements

Possible future extensions include:

- Additional Windows endpoints.
- Linux endpoint monitoring.
- Authentication attack detection.
- PowerShell Script Block Logging.
- DNS-based detections.
- Threat-intelligence enrichment.
- Sigma rule integration.
- Automated response workflows.
- Alert dashboards and metrics.
- Additional MITRE ATT&CK coverage.
- Detection tuning and false-positive analysis.

---

# Disclaimer

All activity in this repository was performed in an isolated lab environment for educational and defensive security purposes.

No unauthorized systems were targeted.
