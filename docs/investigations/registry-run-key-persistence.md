# Registry Run Key Persistence Investigation

## Scenario Overview

This scenario simulates a persistence technique on the Windows victim by creating a Registry Run Key under:

`HKCU\Software\Microsoft\Windows\CurrentVersion\Run`

The purpose of the test was to verify whether Sysmon and Wazuh could detect a registry value modification associated with Windows logon persistence.

## MITRE ATT&CK Mapping

- Technique: Registry Run Keys / Startup Folder
- Technique ID: T1547.001
- Tactic: Persistence / Privilege Escalation

## Test Environment

- Windows victim: `soctest`
- Wazuh agent ID: `001`
- Wazuh manager: `soc-wazuh`
- Telemetry source: Sysmon
- Relevant Sysmon Event ID: `13`

## Persistence Simulation

The following PowerShell command was executed on the Windows victim:

```powershell
New-ItemProperty -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\Run" -Name "MiniSOCTest11" -PropertyType String -Value 'cmd.exe /c echo MiniSOCTest11 > "%TEMP%\mini-soc-persistence11.txt"' -Force
```

This created the following registry value:

`HKCU\Software\Microsoft\Windows\CurrentVersion\Run\MiniSOCTest11`

The configured command would execute during user logon.

## Telemetry Observed

Sysmon generated Event ID 13 for the registry value modification.

Relevant event fields included:

- Event type: `SetValue`
- Image: `powershell.exe`
- Target object: `...\Software\Microsoft\Windows\CurrentVersion\Run\MiniSOCTest11`
- User: `soctest\shaya`
- Process ID: `3612`
- Command value: `cmd.exe /c echo MiniSOCTest11 > "%TEMP%\mini-soc-persistence11.txt"`

The event was successfully received by the Wazuh manager through the `windows_eventchannel` decoder.

## Wazuh Detection Chain

The event passed through the following Wazuh rule hierarchy:

`Sysmon Event ID 13`
→ `61615`
→ `92300`
→ `100106`

Wazuh built-in rule `92300` identifies registry entries associated with execution during the next user logon.

A custom child rule was then used to raise the severity of the detection.

## Custom Detection Rule

```xml
<rule id="100106" level="10">
  <if_sid>92300</if_sid>
  <description>Possible registry Run key persistence detected</description>
  <mitre>
    <id>T1547.001</id>
  </mitre>
  <group>persistence,windows,registry,run_key,</group>
</rule>
```

## Detection Result

The custom rule successfully generated a Level 10 alert:

- Rule ID: `100106`
- Level: `10`
- Description: `Possible registry Run key persistence detected`
- MITRE ATT&CK: `T1547.001`
- Technique: `Registry Run Keys / Startup Folder`

The alert was associated with the Windows victim `soctest` and the PowerShell registry modification.

## Investigation Assessment

The activity represents a known Windows persistence mechanism because values stored under the Registry Run Key can execute automatically when the user logs on.

In this lab, the activity was intentionally generated as an authorized security simulation.

Classification:

**True Positive — Authorized Simulation**

No real compromise occurred.

## Analyst Conclusion

The scenario confirmed that the Mini SOC pipeline can:

1. Collect registry modification telemetry using Sysmon.
2. Forward Windows Event Channel data through the Wazuh agent.
3. Identify Sysmon Event ID 13 registry activity.
4. Map the behavior to MITRE ATT&CK T1547.001.
5. Escalate the built-in Wazuh detection through a custom Level 10 rule.
6. Provide sufficient telemetry for analyst investigation.

The persistence detection scenario was therefore considered successful.
