Project Overview

This project demonstrates the configuration of a Splunk Enterprise SIEM to ingest, parse, and visualize Active Directory attack telemetry from a Windows Server Domain Controller. The objective was to create a Real-Time SOC Dashboard capable of detecting brute-force attempts (Password Spraying) and replication abuse (DCSync) while overcoming common telemetry pipeline failures.

Architecture & Tools

    Hyper-V Infrastructure: Windows Server Domain Controller (lab.local), Windows 11 Endpoint.

    SIEM: Splunk Enterprise (Search Head & Indexer), Splunk Universal Forwarder.

    Offensive Tooling: PowerShell (custom spray scripts), Mimikatz (DCSync).

Key Challenges & Detection Engineering

    SPL Query Optimization: Initial Password Spray alerts failed because local attacks against 127.0.0.1 do not reliably populate the Source_Network_Address field in Event 4625. Rewrote the alert logic to rely strictly on stats dc(Account_Name) to trigger the > 3 threshold regardless of network source.

    Advanced Audit Policy Enforcement: DCSync attacks initially failed to generate Event 4662 logs. Diagnosed that default GPOs do not audit Directory Service Access. Engineered a fix by modifying the Default Domain Controllers Policy to force Advanced Audit Policy Configuration and updated the domain root SACL to audit "Replicating Directory Changes."

    Cron Job Contention: Simultaneous real-time alerts caused Splunk to hit maximum concurrent search limits, silently dropping alerts. Mitigated by staggering cron schedules (*/2 and */3) to optimize VM resource utilization.

    Privilege Execution: Diagnosed UAC interference preventing auditpol from verifying policy enforcement and Mimikatz from interacting with LSASS.

Outcomes
Successfully deployed a centralized Active Directory SOC dashboard that visualizes Event 4625 (Failed Logon) and Event 4662 (Directory Service Access) telemetry with automated alert triggers.

Consolidated Reading Resources

    Event 4625 (Failed Logon): Search the Microsoft Learn documentation for "Event ID 4625." Pay special attention to the "Failure Reason" codes (like 0xC000006A for bad passwords) and how Windows logs local vs. network logons.

    Event 4662 & DCSync Mechanics: Read the Microsoft documentation on "Audit Directory Service Access" and "Event 4662." For the offensive side, search for "Harmj0y DCSync" to read the original breakdown of how Mimikatz abuses the Directory Replication Service (DRS) Remote Protocol.

    Advanced Audit Policies vs. Legacy Auditing: Search for "Microsoft Advanced Audit Policy Configuration." This will explain why your lab reverted and why modern SOCs force advanced policies to override the legacy category settings.

    Splunk SPL stats Command: Review the Splunk documentation for the stats command, specifically the dc() (distinct count) function you used today. Understanding how by clauses group data—and why they fail when a field is missing—is critical for SIEM engineering.
