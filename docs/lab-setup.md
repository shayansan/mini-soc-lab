# Mini SOC Lab Setup

## Host Machine

- OS: Windows
- CPU: AMD Ryzen 9 8940HX
- Physical cores: 16
- Logical processors: 32
- RAM: 32 GB
- Storage: 1 TB SSD
- Purpose: Physical host running the complete Mini SOC virtual lab

## Virtualization Platform

- Platform: Oracle VirtualBox
- Purpose: Run the attacker, victim, and SIEM virtual machines
- Snapshots: Recommended before major configuration or attack-simulation changes

## Virtual Machines

### Kali Linux Attacker

- Role: Generate controlled reconnaissance and attack-simulation traffic
- RAM: 4 GB
- vCPU: 4
- Disk: 40 GB
- Host-only IP: `192.168.70.10`
- Internet access: VirtualBox NAT

### Windows 11 Victim

- Role: Monitored Windows endpoint
- RAM: 8 GB
- vCPU: 4
- Disk: 80 GB
- Host-only IP: `192.168.70.20`
- Wazuh agent name: `soctest`
- Internet access: VirtualBox NAT

Telemetry sources currently enabled:

- Windows Application Event Log
- Windows Security Event Log
- Windows System Event Log
- Sysmon
- Windows Defender Firewall log
- Wazuh File Integrity Monitoring
- Wazuh Security Configuration Assessment
- Wazuh system inventory

### Ubuntu Server / Wazuh

- OS: Ubuntu Server 24.04 LTS
- Role: Central SIEM and security monitoring server
- RAM: 8 GB
- vCPU: 4
- Disk: 80 GB
- Host-only IP: `192.168.70.30`
- Internet access: VirtualBox NAT

Wazuh components:

- Wazuh Manager
- Wazuh Indexer
- Wazuh Dashboard
- Filebeat

## Network Design

### Host-only SOC Network

The original design used a VirtualBox Internal Network. During implementation, Layer 2 connectivity problems occurred between the virtual machines, so the final lab was moved to a VirtualBox Host-only network.

- Network: `192.168.70.0/24`
- Host adapter: `192.168.70.1`
- Kali attacker: `192.168.70.10`
- Windows victim: `192.168.70.20`
- Wazuh server: `192.168.70.30`

The Host-only network is used for:

- Attacker-to-victim traffic
- Wazuh agent-to-manager communication
- Administration of the Wazuh server from the physical host
- Access to the Wazuh Dashboard

No default gateway or DNS server is configured on the Host-only interfaces.

### Internet Access

Each VM also has a separate NAT adapter.

The NAT adapter is used only for:

- Package installation
- Operating system updates
- Tool downloads
- General Internet access

Simulated attack traffic uses the Host-only network.

## Routing Design

### Kali Linux

- NAT interface: Internet access
- Host-only interface: `192.168.70.10/24`
- Default route: NAT interface
- SOC traffic: Host-only interface

Expected route to the Windows victim:

```text
192.168.70.20 dev eth1 src 192.168.70.10
```

### Windows Victim

- NAT interface: Internet access
- Host-only interface: `192.168.70.20/24`
- Wazuh manager: `192.168.70.30`

### Wazuh Server

- NAT interface: Internet access
- Host-only interface: `192.168.70.30/24`

## Wazuh Agent

The Windows victim runs the Wazuh agent and sends telemetry to the manager at:

```text
192.168.70.30:1514/TCP
```

The endpoint is registered in Wazuh as:

```text
soctest
```

## Sysmon

Sysmon is installed on the Windows victim and the following event channel is collected by the Wazuh agent:

```text
Microsoft-Windows-Sysmon/Operational
```

Verified Sysmon telemetry:

- Event ID 1 - Process Create
- Event ID 3 - Network Connection
- Event ID 11 - File Create
- Event ID 12 - Registry Object Create/Delete
- Event ID 13 - Registry Value Set
- Event ID 22 - DNS Query

## Wazuh Raw Event Archives

Raw event archives are enabled for lab investigation.

Filebeat indexes both alerts and archives. The archive index pattern is:

```text
wazuh-archives-*
```

This allows investigation of telemetry that does not necessarily trigger a Wazuh alert.

## Windows Defender Firewall Logging

Blocked inbound traffic is logged on the Windows victim at:

```text
C:\Windows\System32\LogFiles\Firewall\pfirewall.log
```

The Wazuh agent collects this file and forwards the events to the manager.

## Reconnaissance Test

The Kali VM was used to perform a controlled Nmap scan against the Windows victim.

Example:

```bash
nmap -Pn -p 135,139,445,5040,49664-49673 --reason 192.168.70.20
```

Windows Defender Firewall blocked the probes and logged the Kali source address `192.168.70.10`.

## Custom Wazuh Detection

A custom correlation rule was created to identify repeated Windows Firewall DROP events from the same source IP.

- Custom rule ID: `100100`
- Base rule: `4101` - Firewall drop event
- Detection type: Repeated DROP events from the same source IP
- Alert level: `10`
- MITRE ATT&CK: `T1046 - Network Service Scanning`

The rule successfully detected Nmap reconnaissance from Kali.

## Verified Detection Pipeline

```text
Kali
  ↓
Nmap reconnaissance
  ↓
Windows Defender Firewall
  ↓
pfirewall.log
  ↓
Wazuh Agent
  ↓
Wazuh Manager
  ↓
Built-in firewall rule 4101
  ↓
Custom correlation rule 100100
  ↓
Level 10 alert
  ↓
MITRE ATT&CK T1046
  ↓
Wazuh Dashboard
```

## Current Lab Status

Completed:

- VirtualBox lab environment
- Ubuntu/Wazuh SIEM server
- Windows 11 victim
- Kali attacker
- Host-only SOC network
- Wazuh agent enrollment
- Wazuh Dashboard access
- Wazuh API connectivity
- Sysmon installation
- Sysmon telemetry ingestion
- Raw Wazuh archives
- Windows Defender Firewall logging
- Firewall log ingestion
- Nmap reconnaissance
- Custom port-scan detection
- MITRE ATT&CK mapping

Next planned work:

- Additional Windows discovery simulations
- More custom Wazuh detection rules
- Detection tuning
- Investigation timelines
- Incident documentation
- Final project report
