# Suspicious File Creation Investigation

## Scenario Overview

This scenario simulates suspicious file creation behavior on the Windows victim by creating an executable-looking file inside the user's temporary directory.

The objective was to verify whether Sysmon and Wazuh could identify file creation activity commonly associated with dropper-like or tool-transfer behavior.

## MITRE ATT&CK Mapping

- Technique: Ingress Tool Transfer
- Technique ID: T1105
- Tactic: Command and Control

## Test Environment

- Windows victim: `soctest`
- Wazuh agent ID: `001`
- Wazuh manager: `soc-wazuh`
- Telemetry source: Sysmon
- Relevant Sysmon Event ID: `11`

## Test Command

The following PowerShell command was executed:

```powershell
Set-Content -Path "$env:TEMP\MiniSOC-dropper-test2.exe" -Value "MiniSOC test file 2"
```

The file was not a functional executable. It was a harmless text file using an `.exe` extension to generate file-creation telemetry for the lab.

## Telemetry Observed

Sysmon generated Event ID 11 for the file creation.

Relevant event fields included:

- Image: `powershell.exe`
- Process ID: `3612`
- User: `soctest\shaya`
- Target Filename: `C:\Users\shaya\AppData\Local\Temp\MiniSOC-dropper-test2.exe`

## Wazuh Detection Chain

The activity first matched Wazuh built-in rule `92213`:

`Executable file dropped in folder commonly used by malware`

A custom child rule was then used to identify the additional context that PowerShell was the process responsible for creating the file.

Detection chain:

`Sysmon Event ID 11`
→ `92213`
→ `100108`

## Custom Detection Rule

```xml
<rule id="100108" level="12">
  <if_sid>92213</if_sid>
  <field name="win.eventdata.image" type="pcre2">(?i)\\powershell\.exe$</field>
  <description>PowerShell created an executable file in a temporary directory</description>
  <mitre>
    <id>T1105</id>
  </mitre>
  <group>file_creation,powershell,windows,temp,dropper_like,</group>
</rule>
```

## Detection Result

The custom rule successfully generated a Level 12 alert:

- Rule ID: `100108`
- Rule Level: `12`
- Description: `PowerShell created an executable file in a temporary directory`
- MITRE ATT&CK: `T1105`
- Technique: `Ingress Tool Transfer`

## Investigation Assessment

The observed activity is suspicious because executable files written to temporary directories are commonly associated with staging, payload delivery, droppers, and downloaded tools.

The use of PowerShell as the creating process adds additional context that can increase the relevance of the alert.

In this lab, the activity was intentionally generated as an authorized security simulation.

Classification:

**True Positive — Authorized Simulation**

No real malware or executable payload was used.

## Analyst Conclusion

The scenario demonstrated that the Mini SOC pipeline can:

1. Capture file creation telemetry using Sysmon.
2. Detect executable-looking files written into temporary directories.
3. Identify the process responsible for the file creation.
4. Use built-in Wazuh detection logic.
5. Add contextual detection through a custom child rule.
6. Map the activity to MITRE ATT&CK T1105.

The suspicious file creation scenario was therefore considered successful.
