# Digital-Forensics-Investigation

![Investigation](https://img.shields.io/badge/Investigation-Forensics-blue?style=flat-square)
![Autopsy](https://img.shields.io/badge/Tool-Autopsy-orange?style=flat-square)
![EZ Tools](https://img.shields.io/badge/Tool-EZ_Tools-green?style=flat-square)
![CY Tech](https://img.shields.io/badge/Institution-CY_Tech-red?style=flat-square)
![Status](https://img.shields.io/badge/Status-Completed-brightgreen?style=flat-square)
![Year](https://img.shields.io/badge/Year-2026-lightgrey?style=flat-square)

> Digital forensics investigation lab — Mastère Spécialisé Cybersécurité & Smart Systems, CY Tech / CY Cergy Paris Université (May 2026).  
> **Case:** Suspected data exfiltration of financial records and intellectual property — *Cartesian Lab / Jean Martin case*.  
> Investigation period: **October 21–23, 2024** — NVMe disk image (E01), Windows 10/11 host `DESKTOP-35OC8BV`.

---

## Table of Contents

1. [Objectives](#objectives)
2. [Technical Environment](#technical-environment)
3. [Methodology](#methodology)
4. [Artifacts Analyzed](#artifacts-analyzed)
5. [Repository Structure](#repository-structure)
6. [Key Findings](#key-findings)
7. [Report](#report)
8. [Legal Disclaimer](#legal-disclaimer)

---

## Objectives

This practical work develops the following competencies:

- Acquiring and validating forensic disk images (E01 format, hash verification)
- Performing NTFS filesystem analysis: MFT parsing, MACB timestamps, deleted file recovery
- Investigating Windows Event Logs to reconstruct user activity and lateral movement
- Conducting Prefetch and registry artifact analysis for program execution evidence
- Detecting steganographic content (LSB encoding) embedded in image files
- Identifying and characterizing encrypted volumes (VeraCrypt)
- Mapping network exfiltration paths via SMB event correlation
- Detecting anti-forensics techniques (log clearing, secure wiping, artifact destruction)
- Producing a structured forensic report with a reconstructed attack timeline and remediation recommendations

---

## Technical Environment

| Tool | Purpose | Version |
|------|---------|---------|
| [Autopsy](https://www.autopsy.com/) | Disk image analysis, artifact extraction, timeline | 4.21+ |
| [MFTECmd](https://github.com/EricZimmermann/MFTECmd) — EZ Tools | MFT parsing, NTFS metadata | Latest |
| [EvtxECmd](https://github.com/EricZimmermann/EvtxECmd) — EZ Tools | Windows Event Log (EVTX) parsing | Latest |
| [WxTCmd](https://github.com/EricZimmermann/WxTCmd) — EZ Tools | Prefetch file analysis (.pf) | Latest |
| [RECmd](https://github.com/EricZimmermann/RECmd) — EZ Tools | Registry hive analysis | Latest |
| [PECmd](https://github.com/EricZimmermann/PECmd) — EZ Tools | Prefetch deep parsing | Latest |
| [SANS SIFT Workstation](https://www.sans.org/tools/sift-workstation/) | Memory analysis, additional Linux tooling | Latest |
| [steghide](https://steghide.sourceforge.net/) | LSB steganography detection and extraction | 0.5.1 |
| [VeraCrypt](https://www.veracrypt.fr/) | Encrypted volume identification and header analysis | 1.26+ |
| Python 3 | Custom parsing scripts, timeline generation | 3.11+ |

---

## Methodology

The investigation followed a structured, court-admissible forensic workflow:

1. **Evidence acquisition and integrity verification** — Validate E01 image hashes (MD5/SHA-256), document chain of custody, mount image read-only.
2. **Filesystem analysis (NTFS)** — Parse `$MFT` with MFTECmd; recover deleted files; analyze MACB timestamps for modification, access, change, and birth times to establish a preliminary timeline.
3. **Windows Event Log analysis** — Process EVTX files with EvtxECmd; focus on EventID 4624 (successful logon), 4648 (explicit credential logon), 4663 (object access), and EventLog clearing events.
4. **Prefetch and execution analysis** — Run WxTCmd and PECmd to identify program execution history, associated volumes, and frequency counts.
5. **Registry artifact analysis** — Use RECmd to examine UserAssist (GUI program execution), ShimCache / AppCompatCache (execution evidence), MRU lists (recently accessed files), and USB device history.
6. **Steganography analysis** — Identify suspicious JPEG files and run steghide to detect and extract hidden payloads (LSB encoding).
7. **Encrypted volume investigation** — Locate and characterize VeraCrypt container headers; attempt passphrase recovery using extracted steganographic credentials.
8. **SMB exfiltration mapping** — Correlate EventID 4648 entries with network share access patterns to map data exfiltration paths and destination targets.
9. **Email forensics** — Analyze Thunderbird profile: MBOX files, sent/received artifacts, attachment metadata.
10. **Anti-forensics detection** — Identify CCleaner registry traces and EVTX entries indicative of log clearing and partial disk wiping.
11. **Timeline reconstruction** — Merge all artifact timestamps into a unified minute-by-minute attack timeline.
12. **Reporting** — Produce structured forensic report: executive summary, evidence table, attack timeline, and remediation recommendations.

---

## Artifacts Analyzed

| Artifact | Tool | Key Finding |
|----------|------|-------------|
| `$MFT` (NTFS Master File Table) | MFTECmd | Recovered deleted financial files and staging directories; MACB anomalies indicating timestamp manipulation |
| Windows Event Log — Security.evtx | EvtxECmd | EventID 4648 cluster confirming explicit credential use; EventLog clearing event detected |
| Windows Event Log — System.evtx | EvtxECmd | Service installation and unexpected shutdown events correlated with wiping activity |
| Prefetch files (`.pf`) | WxTCmd / PECmd | Execution of `robocopy.exe`, `veracrypt.exe`, and staging utilities confirmed |
| Registry — NTUSER.DAT | RECmd | UserAssist entries: VeraCrypt and data staging tools executed repeatedly over the investigation window |
| Registry — SYSTEM hive | RECmd | USB device history; ShimCache entries for unsigned tools not present on disk |
| JPEG image (`vacation_photo.jpg`) | steghide | LSB payload extracted — contains VeraCrypt passphrase |
| VeraCrypt container (`backup.vc`) | VeraCrypt | Encrypted volume identified; passphrase recovery attempted using steganographic credential |
| SMB Event logs | EvtxECmd | EventID 4648: authenticated connections to external share `\\192.168.x.x\fileshare` — exfiltration path confirmed |
| Thunderbird profile (MBOX) | Autopsy | Outbound emails with attachments matching exfiltrated financial records |
| CCleaner registry traces | RECmd | Evidence of artifact destruction: CCleaner run timestamps align with post-exfiltration window |
| Deleted file carving | Autopsy | Partial recovery of SSH private keys and client database exports from unallocated space |

---

## Repository Structure

```
Digital-Forensics-Investigation/
├── README.md
├── methodology/
│   ├── 01_disk_acquisition.md
│   ├── 02_filesystem_analysis.md
│   ├── 03_eventlogs_analysis.md
│   ├── 04_prefetch_registry.md
│   ├── 05_steganography.md
│   ├── 06_veracrypt.md
│   └── 07_network_smb.md
├── artifacts/
│   ├── timeline/
│   │   └── attack_timeline.csv
│   ├── evtx/
│   │   └── suspicious_events.csv
│   └── mft/
│       └── deleted_files.csv
├── scripts/
│   ├── parse_evtx.py
│   └── timeline_generator.py
└── report/
    └── [CONFIDENTIAL - available on request]
```

**Directory notes:**

- `methodology/` — Step-by-step documentation for each investigation phase, tool commands, and observations.
- `artifacts/` — CSV exports from EZ Tools and Autopsy. No raw disk image or EVTX files are included in this repository.
- `scripts/` — Python utilities for EVTX parsing and timeline generation from multi-source artifact CSVs.
- `report/` — The complete forensic report is not published in this repository. See [Report](#report) section below.

---

## Key Findings

- **Intentional, sophisticated exfiltration confirmed** — Financial records, client database, and SSH private keys were staged and transmitted to a remote SMB share over a 48-hour window.
- **LSB steganography** — A VeraCrypt passphrase was concealed inside a JPEG image file to evade detection.
- **VeraCrypt encryption** — Sensitive data was stored in an encrypted container prior to exfiltration; the passphrase was recovered via steganographic extraction.
- **Anti-forensics** — CCleaner was executed immediately after the exfiltration window; partial EVTX clearing was detected, indicating awareness of forensic investigation risk.
- **Attack timeline** — Minute-by-minute reconstruction produced from MFT, Prefetch, Event Log, and registry timestamps spanning October 21–23, 2024.

---

## Report

The complete forensic report — including the full evidence table, annotated attack timeline, chain of custody documentation, and remediation recommendations — is **available on request**.

> **Contact:** [medcheikhmed01@gmail.com](mailto:medcheikhmed01@gmail.com)

The report is treated as **confidential** and is not published in this repository, as it contains detailed exploitation paths and sensitive case reconstruction.

---


**Supervised by:** Mastère Spécialisé Cybersécurité & Smart Systems — CY Tech / CY Cergy Paris Université  
**Lab date:** May 2026

---

## Legal Disclaimer

> All work in this repository was conducted in a **strictly academic and pedagogical context** as part of the Mastère Spécialisé Cybersécurité & Smart Systems program at CY Tech / CY Cergy Paris Université.
>
> The disk image, case files, user accounts, and all data used in this investigation are **entirely fictitious**. Any resemblance to real persons, organizations, or events is purely coincidental.
>
> The techniques documented here are shared for **educational purposes only**. The authors do not condone or encourage unauthorized access to computer systems or any illegal activity.  
> Applying these techniques outside of authorized, controlled environments may violate applicable law.

---

<p align="center">
  <sub>CY Tech · Mastère Spécialisé Cybersécurité &amp; Smart Systems · 2025–2026</sub>
</p>
