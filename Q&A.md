# IT Audit Q&A

This document provides standardized answers to common Information Technology (IT) Audit and Information Security questions regarding the **Gumelar Document Generator**. It is intended to assist IT Auditors, Security Analysts, and Compliance teams in understanding the application's architecture, data flows, and security posture.

## 1. Architecture & Deployment

**Q1.1: What is the overall architecture and deployment model of the application?**
**Answer:** The Gumelar Document Generator is a **single-tier thick client desktop application**. It is built using React (web frontend) wrapped inside a native desktop executable using **Tauri** (Rust-based). It does not rely on a centralized backend server, cloud service, or external databases for its core operations.

**Q1.2: What operating systems are supported?**
**Answer:** The application is compiled primarily for Windows operating systems (Windows 10/11) distributing via MSI/NSIS installation packages.

**Q1.3: Does the application require an active internet connection to function?**
**Answer:** No. The application is completely **offline** by design. It does not phone home, require cloud licensing, or synchronize data across the internet.

---

## 2. Data Storage & Privacy

**Q2.1: Where is application data stored?**
**Answer:** All operational data (projects, timelines, timesheets, handover forms, and personnel details) is stored locally on the user’s workstation. The application utilizes **IndexedDB** governed by the `Dexie.js` wrapper, which stores data within the user's local AppData directory (`%APPDATA%` or `%LOCALAPPDATA%`) managed by the system's WebView2 browser control.

**Q2.2: Does the application transmit any Personally Identifiable Information (PII) or corporate data?**
**Answer:** No. Because the application operates completely offline, no PII, user data, or corporate data ever leaves the local machine. There is zero telemetry, analytics, or remote logging.

**Q2.3: How is data separated between different users on the same machine?**
**Answer:** Data separation relies on the operating system's Local User Profiles. The WebView2 data folder is created inside the specific local user's AppData directory. Another Windows user on the same machine cannot access the database unless they have administrative access or compromise the OS profile.

---

## 3. Storage Security & Encryption

**Q3.1: Is data encrypted at rest?**
**Answer:** The application itself does not apply app-level AES encryption to the IndexedDB databases natively. Instead, it **relies on OS-level Full Disk Encryption (FDE)**. For compliance, the host workstation MUST have BitLocker (or an equivalent disk encryption solution) enabled. 

**Q3.2: Is data encrypted in transit?**
**Answer:** Not Applicable. Since the system does not transmit data over local networks or the internet, there is no "data in transit."

---

## 4. Authentication & Authorization

**Q4.1: How does the application authenticate users?**
**Answer:** The application delegates authentication to the underlying Operating System. There is no native login screen. If an employee can successfully authenticate to their Windows workstation (e.g., via Active Directory, Windows Hello, or Smart Card), they can execute the software.

**Q4.2: Are there role-based access controls (RBAC) within the application?**
**Answer:** No. The application operates in a single-user context meant for the local project manager or engineer. Role-based access is not applicable. Access control should be restricted via Windows NTFS file permissions on the application directory.

---

## 5. Vulnerability & Patch Management

**Q5.1: How are third-party software vulnerabilities handled?**
**Answer:** The application utilizes package managers (`npm` for frontend, `cargo` for Rust/Tauri backend). Dependency vulnerabilities are audited utilizing `npm audit` and `cargo audit`. Updates to these dependencies will require a new version of the MSI installer to be deployed to the endpoints.

**Q5.2: What is the attack surface of the application?**
**Answer:** The attack surface is minimal due to the absence of active listening ports, server-side APIs, or cloud infrastructure. The primary threat vector would involve local malware compromising the workstation's AppData directory, or malicious input vectors. User inputs are sanitized utilizing React's built-in XSS protection mechanisms.

---

## 6. Backup and Disaster Recovery

**Q6.1: What is the backup and recovery process?**
**Answer:** The application does not perform automatic cloud replication. Disaster Recovery (DR) depends on two procedures:
1. **Host-Level Backups:** IT administrators should enable workstation backups (e.g., OneDrive Known Folder Move, or centralized endpoint backup agents) targeting the user's AppData.
2. **Artifact Generation:** The primary function of the app is to export `.docx` and `.xlsx` artifacts. Users save these artifacts to synced corporate drives (e.g., SharePoint, OneDrive).

---

## 7. Change Management & Code Signing

**Q7.1: How is the integrity of the application enforced?**
**Answer:** To ensure that the executable has not been tampered with and comes from a trusted source, the Windows binaries and MSI installers are **Code Signed** using a trusted digital certificate (e.g., EV Code Signing Certificate) during the CI/CD compilation process.

**Q7.2: Are build pipelines automated and secure?**
**Answer:** Tauri compilation occurs off-environment using secure build chains. The code-signing process is fully integrated into the Tauri bundler configuration (`tauri.conf.json`).
