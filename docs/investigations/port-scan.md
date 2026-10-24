# Port Scan Investigation

## Scenario Overview

This scenario simulates TCP port scanning activity against the Windows victim from the Kali Linux attacker machine.

The objective was to determine whether Windows Firewall telemetry and Wazuh correlation rules could identify repeated connection attempts associated with network reconnaissance.

## MITRE ATT&CK Mapping

- Technique: Network Service Scanning
- Technique ID: T1046
- Tactic: Discovery

## Test Environment

- Attacker: Kali Linux
- Attacker IP: `192.168.70.10`
- Windows victim: `soctest`
- Victim IP: `192.168.70.20`
- Wazuh manager: `soc-wazuh`

## Test Activity

The following Nmap scan was executed from Kali Linux:

```bash
nmap -Pn -p 135,139,445,5040,49664-49673 --reason 192.168.70.20
```

The scan generated multiple TCP connection attempts against the Windows victim.

## Telemetry Observed

Windows Firewall logged blocked inbound connection attempts from the Kali attacker.

Wazuh decoded the firewall events using the Windows Firewall decoder and matched the built-in firewall drop rule.

Relevant characteristics included:

- Source IP: `192.168.70.10`
- Destination: Windows victim
- Multiple destination ports
- Repeated connection attempts within a short time window

## Custom Detection Rule

A correlation rule was created to identify multiple firewall drops originating from the same source IP.

```xml
<rule id="100100" level="10" frequency="8" timeframe="10">
  <if_matched_sid>4101</if_matched_sid>
  <same_srcip />
  <description>Possible TCP port scan detected from source IP $(srcip)</description>
  <mitre>
    <id>T1046</id>
  </mitre>
  <group>network_scan,reconnaissance,windows_firewall,</group>
</rule>
```

The rule requires multiple matching firewall-drop events from the same source within a short time period.

## Detection Result

The custom correlation rule successfully generated a Level 10 alert:

- Rule ID: `100100`
- Rule Level: `10`
- Description: `Possible TCP port scan detected from source IP`
- MITRE ATT&CK: `T1046`
- Technique: `Network Service Scanning`

## Investigation Assessment

The repeated connection attempts across multiple ports were consistent with network reconnaissance behavior.

The source address was identified as the Kali Linux attacker machine used in the Mini SOC lab.

Classification:

**True Positive — Authorized Simulation**

The activity was intentionally generated for detection testing and no unauthorized compromise occurred.

## Analyst Conclusion

The scenario demonstrated that the Mini SOC pipeline can:

1. Capture Windows Firewall network telemetry.
2. Identify blocked connection attempts.
3. Correlate repeated events from the same source.
4. Detect probable port-scanning behavior.
5. Generate a high-severity Wazuh alert.
6. Map the activity to MITRE ATT&CK T1046.

The port scan detection scenario was therefore considered successful.
