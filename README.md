# Azure-Honeypot - Brute-Force-Investigation-Incident-Closure - Incident Report
Azure Honeypot - Brute Force Investigation &amp; Incident Closure - Alerting &amp; Investigation into bruteforce attempts, checking to see if successful actions were taken against the machine.
Deployed a deliberately exposed Windows VM in Azure to observe real-world attack behaviour and analyse authentication events using Microsoft Sentinel.

 - Detected high volumes of failed logons (EventID 4625) from global IPs, consistent with automated brute-force reconnaissance

 - Analysed all successful logons (EventID 4624) by LogonType

 - Confirmed no interactive compromise:

 - No RDP (LogonType 10), console (2), batch (4), or credential delegation (9) logons

 - Successful events limited to network authentication (LogonType 3) and system services (LogonType 5)

 - Investigated privilege assignment events and validated they were normal system service behaviour

 - Removed permissive NSG rule to contain exposure

 - Performed post-remediation monitoring to confirm no persistence or follow-on activity

Outcome:
Incident classified as low-risk brute-force reconnaissance with no compromise. Demonstrates hands-on SOC investigation, log analysis, KQL querying, cloud security hardening, and incident lifecycle closure. Highlighting the importance of securing infrastructure and properly setting security controls to mitigate potential risk vectors.

Incident Report
Incident ID

SEC1-AZ-0001

Analyst

Date
23/1/2026

15/1/2026 - 22/1/2026

Executive Summary

A Windows-based Azure virtual machine intentionally exposed to the internet experienced a high volume of authentication attempts. Investigation determined the activity to be automated brute-force reconnaissance. No interactive access, credential compromise, or persistence mechanisms were identified. The incident was contained and closed.

Environment Overview

- Cloud Platform: Microsoft Azure

- VM OS: Windows 11

- SIEM: Microsoft Sentinel

- Logging: Azure Monitor Agent (AMA), Log Analytics Workspace

- Initial Exposure: Permissive inbound NSG rule

Detection & Alerting

<img width="1080" height="1080" alt="Untitled design" src="https://github.com/user-attachments/assets/495ae46f-ba40-41a6-adb8-896237b412a4" />
Setting up the alert with schedulued analytics, with added IP entity mapping enrichment. 

Primary detection via Windows Security Events
High volume of:

- EventID 4625 (Failed Logons)

- GeoIP enrichment indicated globally distributed IP sources

Investigation & Analysis:
  Authentication Analysis

<img width="2402" height="1502" alt="scrnli_0JO32b8KwNWGT0" src="https://github.com/user-attachments/assets/6d830091-01dd-400e-b13b-0b0c0024382d" />
Reviewed all EventID 4624 (Successful Logons) against EventID 2625 (Failed Logons)

LogonTypes observed:

- Type 3 – Network authentication

- Type 5 – Service logons

<img width="2000" height="1600" alt="The only successful logons were by a French InfoSec company" src="https://github.com/user-attachments/assets/b5dcfef7-8fb6-4673-801a-3221a85e6e19" />
This IpAddress was the only known entity to achieve "successful" logon attempts, which after checking the address against IP abuse open source sites revealed to be a french infosec company running ipv4 reconnaissance. "Successful" ANONYMOUS LOGON - They established an SMB null session to gather information.
Why it shows as "Successful":
Windows logs anonymous SMB connections as successful logons because technically a session was established.
But NT AUTHORITY\ANONYMOUS LOGON has extremely limited privileges, basically read-only access to public information.
They never accessed actual user account or files.

---

No evidence of:

  - RDP access (LogonType 10)

  - Interactive console logons (LogonType 2)

  - Batch or delegated credential use (Types 4, 9)

Privilege Activity

  - Observed EventID 4672 associated only with service logons

  - Confirmed privileges were assigned to system-managed service accounts

  - No group membership changes or account modifications detected


Containment & Remediation

  - Removed “allow all inbound” NSG rule

  - Reduced external exposure to the VM

  - Maintained logging and monitoring post-remediation


Post-Remediation Monitoring

  - Continued monitoring for:

  - Authentication attempts

  - Privilege escalation

  - Persistence mechanisms

Observed activity:

  - LogonType 5 only

  - No IP addresses

  - No anomalous events


Persistence Checks

Reviewed for:

  - Account creation/modification (4720–4733)

  - Password changes (4723/4724)

  - Service installation (7045)

  - Scheduled task creation (4698)

Result: No persistence indicators found.


Incident Classification & Closure

Incident Type: External brute-force reconnaissance

Severity: Low

Impact: None

Status: Closed



Closure Statement:
All activity was determined to be automated reconnaissance with no compromise. 


Remediation actions were successful, and post-remediation monitoring confirmed system integrity.
