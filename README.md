# AFS SOC - Vulnerability Management & Security Risk Analytics

> **Report Reference:** AFS-SOC-VULN-2026-001  
> **Assessment Period:** 20–26 February 2026  
> **Classification:** Internal Use Only  
> **Prepared by:** SOC Analyst - Emmanuel Ateji  
> **Organisation:** Aureon Financial Systems (AFS)

---

## Overview

This repository contains the complete deliverables for the **Vulnerability Management & Security Risk Analytics** project conducted by the Security Operations Centre (SOC) of Aureon Financial Systems (AFS).

The project assessed two representative target systems using seven scan configurations across five industry-standard tools, identifying and prioritising **119 unique vulnerabilities** across a legacy Linux host and a customer-facing web application. The goal was to transition AFS from a reactive security posture to a structured, risk-based vulnerability management framework.

---

## Targets Assessed

| Target | Identifier | Type | Notes |
|--------|-----------|------|-------|
| Metasploitable 2 | `10.0.2.4` | Linux Host | Ubuntu 8.04 (EOL) - stands in for AFS legacy server infrastructure |
| VulnWeb PHP App | `testphp.vulnweb.com` (44.228.249.3:80) | Web Application | PHP 5.6.40 - represents a customer-facing PHP service |

---

## Key Findings at a Glance

| Severity | Metasploitable 2 | VulnWeb App | Combined Total |
|----------|-----------------|-------------|----------------|
| Critical | 19 | 2 | **21** |
| High | 33 | 3 | **36** |
| Medium | 34 | 4 | **38** |
| Low | 9 | 4 | **13** |
| Informational | 6 | 5 | **11** |
| **Total** | **101** | **18** | **119** |

### Critical Highlights

- **Metasploitable 2** hosts **four confirmed active backdoors** granting immediate root-level access with zero authentication (Ingreslock, vsftpd 2.3.4, UnrealIRCd, ProFTPD 1.3.3c), plus a second exposed Apache Tomcat Manager on port 8180.
- **VulnWeb App** exposes SQL injection across six endpoints and transmits all credentials in cleartext over HTTP.
- Credentialed (SSH) scanning revealed **383+ CVEs** on Metasploitable 2 that unauthenticated scanning alone could not detect.

---

## Tools Used

| Tool | Version | Scan Type |
|------|---------|-----------|
| Tenable Nessus | Latest | Basic unauthenticated + Advanced SSH-credentialed scan |
| Qualys VMDR | Latest | Basic unauthenticated + Advanced SSH-credentialed scan |
| Greenbone OpenVAS | Latest | Network vulnerability scan |
| OWASP ZAP | v2.17.0 | Active web application scan |
| Nikto | v2.5.0 | Web server scan |

---

## Methodology

The assessment followed a structured six-phase approach aligned with **NIST SP 800-115** and the **OWASP Testing Guide v4.2**:

1. **Reconnaissance & Asset Identification** - Port enumeration, OS and service fingerprinting.
2. **Unauthenticated Scanning** - Simulated external attacker perspective across both targets.
3. **Credentialed Scanning (SSH)** - Authenticated deep-dive on Metasploitable 2 via Nessus and Qualys.
4. **Web Application Testing** - Active scanning with ZAP, Nikto, and OpenVAS.
5. **Finding Analysis & Risk Scoring** - CVSS v3.0 scoring, EPSS ratings, cross-tool deduplication and prioritisation.
6. **Reporting** - Consolidated findings for both SOC analyst (technical detail) and senior leadership (executive summary + remediation roadmap).

---

## Compliance Frameworks Referenced

- **PCI-DSS** - Multiple findings constitute likely violations around cardholder data and patch management.
- **GDPR** - Exposure of personal data for EU-resident business users.
- **SOX** - IT controls supporting financial reporting integrity.
- **NIST CSF** - Identify, Protect, Detect, Respond, Recover.

---

## Repository Contents

```
/
├── README.md
├── AFS-Presentation-Slides.pdf         ← You are here
├── AFS_SOC_VulnMgmt_Report.docx        ← Full assessment report
└── vuln-scans-reports/
    ├── Metasploitable 2/
    ├── VulnWeb PHP App/
```

> **Note:** The [`vuln-scans-reports/`](https://github.com/AtejiEmmanuel/afs-soc-vuln-management/tree/main/vuln-scans-reports) folder contains the raw scan outputs referenced as Appendix B in the main report. 


---

## Remediation Summary

The report includes a full prioritised remediation roadmap. Top immediate actions (< 24 hours):

1. **Isolate `10.0.2.4`** from all networks immediately - treat as actively compromised.
2. **Remove all four active backdoors** - Ingreslock (port 1524), vsftpd 2.3.4 (port 6200), UnrealIRCd (ports 6667/6697), ProFTPD 1.3.3c (port 2121).
3. **Upgrade PHP 5.6.40 to PHP 8.3.x** on the VulnWeb application and delete `phpinfo.php`.
4. **Replace all dynamic SQL** with parameterised queries (PDO/MySQLi) across all web endpoints.

---

## Full Project Documentation
For comprehensive details, please refer to the [complete documentation](https://github.com/AtejiEmmanuel/AFS-SOC-Vulnerability-Management/blob/main/AFS_SOC_VulnMgmt_Report.pdf).

## Disclaimer

This assessment was conducted in a controlled lab environment against intentionally vulnerable systems (Metasploitable 2) and a designated test web application (testphp.vulnweb.com). All findings should be validated through manual penetration testing prior to sign-off. Severity ratings use CVSS v3.0 where available; CVSS v2.0 is noted where used.
