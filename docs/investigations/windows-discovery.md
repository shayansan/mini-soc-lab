# Investigation Report — Windows Discovery Sequence

## Summary

A sequence of Windows discovery commands was executed on the monitored Windows 11 endpoint and detected by Wazuh.

The activity was generated intentionally as part of the Mini SOC Lab.

## Endpoint

- Agent name: soctest
- Agent ID: 001
- Victim IP: 192.168.70.20
- User: soctest\shaya

## Detection

- Correlation Rule ID: 100105
- Alert Level: 10
- Description: Possible Windows discovery sequence detected
- Detection type: Behavioral correlation
- Time window: 30 seconds
- Frequency threshold: 3 events

## Timeline

| UTC Time | Command | Process ID | Parent Process | Parent PID |
|---|---|---:|---|---:|
| 09:35:03.304 | whoami.exe | 2432 | powershell.exe | 11112 |
| 09:35:06.718 | net.exe user | 7396 | powershell.exe | 11112 |
| 09:35:12.367 | ipconfig.exe /all | 9504 | powershell.exe | 11112 |
| 09:35:15.422 | tasklist.exe | 2960 | powershell.exe | 11112 |

The complete discovery sequence occurred in approximately 12 seconds.

## Process Relationship

All four commands were executed from the same PowerShell parent process:

```text
powershell.exe
  |
  +-- whoami.exe
  +-- net.exe user
  +-- ipconfig.exe /all
  +-- tasklist.exe
