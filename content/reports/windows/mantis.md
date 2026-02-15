---
title: "Engagement: Mantis (Windows Server 2008 R2)"
date: 2026-02-15
tags: ["Windows", "Active Directory", "MS14-068", "MSSQL", "Kerberos"]
summary: "A Critical-risk assessment resulting in full Domain Compromise via Kerberos Golden PAC exploitation."
---

## Executive Summary
This engagement targeted the **Mantis** infrastructure (10.10.10.52), a Windows Server 2008 R2 environment functioning as a Domain Controller.

The assessment identified **3 material findings**, culminating in a critical attack chain that allowed for the complete compromise of the Active Directory domain.

### Key Findings
* **Critical:** Domain Privilege Escalation via Kerberos Checksum Validation (MS14-068).
* **High:** Sensitive Information Disclosure via Publicly Accessible Web Directory.
* **Medium:** Insecure Storage of Credentials (Cleartext Database Entries).

### Attack Chain Overview
1.  **Initial Access:** Enumeration of a public web directory (`/secure_notes/`) revealed a Base64-encoded filename containing administrative credentials for the MSSQL service.
2.  **Lateral Movement:** Authenticated access to the **OrchardDB** database exposed a table (`blog_Orchard_Users`) containing cleartext credentials for the domain user `james`.
3.  **Domain Compromise:** The `james` account was leveraged to exploit **CVE-2014-6324 (MS14-068)**. This allowed for the forgery of a Kerberos Ticket-Granting Ticket (TGT) with "Domain Admin" privileges, granting `NT AUTHORITY\SYSTEM` access.

---

### Download Full Report
This report was generated using **SysReptor** to simulate a corporate deliverable.

<div align="center">
  <a href="/reports/mantis-engagement-report.pdf" style="background-color: #3b82f6; color: white; padding: 10px 20px; text-decoration: none; border-radius: 5px; font-weight: bold;">
    Download Professional PDF Report
  </a>
</div>