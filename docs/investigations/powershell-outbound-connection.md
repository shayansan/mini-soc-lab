# PowerShell Outbound Network Connection Investigation

## Scenario Overview

This scenario simulates an outbound TCP connection initiated by PowerShell on the Windows victim.

The objective was to verify whether Sysmon and Wazuh could identify PowerShell-generated network activity and provide sufficient context for SOC investigation.

## MITRE ATT&CK Mapping

- Technique: Non-Application Layer Protocol
- Technique ID: T1095
- Tactic: Command and Control

## Test Environment

- Windows victim: `soctest`
- Wazuh agent ID: `001`
- Wazuh manager: `soc-wazuh`
- Telemetry source: Sysmon
- Relevant Sysmon Event ID: `3`

## Test Activity

The following PowerShell command was executed:

```powershell
Test-NetConnection 192.168.70.30 -Port 1514
```

The command generated a harmless TCP connection from the Windows victim to the Wazuh manager.

## Telemetry Observed

Sysmon generated Event ID 3 for the network connection.

Relevant fields included:

- Image: `powershell.exe`
- Process ID: `3612`
- User: `soctest\shaya`
- Protocol: `tcp`
- Initiated: `true`
- Source IP: `192.168.70.20`
- Destination IP: `192.168.70.30`
- Destination Port: `1514`

## Wazuh Detection Chain

The activity first matched the Sysmon Event ID 3 rule chain.

Wazuh built-in rule `92101` identifies PowerShell processes communicating over TCP.

A custom child rule was then used to raise the severity for outbound connections initiated by PowerShell.

Detection chain:

`Sysmon Event ID 3`
→ `61605`
→ `92101`
→ `100110`

## Custom Detection Rule

```xml
<rule id="100110" level="10">
  <if_sid>92101</if_sid>
  <field name="win.eventdata.initiated">^true$</field>
  <description>Outbound TCP network connection initiated by PowerShell</description>
  <mitre>
    <id>T1095</id>
  </mitre>
  <group>network,powershell,windows,outbound_connection,</group>
</rule>
```

## Detection Result

The custom rule successfully generated a Level 10 alert:

- Rule ID: `100110`
- Rule Level: `10`
- Description: `Outbound TCP network connection initiated by PowerShell`
- MITRE ATT&CK: `T1095`
- Technique: `Non-Application Layer Protocol`

## Investigation Assessment

PowerShell-generated network connections are not inherently malicious, but they can be associated with remote command execution, payload retrieval, command-and-control traffic, or attacker tooling.

The connection observed in this scenario was intentionally generated as part of the Mini SOC lab.

Classification:

**True Positive — Authorized Simulation**

No real command-and-control activity occurred.

## Analyst Conclusion

The scenario demonstrated that the Mini SOC pipeline can:

1. Capture outbound network activity using Sysmon.
2. Identify PowerShell as the initiating process.
3. Inspect source and destination network metadata.
4. Use built-in Wazuh detection logic.
5. Escalate PowerShell-generated TCP activity with a custom rule.
6. Map the activity to MITRE ATT&CK T1095.

The outbound PowerShell network connection scenario was therefore considered successful.
