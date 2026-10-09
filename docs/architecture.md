# Mini SOC Architecture

## Overview

This project contains three main virtual machines:

- Kali Linux attacker
- Windows 11 victim
- Ubuntu Server running Wazuh

All virtual machines run inside VirtualBox.

## Network

The lab uses a Host-only network:

- Host: 192.168.70.1
- Kali attacker: 192.168.70.10
- Windows victim: 192.168.70.20
- Wazuh server: 192.168.70.30

Subnet:

192.168.70.0/24

Each virtual machine also has a NAT adapter for Internet access.

## Kali Linux Attacker

Purpose:

- Generate controlled reconnaissance traffic
- Perform Nmap scans
- Simulate attacker activity against the Windows victim

IP:

192.168.70.10

## Windows 11 Victim

Purpose:

- Act as the monitored endpoint
- Generate security telemetry
- Receive controlled attacker traffic

IP:

192.168.70.20

Monitoring components:

- Wazuh Agent
- Sysmon
- Windows Event Logs
- Windows Defender Firewall logging

Wazuh agent name:

soctest

## Wazuh SIEM Server

Operating System:

Ubuntu Server

IP:

192.168.70.30

Components:

- Wazuh Manager
- Wazuh Indexer
- Wazuh Dashboard
- Filebeat

Purpose:

- Receive telemetry
- Decode events
- Apply detection rules
- Generate alerts
- Store events
- Support investigation

## Sysmon Telemetry

Verified Sysmon events:

- Event ID 1 — Process Create
- Event ID 3 — Network Connection
- Event ID 11 — File Create
- Event ID 12 — Registry Object Create/Delete
- Event ID 13 — Registry Value Set
- Event ID 22 — DNS Query

## Windows Firewall Telemetry

Blocked inbound traffic is logged in:

C:\Windows\System32\LogFiles\Firewall\pfirewall.log

The Wazuh agent collects this log and forwards it to the Wazuh server.

## Detection Scenario

Kali performs an Nmap scan against the Windows victim.

Example:

```bash
nmap -Pn -p 135,139,445,5040,49664-49673 --reason 192.168.70.20
