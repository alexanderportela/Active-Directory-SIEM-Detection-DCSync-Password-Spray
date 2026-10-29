📖 Overview

This project demonstrates the deployment of a centralized Active Directory Security Operations Center (SOC) dashboard using Splunk Enterprise. The objective was to engineer an end-to-end telemetry pipeline capable of detecting two critical attack vectors: Password Spraying (stealthy brute-force) and DCSync (domain replication abuse). By enforcing Advanced Audit Policies and tuning custom Search Processing Language (SPL) queries, the SIEM now successfully ingests, parses, and visualizes localized attack telemetry, providing real-time visibility into identity compromises and privileged credential dumping.

🏗️ Lab Architecture

    Domain Controller: Windows Server (WIN-CC7CVJNT055 on lab.local)

    Endpoint/Jump Box: Windows 11 Virtual Machine

    SIEM Engine: Splunk Enterprise (Indexer & Search Head)

    Telemetry Agent: Splunk Universal Forwarder

    Offensive Tooling: Mimikatz (DCSync) and Custom PowerShell (Password Spraying)

⚙️ Deployment Lifecycle

    Advanced Audit Policy Enforcement
    To capture the telemetry required for detecting replication abuse, Active Directory's default legacy auditing had to be explicitly overridden.

    Modified the Default Domain Controllers Policy to enforce "Advanced Audit Policy Configurations," specifically enabling Success auditing for "Directory Service Access" to generate Event ID 4662.

    Configured the domain root System Access Control List (SACL) to audit "Replicating Directory Changes" for the Everyone principal, allowing Windows to monitor DCSync requests.

    Diagnosed and bypassed local User Account Control (UAC) suppression by strictly enforcing elevated Administrator contexts for all auditpol verifications and Mimikatz executions.

    Telemetry Ingestion & Resource Management
    The telemetry pipeline required performance tuning to ensure the Splunk alert engine could process the logs without dropping events due to lab hardware constraints.

    Diagnosed skipped searches caused by multiple real-time alerts hitting Splunk's maximum concurrent search limits simultaneously.

    Mitigated the contention by staggering the underlying alert cron schedules to */2 (DCSync) and */3 (Password Spray) minute intervals, optimizing CPU utilization.

    SPL Query Optimization
    Detection logic was re-engineered to account for variables in how the Windows Local Security Authority (LSA) records network information during local vs. remote attacks.

    Discovered that localized password sprays against 127.0.0.1 fail to reliably populate the Source_Network_Address field in Windows Event 4625.

    Engineered a robust Splunk SPL query bypassing the missing IP dependency by utilizing stats dc(Account_Name). This allowed the alert to trigger purely on a mathematical threshold (> 3) of unique targeted accounts.

    Stripped rigid hostname constraints from the DCSync alert logic, ensuring Event 4662 triggers independently of the executing Domain Controller's naming convention.

    SOC Dashboard Visualization

    Pushed the validated, scheduled alert metrics to a centralized Active Directory SOC dashboard.

    Visualized the live telemetry via a Real-Time Alert Queue and prioritized Pie Charts to facilitate immediate analyst triage.

🛡️ Strategic Impact (Blue Team Perspective)
DCSync is notoriously difficult to detect because it abuses legitimate Active Directory replication protocols (MS-DRSR) rather than exploiting a software vulnerability. Similarly, "low and slow" password sprays are specifically designed to evade traditional account lockout thresholds. By forcing granular SACL auditing at the domain root and engineering SIEM alerts to mathematically correlate anomalous event groupings, defenders gain the critical visibility required to identify and intercept these stealthy attacks before threat actors can achieve total domain dominance.
