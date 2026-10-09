# Project Checklist

## Phase 1 — Project Design

- [x] Define project goal
- [x] Design architecture
- [x] Define machines
- [x] Define tools
- [x] Select first detection scenario

## Phase 2 — Lab Infrastructure

- [x] Create virtual network
- [x] Create Kali attacker VM
- [x] Create Windows victim VM
- [x] Create Ubuntu/Wazuh SIEM environment
- [x] Configure Host-only addressing
- [x] Verify VM connectivity

## Phase 3 — Telemetry

- [x] Install Wazuh agent
- [x] Verify agent connectivity
- [x] Collect Windows Event Logs
- [x] Install Sysmon
- [x] Collect Sysmon telemetry
- [x] Enable Wazuh raw archives
- [x] Enable Windows Defender Firewall logging
- [x] Collect Firewall logs
- [x] Verify logs reach Wazuh

## Phase 4 — Detection

- [x] Generate controlled reconnaissance
- [x] Perform Nmap scan
- [x] Observe Firewall DROP telemetry
- [x] Validate Wazuh firewall decoding
- [x] Create custom detection rule
- [x] Trigger custom detection
- [x] Map detection to MITRE ATT&CK T1046

## Phase 5 — Investigation

- [x] Identify attacker source IP
- [x] Identify victim destination IP
- [x] Inspect source/destination ports
- [x] Analyze detection rule
- [ ] Build full event timeline
- [ ] Determine tuning requirements
- [ ] Perform additional investigation scenarios

## Phase 6 — Incident Documentation

- [ ] Write incident report
- [ ] Add screenshots and evidence
- [ ] Document findings
- [ ] Document analyst conclusions

## Phase 7 — Project Cleanup

- [x] Improve README
- [x] Update architecture documentation
- [x] Update lab setup documentation
- [x] Document first detection scenario
- [ ] Add architecture diagram
- [ ] Add lessons learned
- [ ] Add final incident report
