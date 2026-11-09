# SSH Brute-Force Simulation and Investigation

## 1. Objective

The purpose of this lab is to simulate controlled SSH authentication failures and investigate the recorded events using Wazuh. The objective is to learn how to collect and identify important event details related to repeated authentication failures, such as the source IP address, targeted username, number of failed attempts, and severity level.

## 2. Lab Environment
Kali Linux IP address: 10.0.2.15 (initially displayed) Purpose: Simulation machine

Wazuh Server and Dashboard Ip address: 192.168.1.14 Purpose: SIEM and security event monitoring

Linux OS (Linux-PC) IP address: 192.168.1.13 Purpose: Target system

**Note:** The Linux SSH logs recorded `192.168.1.9` as the source IP address. This differs from the `10.0.2.15` address initially displayed by Kali. The exact reason for this difference has not yet been confirmed.

## 3. Attack Simulation

First, I pinged the Linux OS from Kali Linux to check if the target machine was reachable. After confirming the connection, I performed a controlled authentication failure simulation by entering an incorrect password six (6) times when attempting to connect through SSH.

The attempts generated authentication failure events in the Linux system logs and Wazuh SIEM. Wazuh recorded individual SSH authentication failures under Rule ID `5760` and repeated password failures under Rule ID `2502`. This confirmed that the authentication events were successfully collected and detected for further investigation.

## 4. Detection and Evidence

The detection was verified through the Linux system logs and Wazuh Dashboard. The Linux authentication logs showed that the source IP address was `192.168.1.9`, which attempted to authenticate using the `vboxuser` account.

The Wazuh Dashboard also displayed the corresponding authentication failure events. Rule ID `5760` recorded SSH authentication failures, while Rule ID `2502` identified repeated password failures with a severity level of 10. This higher severity indicates that the event requires further investigation and review.

## 5. Investigation and Analysis

- **Source IP Address:** `192.168.1.9`
- **Targeted Username:** `vboxuser`
- **Number of Failed Attempts:** 6
- **Meaning of Rules 5760 and 2502:** Rule 5760 identifies an SSH authentication failure, while Rule 2502 identifies repeated password failures.
- **Why the Activity Was Suspicious:** Multiple authentication failures were recorded from the same source IP address against the same user account within a short period. Wazuh also generated a higher-severity alert (Level 10), indicating that the activity required further investigation.
- **How Linux Logs and Wazuh Events Correlated:** The Linux system logs recorded the original SSH authentication failures, while Wazuh collected and processed the corresponding events. The source IP address and targeted username were consistent across the evidence, confirming that Wazuh successfully detected the simulated activity.

## 6. Findings and Conclusion

The investigation confirmed that six (6) controlled SSH authentication failures were generated and recorded by the Linux target system. Wazuh successfully collected and detected the corresponding events through its SSH authentication rules.

The activity demonstrated repeated authentication failure behavior commonly associated with SSH brute-force attempts. However, since the activity was intentionally generated in a controlled environment, it does not represent an actual malicious attack.

## 7. Recommended Response

A SOC Analyst should first perform triage and further investigate the event by reviewing the source IP address, targeted account, timestamps, number of failed attempts, and severity level.

After gathering sufficient evidence, the analyst should document the findings and escalate the event to the appropriate security team for further investigation. The analyst should also follow the organization's incident response playbook to determine the appropriate next steps, such as additional monitoring, blocking the source IP address, or applying other security measures when authorized.

## 8. Lessons Learned

Performing a controlled SSH brute-force simulation and investigating SSH authentication failure events were the most important lessons I learned in this lab. I intentionally entered incorrect credentials and then investigated the resulting events using the SIEM tool.

I also learned which event details are important to identify and include in an investigation report, such as the source IP address, targeted username, event count, and severity level. These details are important when documenting findings and escalating security events to higher-level analysts or the appropriate security team.
