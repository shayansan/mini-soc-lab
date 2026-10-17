# Mini SOC Lab

A hands-on SOC lab built with Kali Linux, Windows 11, Sysmon, Windows Defender Firewall, and Wazuh.

## Goal

The goal of this project is to simulate suspicious activity, collect telemetry, detect malicious behavior, investigate alerts, and document incidents.

## Lab Architecture

- Kali Linux attacker: 192.168.70.10
- Windows 11 victim: 192.168.70.20
- Wazuh SIEM server: 192.168.70.30

The lab uses a VirtualBox Host-only network:

192.168.70.0/24

Each virtual machine also has a NAT adapter for Internet access.

## Telemetry Sources

- Windows Event Logs
- Sysmon
- Windows Defender Firewall logs
- Wazuh File Integrity Monitoring
- Wazuh Security Configuration Assessment

## Verified Sysmon Events

- Event ID 1 — Process Create
- Event ID 3 — Network Connection
- Event ID 11 — File Create
- Event ID 12 — Registry Object Create/Delete
- Event ID 13 — Registry Value Set
- Event ID 22 — DNS Query

## First Detection Scenario

Kali performs an Nmap scan against the Windows victim.

Example:

```bash
nmap -Pn -p 135,139,445,5040,49664-49673 --reason 192.168.70.20

## Second Detection Scenario — Windows Discovery Sequence

A second detection scenario was created to identify rapid Windows discovery activity.

The following commands were executed on the Windows victim:

```text
whoami
net user
ipconfig /all
tasklist
