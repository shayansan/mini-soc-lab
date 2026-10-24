# Scheduled Task Creation Investigation

## Scenario Overview

This scenario simulates the creation of a Windows Scheduled Task using `schtasks.exe`.

The objective was to determine whether the Mini SOC pipeline could detect task creation activity commonly associated with persistence and execution techniques.

## MITRE ATT&CK Mapping

- Technique: Scheduled Task/Job: Scheduled Task
- Technique ID: T1053.005
- Tactics: Execution, Persistence, Privilege Escalation

## Test Environment

- Windows victim: `soctest`
- Wazuh agent ID: `001`
- Wazuh manager: `soc-wazuh`
- Telemetry source: Sysmon
- Relevant Sysmon Event ID: `1`

## Test Command

The following command was executed on the Windows victim:

```powershell
schtasks /Create /SC ONLOGON /TN "MiniSOC-TestTask2" /TR "cmd.exe /c echo MiniSOC-ScheduledTask2 > %TEMP%\mini-soc-task2.txt" /F
```

The scheduled task was configured to execute a harmless command at user logon.

## Telemetry Observed

Sysmon generated Event ID 1 for the creation of the `schtasks.exe` process.

Relevant fields included:

- Image: `C:\Windows\System32\schtasks.exe`
- Process ID: `4280`
- User: `soctest\shaya`
- Integrity Level: `High`
- Parent Image: `powershell.exe`
- Command Line containing `/Create`

## Custom Detection Rule

```xml
<rule id="100109" level="10">
  <if_group>sysmon_event1</if_group>
  <field name="win.eventdata.image" type="pcre2">(?i)\\schtasks\.exe$</field>
  <field name="win.eventdata.commandLine" type="pcre2">(?i)\s/Create(\s|$)</field>
  <description>Scheduled task creation detected via schtasks.exe</description>
  <mitre>
    <id>T1053.005</id>
  </mitre>
  <group>persistence,execution,windows,scheduled_task,</group>
</rule>
```

## Detection Result

The custom rule successfully generated a Level 10 alert:

- Rule ID: `100109`
- Rule Level: `10`
- Description: `Scheduled task creation detected via schtasks.exe`
- MITRE ATT&CK: `T1053.005`
- Technique: `Scheduled Task`

## Investigation Assessment

Scheduled tasks can be used legitimately for system administration and automation, but they are also commonly abused by attackers to establish persistence or execute commands at predefined times or user logon.

In this lab, the activity was deliberately generated as an authorized simulation.

Classification:

**True Positive — Authorized Simulation**

No real malicious payload was used.

## Analyst Conclusion

The scenario demonstrated that the Mini SOC pipeline can:

1. Capture process creation telemetry using Sysmon.
2. Detect execution of `schtasks.exe`.
3. Inspect command-line arguments for task creation behavior.
4. Identify Scheduled Task creation activity.
5. Generate a custom high-severity Wazuh alert.
6. Map the activity to MITRE ATT&CK T1053.005.

The Scheduled Task creation detection scenario was therefore considered successful.
