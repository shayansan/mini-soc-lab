# Suspicious PowerShell Execution Investigation

## Scenario Overview

This scenario simulates suspicious PowerShell execution on the Windows victim.

The objective was to determine whether Sysmon and Wazuh could detect PowerShell execution using potentially suspicious command-line options such as `-ExecutionPolicy Bypass`.

## MITRE ATT&CK Mapping

- Technique: PowerShell
- Technique ID: T1059.001
- Tactic: Execution

## Test Environment

- Windows victim: `soctest`
- Wazuh agent ID: `001`
- Wazuh manager: `soc-wazuh`
- Telemetry source: Sysmon
- Relevant Sysmon Event ID: `1`

## Test Command

The following command was executed on the Windows victim:

```powershell
powershell.exe -NoProfile -ExecutionPolicy Bypass -Command "Write-Output 'MiniSOC-PowerShell-Test2'"
```

The command itself was harmless and only generated test output.

However, `-ExecutionPolicy Bypass` can also appear in malicious PowerShell activity and is therefore useful as a detection indicator when combined with process context.

## Telemetry Observed

Sysmon generated Event ID 1 for the PowerShell process creation.

Relevant fields included:

- Image: `powershell.exe`
- Process ID: `10056`
- User: `soctest\shaya`
- Integrity Level: `High`
- Parent Image: `powershell.exe`
- Command Line containing `-ExecutionPolicy Bypass`

## Wazuh Detection Chain

The activity first matched Wazuh built-in rule `92027`:

`Powershell process spawned powershell instance`

A custom child rule was then used to increase the severity when the PowerShell command line contained `ExecutionPolicy Bypass`.

Detection chain:

`Sysmon Event ID 1`
→ `92027`
→ `100107`

## Custom Detection Rule

```xml
<rule id="100107" level="10">
  <if_sid>92027</if_sid>
  <field name="win.eventdata.commandLine" type="pcre2">(?i)-ExecutionPolicy\s+Bypass</field>
  <description>Suspicious PowerShell execution with ExecutionPolicy Bypass detected</description>
  <mitre>
    <id>T1059.001</id>
  </mitre>
  <group>execution,powershell,windows,suspicious_command,</group>
</rule>
```

## Detection Result

The custom rule successfully generated a Level 10 alert:

- Rule ID: `100107`
- Rule Level: `10`
- Description: `Suspicious PowerShell execution with ExecutionPolicy Bypass detected`
- MITRE ATT&CK: `T1059.001`
- Technique: `PowerShell`

## Investigation Assessment

The activity was intentionally generated as part of the Mini SOC lab.

Although the use of `ExecutionPolicy Bypass` is not inherently malicious, it can be associated with attempts to execute PowerShell content while bypassing local policy restrictions.

The observed process context also showed PowerShell spawning another PowerShell process.

Classification:

**True Positive — Authorized Simulation**

No real compromise occurred.

## Analyst Conclusion

The scenario demonstrated that the Mini SOC pipeline can:

1. Capture PowerShell process creation using Sysmon.
2. Forward process telemetry to Wazuh.
3. Detect PowerShell execution using built-in Wazuh logic.
4. Inspect command-line arguments.
5. Escalate suspicious PowerShell behavior through a custom detection rule.
6. Map the activity to MITRE ATT&CK T1059.001.

The PowerShell execution detection scenario was therefore considered successful.
