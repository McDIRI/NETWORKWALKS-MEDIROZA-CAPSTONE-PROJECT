# NETWORKWALKS-MEDIROZA-CAPSTONE-PROJECT
**Black-Box Web Application Penetration Test &amp; Vulnerability Assessment**

<p align="center">
  <img src="https://img.shields.io/badge/Skill-Cybersecurity-404040?style=flat-square&labelColor=C00000" />
  <img src="https://img.shields.io/badge/Ver-JohnTheRipper%20v1.9.0-0070C0?style=flat-square&labelColor=000000" />
  <img src="https://img.shields.io/badge/Skill-Weakness%20Pattern-E87500?style=flat-square&labelColor=000000&logo=kalilinux&logoColor=white" />
  <img src="https://img.shields.io/badge/Skill-Report%20Writting-404040?style=flat-square&labelColor=C00000" />
  <img src="https://img.shields.io/badge/Skill-Hash%20Extraction-238F89?style=flat-square&labelColor=000000" />
  <img src="https://img.shields.io/badge/Penetration%20Testing-C00000?style=flat-square&labelColor=000000&logo=kalilinux&logoColor=white" />
  <img src="https://img.shields.io/badge/Skill-Risk%20Assessment-404040?style=flat-square&labelColor=C00000" />
  <img src="https://img.shields.io/badge/GitHub-404040?style=flat-square&labelColor=0070C0&logo=github&logoColor=white" />
  <img src="https://img.shields.io/badge/Networkwalks%20Hash%20Calculator-404040?style=flat-square&labelColor=C00000&logo=kalilinux&logoColor=white" />
  <img src="https://img.shields.io/badge/NetworkWalks%20Password%20Cracker-404040?style=flat-square&labelColor=C00000" />
  <img src="https://img.shields.io/badge/Ethical%20Hacking-E87500?style=flat-square&labelColor=000000&logo=kalilinux&logoColor=white" />
  <img src="https://img.shields.io/badge/Solomon%20Diri%20INTERN-C00000?style=flat-square" />
</p>

| | |
|---|---|
| **Target** | `https://medirozahospital.com` |
| **Client** | Mediroza General Hospital |
| **Assessment Type** | Full black-box web application penetration test |
| **Assessment Period** | 28 September – 02 october 2026 |
| **Duration** | 5 days |
| **Testing Environment** | Kali Linux (VirtualBox) |
| **Authorization** | Written authorization provided by the client |
| **Excluded Activities** | Social engineering, denial-of-service, out-of-scope testing |
| **Prepared By** | SOLOMON ISAIAH DIRI |
| **Classification** | CONFIDENTIAL |

> This document summarizes a confidential assessment report. Evidence containing patient, staff, or shareholder data has been omitted/redacted here — see [Evidence Handling](#-evidence-handling--redaction) below.

---

## NetworkWalks

```
N E T W O R K W A L K S
```

**Penetration Testing Project — Mediroza General Hospital**
Batch B083 | Week 4
Target: `https://medirozahospital.com`

| | |
|---|---|
| **Client** | Mediroza General Hospital |
| **Type** | Black-box Pentest |
| **Duration** | 5 Days |

> This project was conducted in a controlled environment for educational purposes only. These techniques must never be applied to any system without explicit written permission from the owner.

This assessment was carried out under the **NetworkWalks** Batch B083 internship program — credit and thanks to NetworkWalks for the opportunity to complete this project.

---

## Purpose

The purpose of this engagement was to evaluate the security of the Mediroza General Hospital web application under an authorized black-box test, identify security weaknesses, and demonstrate their practical impact through controlled exploitation.

The assessment addressed three trainer-defined milestones:

1. Attack the website and locate the 3 confidential PDF lab reports of patients.
2. Crack the encryption on all 3 retrieved files.
3. Find the critical data exposure on the client server.

---

## Executive Summary

A three-day black-box penetration test was conducted against the Mediroza General Hospital web application. Testing identified significant weaknesses affecting **authentication**, **protection of confidential patient reports**, and **exposure of internal organizational data**.

The most significant finding was a **SQL injection vulnerability** in the patient portal authentication mechanism. A SQL syntax error confirmed that user-controlled input reached backend SQL processing, and further testing demonstrated a full **authentication bypass**. This exposed three password-protected pathology laboratory reports, whose passwords were subsequently recovered via **dictionary-based cracking** (`pdf2john` + `john.lst`+ 'john tthe ripper' + 'networkwalks hash calculator' + 'networkwalks password cracker').

A separate, unrelated critical exposure was identified through `robots.txt`, which disclosed a legacy `/old/` path. That path served a public directory listing containing an **unauthenticated internal SQL database backup** with confidential staff and shareholder records.

**Overall Risk: CRITICAL** — two Critical findings and one High finding were demonstrated.

| ID | Finding | Severity | Primary Impact |
|---|---|---|---|
| F-01 | SQL Injection -> Patient Portal Authentication Bypass | Critical | Unauthorized access to patient reports |
| F-02 | Weak Password Protection on Patient Reports | High | Recovery of passwords protecting sensitive reports |
| F-03 | Unauthenticated Exposure of Internal Database Backup | Critical | Exposure of staff/shareholder records |

**Sensitive data categories demonstrated as exposed:**

- Patient personally identifiable information (PII)
- Patient laboratory / health information
- Staff personal and employment information
- Staff contact and national identification information
- Staff salary information
- Shareholder and ownership information

---

## Scope & Rules of Engagement

**In scope:**
```
https://medirozahospital.com
```

**Rules of Engagement:**
- Testing limited strictly to the target domain
- Social engineering excluded
- Denial-of-service testing excluded
- Testing outside the agreed scope prohibited
- Conducted under written client authorization

**Environment covered:** public site, patient-facing functionality, staff authentication functionality, and legacy web resources.

---

## Methodology

Testing followed the three trainer-supplied milestones, performed manually via browser and Linux CLI tooling, with controlled exploitation limited to the authorized target.

```
Phase 1  Reconnaissance         -> nmap, curl, robots.txt review
Phase 2  Authentication Testing -> SQL injection probing on patient login
Phase 3  Exploitation            -> auth bypass -> patient portal access
Phase 4  Data Extraction         -> 3 encrypted lab report PDFs retrieved
Phase 5  Online & Offline Cracking   -> networkwalks web base + pdf2john + john the ripper -> all 3 recovered
Phase 6  Further Exposure Check -> robots.txt -> /old/ -> exposed DB backup
```

---

## Tools & Techniques

| Tool / Technique | Purpose |
|---|---|
| Nmap | Service and version enumeration |
| ICMP / ping | Basic host connectivity testing |
| cURL | HTTP response and application reconnaissance |
| Web browser | Manual web application testing |
| SQL injection testing | Authentication and input validation testing |
| `pdf2john` | Extraction of PDF password hashes |
| john the ripper | Dictionary-based password recovery |
| `john.lst` | Common-password dictionary |

---

## Reconnaissance

Service enumeration was attempted with:
```bash
nmap -sV medirozahospital.com
```
No specific Nmap results are asserted in the report, as no captured output was retained.

HTTP reconnaissance:
```bash
curl -i https://medirozahospital.com/patient/login.php
```
This disclosed HTTP/application details including PHP, LiteSpeed, Mediroza CMS version information, and a session cookie — used for technology fingerprinting only; no standalone finding is raised solely on version disclosure.

---

## Findings

### F-01 — SQL Injection Leading to Patient Portal Authentication Bypass
**Severity: CRITICAL**

| Attribute | Details |
|---|---|
| Affected Component | `https://medirozahospital.com/patient/login.php` |
| Category | SQL Injection / Authentication Bypass |
| CWE | CWE-89 — Improper Neutralization of Special Elements used in an SQL Command |
| Primary Impact | Unauthorized access to confidential patient laboratory reports |

**Description:** The patient portal login was vulnerable to SQL injection. Malformed input first produced a MySQL syntax error, and the following username payload achieved a full authentication bypass:

```
admin'--
```

This bypassed the authentication logic entirely, granting access to the patient portal and exposing three password-protected pathology reports.

**Attack Chain:**
```
SQL Injection -> Authentication Bypass -> Patient Portal -> 3 Encrypted Reports
   -> PDF Hash Extraction -> Dictionary Attack -> Password Recovery -> Report Access
```

**Recommendations:**
- Use parameterized queries / prepared statements for all database queries
- Never construct SQL by concatenating user-controlled input
- Implement server-side input validation
- Disable detailed database error messages in production; return generic auth errors
- Implement strong authentication and secure password storage
- Apply authorization checks after authentication
- Review other authentication endpoints for the same weakness
- Perform regression testing after remediation

---

### F-02 — Weak Password Protection on Confidential Patient Reports
**Severity: HIGH**

| Attribute | Details |
|---|---|
| Affected Components | Three password-protected patient laboratory reports |
| Category | Weak Password / Insufficient Protection of Sensitive Files |
| CWE | CWE-521 — Weak Password Requirements |
| Primary Impact | Recovery of passwords protecting sensitive patient reports |

**Description:** All three encrypted PDF reports obtained via F-01 had their password protection defeated using dictionary-based recovery. Hashes were extracted with Networkwalks hash calculator and `pdf2john` and cracked with Networkwalks password cracker and john the ripper against `john.lst`. Different passwords were recovered for each report; all three opened successfully.

# **Technical Procedure:**
 Hash extraction
for patient_report_1.pdf and patient_rport_2.pdf .

 upload file to Networkwalks hash generator to extraction .
 upload the file hash to Networkwalks password cracker

```bash for patient_report_3.pdf
# Extract hash
pdf2john patient_report_1.pdf > hash3-fixed.txt

# john the ripper
john --format=PDF --wordlist=/usr/share/wordlists/john.lst hash3-fixed.txt
 
```

**Impact:** The reports contained patient name and identifier, date of birth and gender, lab reference/specimen data, referring physician information, clinical test results and reference ranges, and abnormality flags — a loss of confidentiality.

**Recommendations:**
- Use strong, randomly generated passwords for sensitive PDFs
- Do not use common or dictionary-based passwords
- Avoid deriving document passwords from predictable patient information
- Prefer application-level authorization and secure document delivery over distributed passwords
- Use stronger document encryption mechanisms
- Review access controls for all sensitive documents

---

### F-03 — Unauthenticated Exposure of Internal Database Backup
**Severity: CRITICAL**

| Attribute | Details |
|---|---|
| Affected Resource | `https://medirozahospital.com/old/` |
| Exposed File | `/old/mediroza_db_backup_2019.sql` |
| Category | Sensitive Information Exposure / Exposure of Backup File |
| CWE | CWE-530 — Exposure of Backup File |
| Primary Impact | Unauthorized disclosure of internal staff and shareholder records |

**Discovery Chain:**
```
robots.txt -> /old/ -> directory listing -> mediroza_db_backup_2019.sql -> unauthenticated access
```

**Description:** `robots.txt` disclosed the `/old/` path (alongside `/patient/` and `/staff/`). Accessing `/old/` revealed a public directory listing containing a SQL database backup, viewable directly through the browser without any authentication.

> **Note:** `robots.txt` is not an access-control mechanism — the impact arises because the disclosed `/old/` resource was itself publicly accessible.

**Data exposed in the backup:**

*Staff table:*
- Full name, job title, department
- Email address and telephone number
- National identification number
- Monthly salary
- Date joined

*Shareholders table:*
- Shareholder name
- Share percentage
- Shares held
- Share class

**Impact:** Unauthorized disclosure of employee PII, contact and national ID information, employment and salary data, and shareholder/ownership information — directly accessible over HTTP(S) with no authentication, and useful for follow-on targeted attacks.

**Recommendations:**
- Remove database backups from publicly accessible web directories
- Store backups outside the web server document root
- Implement strict access controls for backup files
- Disable directory listing on the web server
- Remove obsolete/legacy directories such as `/old/`
- Audit the web root for exposed `.sql`, `.bak`, `.zip`, `.tar` and similar files
- Establish secure backup storage procedures
- Prevent backup files from being served over HTTP/HTTPS
- Perform periodic external checks for exposed backup files

---

## Overall Risk Summary

| ID | Finding | Severity | Confidentiality Impact | Priority |
|---|---|---|---|---|
| F-01 | SQL Injection -> Patient Portal Authentication Bypass | Critical | Severe | Immediate |
| F-02 | Weak Password Protection on Patient Reports | High | High | High |
| F-03 | Unauthenticated Database Backup Exposure | Critical | Severe | Immediate |

**Prioritization:** Remediate F-01 and F-03 immediately. F-02 should follow as a high-priority remediation since it directly weakens protection of already-sensitive patient reports.

---

## Milestone Completion Summary

| Milestone | Trainer Objective | Result | Status |
|---|---|---|---|
| **M1** | Attack the website and find the 3 confidential PDF lab reports of patients | SQL injection auth bypass -> access to 3 password-protected reports | Completed |
| **M2** | Crack the encryption on all 3 retrieved files | Networkwalks hash calculator + Networkwalks password cracker on report 1 & 2 `pdf2john` + john the ripper dictionary attack recovered on patient_report_3.pdf passwords | Completed |
| **M3** | Find the critical data exposure on the client server | `robots.txt` -> `/old/` -> exposed internal SQL backup | Completed |

---

## Recommendations

### Priority 1 — Immediate
- Fix the SQL injection with parameterized queries across all authentication endpoints
- Remove `/old/mediroza_db_backup_2019.sql` from the public web root
- Disable directory listing site-wide

### Priority 2 — High
- Replace weak/static PDF passwords with strong secrets or application-level authorization for report delivery
- Disable verbose database error disclosure in production

### Priority 3 — Ongoing
- Store backups outside the web root, encrypted at rest, with restricted access
- Periodically audit the web root for legacy/forgotten resources and exposed backup files
- Conduct a validation re-test after remediation

---

## Evidence Handling & Redaction

This engagement involved real (simulated) patient, staff, and shareholder data categories. In line with the original report's guidance:

- Evidence containing patient, employee, or shareholder information should be **redacted before any distributed version** of a report is shared.
- Recovered PDF passwords are intentionally **omitted** from this summary.
- Original unredacted evidence should be retained only in an authorized, access-controlled evidence repository.

---

## Conclusion

The assessment identified significant weaknesses across input validation, SQL query construction, authentication controls, sensitive-data access controls, document password management, web-server file exposure, backup management, and legacy resource handling. The Critical SQL injection and database backup exposure findings should be remediated first, followed by the High weak-password-protection finding. A validation assessment is recommended after remediation to confirm the issues have been resolved.

---

## Disclaimer

This report was prepared solely for Mediroza General Hospital in connection with an authorized penetration testing and vulnerability assessment. Testing was restricted to the target domain; social engineering and denial-of-service testing were explicitly excluded. Findings reflect the security posture observed during the assessment period (28 september- 02 october2026) only and should not be interpreted as a guarantee that no other vulnerabilities exist or that the environment will remain secure afterward. This document contains security-sensitive assessment information and should be handled as confidential.

---

**Tags:** `penetration-testing` `web-security` `sql-injection` `authentication-bypass` `information-disclosure` `password-cracking` `pdf-security` `vulnerability-assessment`

*Prepared by SOLOMON ISAIAH DIRI · Report Date: 02 OCTOBER 2026*
