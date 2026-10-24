# Final SOC Summary

## Project Overview

The Mini SOC Lab was designed to reproduce a small Security Operations Center workflow using an isolated virtualized environment.

The project focused on generating controlled suspicious activity, collecting Windows endpoint and network telemetry, detecting behaviors with Wazuh, creating custom detection rules, mapping activity to MITRE ATT&CK, and documenting analyst investigations.

## Environment

The lab consisted of three primary systems:

- Kali Linux — attacker and test traffic generator
- Windows 11 — monitored victim endpoint
- Ubuntu Server — Wazuh SIEM infrastructure

Network:

- Kali Linux: `192.168.70.10`
- Windows victim: `192.168.70.20`
- Wazuh server: `192.168.70.30`

The Windows endpoint used Sysmon and the Wazuh Agent to provide telemetry to the SIEM.

## Detection Pipeline

The final monitoring pipeline was:

`Controlled Activity`
→ `Windows / Sysmon Telemetry`
→ `Wazuh Agent`
→ `Wazuh Manager`
→ `Built-in and Custom Rules`
→ `Alert`
→ `SOC Investigation`
→ `MITRE ATT&CK Mapping`
→ `Investigation Report`

## Detection Scenarios

### 1. Network Service Scanning

An Nmap scan from Kali Linux generated multiple blocked connection attempts against the Windows endpoint.

Custom correlation rule:

- Rule ID: `100100`
- MITRE ATT&CK: `T1046`

### 2. Windows Discovery Sequence

Multiple discovery commands were executed within a short period:

- `whoami`
- `net user`
- `ipconfig /all`
- `tasklist`

Custom detection and correlation rules:

- `100101`
- `100102`
- `100103`
- `100104`
- `100105`

MITRE techniques included:

- `T1033`
- `T1087`
- `T1016`
- `T1057`

### 3. Registry Run Key Persistence

A Registry Run Key was created to simulate logon persistence.

Detection chain:

`Sysmon Event ID 13`
→ `61615`
→ `92300`
→ `100106`

MITRE ATT&CK:

- `T1547.001`

### 4. Suspicious PowerShell Execution

PowerShell was executed using `ExecutionPolicy Bypass`.

Detection chain:

`Sysmon Event ID 1`
→ `92027`
→ `100107`

MITRE ATT&CK:

- `T1059.001`

### 5. Suspicious File Creation

PowerShell created an executable-looking file inside the Windows temporary directory.

Detection chain:

`Sysmon Event ID 11`
→ `92213`
→ `100108`

MITRE ATT&CK:

- `T1105`

### 6. Scheduled Task Creation

`schtasks.exe` was used to create a task configured for execution at logon.

Custom rule:

- Rule ID: `100109`

MITRE ATT&CK:

- `T1053.005`

### 7. PowerShell Outbound TCP Connection

PowerShell initiated an outbound TCP connection.

Detection chain:

`Sysmon Event ID 3`
→ `61605`
→ `92101`
→ `100110`

MITRE ATT&CK:

- `T1095`

## Custom Detection Engineering

The project produced eleven custom Wazuh rules.

| Rule | Purpose |
|---|---|
| 100100 | Port scan correlation |
| 100101 | User discovery |
| 100102 | Network configuration discovery |
| 100103 | Process discovery |
| 100104 | Account discovery |
| 100105 | Discovery sequence correlation |
| 100106 | Registry persistence |
| 100107 | PowerShell ExecutionPolicy Bypass |
| 100108 | PowerShell-created executable |
| 100109 | Scheduled Task creation |
| 100110 | PowerShell outbound TCP activity |

## Investigation Methodology

Each scenario was investigated using a consistent workflow:

1. Validate the alert.
2. Identify the endpoint and user.
3. Inspect the originating process.
4. Review command-line arguments.
5. Review related Sysmon telemetry.
6. Inspect registry, file, or network context when relevant.
7. Map behavior to MITRE ATT&CK.
8. Determine whether the event represented malicious or authorized activity.
9. Document the investigation.

All generated lab scenarios were classified as:

**True Positive — Authorized Simulation**

## Key Lessons Learned

The project demonstrated that detection engineering requires more than matching individual fields.

One of the most important lessons was understanding Wazuh rule hierarchy. Custom rules sometimes needed to inherit from existing built-in detection rules rather than directly from the original Sysmon rule.

Examples included:

`92300 → 100106`

and:

`92101 → 100110`

Raw telemetry was also critical during troubleshooting. Comparing `archives.json` and `alerts.json` made it possible to distinguish telemetry collection problems from rule-matching problems.

Sysmon proved especially useful because process, registry, network, and file events could be correlated around the same endpoint and user context.

## Skills Demonstrated

This project demonstrates practical experience with:

- SOC monitoring
- SIEM administration
- Wazuh
- Sysmon
- Windows security telemetry
- Windows Defender Firewall logging
- Detection engineering
- Custom Wazuh rules
- Correlation rules
- MITRE ATT&CK
- Alert triage
- Endpoint investigation
- Process analysis
- Registry analysis
- Network analysis
- Linux administration
- Git
- GitHub documentation

## Limitations

The environment is an educational lab rather than a production SOC.

Limitations include:

- One monitored Windows endpoint
- One attacker VM
- Limited log volume
- No centralized identity infrastructure
- No production threat intelligence feeds
- No automated incident response
- No real malware execution

## Future Improvements

Potential future extensions include:

- PowerShell Script Block Logging
- Windows authentication attack scenarios
- DNS-based detection engineering
- Sigma rules
- Threat intelligence enrichment
- Additional endpoints
- Linux detection scenarios
- Automated response actions
- Detection tuning and false-positive testing
- SOC dashboards and metrics

## Final Assessment

The Mini SOC Lab successfully demonstrated an end-to-end defensive security workflow from telemetry collection through detection, investigation, and documentation.

The project provides a reproducible portfolio example of practical SOC analysis and detection engineering using Windows, Sysmon, Wazuh, Linux, and MITRE ATT&CK.
