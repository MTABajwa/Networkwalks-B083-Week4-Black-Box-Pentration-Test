# Networkwalks-B083-Week4-Black-Box-Pentration-Test

# Mediroza General Hospital - Black-Box Penetration Test

## 📌 Project Overview
This repository contains the documentation and evidence for a black-box penetration test conducted against **Mediroza General Hospital** (`https://medirozahospital.com`). This engagement was completed as part of the Networkwalks Cybersecurity Internship (Batch B083, Week 4).

The objective was to identify vulnerabilities, exploit them to demonstrate real-world impact, and document all findings in a professional report.

## 🎯 Scope & Rules of Engagement
- **Target:** `https://medirozahospital.com`
- **Type:** Black-Box Web Application Penetration Test
- **Authorization:** Written permission granted by the client.
- **Rules:** 
  - Testing limited to the target domain only.
  - No social engineering.
  - No denial of service (DoS).
  - No testing outside the agreed scope.

## 🛠️ Tools & Technologies Used
- **Reconnaissance:** `whois`, `dig`, `nmap`, `whatweb`
- **Enumeration:** `gobuster`, Burp Suite, Firefox
- **Exploitation & Cracking:** `pdf2john`, `pdfcrack`, `john` (John the Ripper), `qpdf`
- **Data Extraction & Analysis:** `exiftool`, `wget`, `grep`, `pdftotext`

## 🚀 Exploitation Walkthrough (Milestones)

### Milestone 1: Initial Access (Retrieve 3 Confidential PDFs)
1. **Reconnaissance:** Conducted port scanning (`nmap`) and directory brute-forcing (`gobuster`) with a custom User-Agent to bypass WAF 403 errors.
2. **Vulnerability Discovery:** Discovered an exposed directory `/patient/` containing `login.php`.
3. **Exploitation (SQL Injection):** Bypassed the Patient Login authentication using the SQLi payload `admin'--` in the Staff ID field. 
4. **Result:** Gained unauthorized access to the Patient Portal and downloaded 3 encrypted PDF lab reports (S. Dlamini, P. Reddy, E. Thompson).

### Milestone 2: Data Extraction (Cracking the PDFs)
1. **Analysis:** Used `pdfinfo` to confirm the files were encrypted.
2. **Hash Extraction:** Extracted password hashes using `pdf2john`.
3. **Cracking:** Used `pdfcrack` and `john` with the `rockyou.txt` wordlist to crack the encryption.
4. **Result:** Successfully recovered the passwords for all 3 files (`123456`, `password`, `!@#$%^&`). Decrypted the PDFs using `qpdf`.

### Milestone 3: Attack (Critical Data Exposure)
1. **Metadata Analysis:** Ran `exiftool` on the unlocked PDFs. Found a hidden comment in `report3_unlocked.pdf`: *"DB backup moved to /old before site migration, do not delete"*.
2. **Directory Traversal:** Navigated to the exposed `/old/` directory on the web server.
3. **Data Exfiltration:** Downloaded the exposed database backup file `mediroza_db_backup_2019.sql`.
4. **Data Extraction:** Used `grep` and `cat` to extract sensitive data:
   - **Employee Salaries:** 30 records containing names, job titles, departments, emails, and monthly salaries.
   - **Shareholder Details:** 10 records containing names, share percentages, shares held, and share classes.

## 📊 Risk Rating

| Vulnerability | Risk Level | Impact |
| :--- | :--- | :--- |
| **SQL Injection (Staff Login)** | **Critical** | Authentication bypass, unauthorized access to internal portals. |
| **Exposed Database Backup** | **Critical** | Complete exposure of employee salaries and shareholder financial data. |
| **Weak PDF Encryption** | **High** | Compromise of patient confidentiality due to trivially crackable passwords. |
| **Information Disclosure (Metadata)** | **High** | Leaks internal server paths and architectural details. |

## 🛡️ Recommendations & Remediation
1. **Fix SQL Injection:** Use parameterized queries (prepared statements) and strict input validation.
2. **Secure Backups:** Immediately remove `mediroza_db_backup_2019.sql` from the public web server. Store backups offline or in a secure, access-controlled cloud environment.
3. **Strengthen PDF Encryption:** Enforce strong, unique passwords for all patient files. Consider delivering reports via a secure authenticated portal instead of encrypted PDFs.
4. **Sanitize Metadata:** Strip all metadata (comments, author, software versions) from files before publishing them to the web.
5. **Implement a WAF:** Deploy a Web Application Firewall to block SQLi payloads and directory brute-forcing attempts.

