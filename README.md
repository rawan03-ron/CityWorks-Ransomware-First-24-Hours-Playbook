# 🔒 Enterprise Incident Response Playbook: Ransomware (First 24 Hours)

**Organization:** CityWorks Utilities (~300 Endpoints, 1 Data Center)  
**Incident Commander / Lead Analyst:** Rawan Semrin  
**Program:** NCSC — Masar Cybersecurity Training (Incident Handling & Crisis Management)  
**Time Standard:** `UTC`  
**Classification:** `TLP:WHITE` — Educational & Operational Playbook  

---

## 1. Incident Timeline & Operational Context

* **03:42 UTC:** Abnormal file modification behavior reported across two office zones.
* **03:47 UTC:** File server `FS-02` exhibits rapid file changes; volume snapshot gap detected since `02:59 UTC`.
* **03:50 UTC:** Ransom note discovered on workstation `FIN-WS-11` referencing confidentiality and extortion demands.
* **03:52 UTC:** Privileged service account `svc_backup` logs multiple failed authentication attempts from `10.20.30.77` (`CARE-WS-07`) followed by a successful login.
* **03:55 UTC:** Off-site backup job triggers a warning alert.
* **03:58 UTC:** Executive leadership requests a Go/No-Go decision on opening Call Center operations at `05:00 UTC`.

---

## A. Trigger & Scope Criteria

This playbook is activated immediately when any of the following high-fidelity triggers occur:
* **Cryptographic File Activity:** Rapid file renaming, encryption extensions, or mass modification alerts on shared file servers (e.g., `FS-02`).
* **Extortion / Ransom Artifacts:** Discovery of ransom notes or unauthorized text/HTML files referencing data confidentiality or payment instructions (e.g., `FIN-WS-11`).
* **Privileged Identity Anomalies:** Brute-force or spray patterns against service/administrative accounts (`svc_backup`) followed by successful authentication from an unexpected internal host (`10.20.30.77`).
* **Backup & Snapshot Disruption:** Unexplained gaps in volume shadow copies/snapshots (since `02:59 UTC`) or warning/failure alerts on nightly/off-site backup jobs (`BK-01`).

---

## B. Incident Decision Tree (Initial Signals vs. Ongoing Impact)

```text
                        [ High-Fidelity Alert / Trigger ]
                                        │
                    ┌───────────────────┴───────────────────┐
                    ▼                                       ▼
      [ BRANCH 1: PRE-IMPACT ]                [ BRANCH 2: ONGOING IMPACT ]
  (Suspicious Credential Activity /         (Active Encryption on FS-02 /
    No Confirmed Encryption Yet)             Ransom Note / Cross-VLAN Spread)
                    │                                       │
                    ▼                                       ▼
         CONTAINMENT ACTION:                     CONTAINMENT ACTION:
   • Disable compromised account           • Disable svc_backup immediately
   • Block source IP at firewall           • Segment Finance & CARE VLANs
   • Hunt for lateral movement             • Take FS-02 file services offline
                    │                                       │
                    ▼                                       ▼
         EVIDENCE CHECKPOINT:                    EVIDENCE CHECKPOINT:
   • Auth logs (Source/Dest IPs)           • Memory dump (CARE-WS-07 & FS-02)
   • Failed/Success login timeline         • Copy of ransom note & snapshot logs
```

| Operational State | Response Branch | Mandatory Containment Action | Mandatory Evidence Checkpoint |
| :--- | :--- | :--- | :--- |
| **Pre-Impact** *(Suspicious Credential Use)* | Observation & Active Threat Hunting | Disable suspected account (`svc_backup`); block source IP (`10.20.30.77`) at internal firewall. | Identity authentication logs (Source/Dest IP, timestamps, failed/successful login counts). |
| **Ongoing Impact** *(Active Encryption / Lateral Spread)* | Active Ransomware Crisis Response | Segment `Finance` and `CARE` VLANs via switch ACLs; disable `svc_backup`; take `FS-02` file services offline. | Volatile memory (`RAM`) dumps from `CARE-WS-07` and `FS-02`; copy of ransom note; backup job logs. |

---

## C. Containment Strategy Matrix & Selection

| Evaluation Criteria | Option 1: Targeted Endpoint Isolation | Option 2: Immediate Account Controls | Option 3: Lateral Communication Restrictions |
| :--- | :--- | :--- | :--- |
| **Speed** | **Slow** (20–30 min manual NetOps ACL) | **Fast** (Immediate via AD/IdP) | **Moderate** (NetOps switch/router ACLs) |
| **Business Impact** | **Low** (Single workstation offline) | **Moderate** (Halts scheduled backup jobs) | **High** (Isolates Finance & CARE departments) |
| **Reversibility** | **Easy** (Re-enable switch port) | **Easy** (Re-enable account / rotate password) | **Complex** (Requires coordinated routing/ACL rollback) |
| **Evidence Considerations** | Preserves live host state if Mgmt VLAN stays open | Preserves authentication & session logs | Prevents further network spread while preserving host artifacts |
| **Owner** | NetOps Team | SysAdmin / Identity Team | NetOps / Security Administration |
| **Decision Criteria** | Single workstation confirmed infected with zero lateral spread | Privileged or service credential compromise suspected | Multi-host or cross-VLAN lateral movement detected (`CARE` ➔ `FS-02`) |
| **Rollback Plan** | Re-enable switch port after clean-image verification | Re-enable service account with newly rotated credentials | Restore standard inter-VLAN routing and ACLs post-eradication |

### Selected Strategy & Operational Rationale
* **Selected Combination:** **Option 2 (Immediate Account Controls)** + **Option 3 (Lateral Communication Restrictions)**.
* **Rationale:** Because one-click endpoint isolation is unavailable and `svc_backup` is confirmed compromised, immediately disabling `svc_backup` strips the attacker of domain-wide administrative leverage within minutes. Simultaneously, because lateral movement has already crossed from `CARE-WS-07` to `FS-02` and `FIN-WS-11`, segmenting the `Finance` and `CARE` VLANs is essential to shield **Call Center operations** (opening at `05:00 UTC`) from becoming the next encryption target.

---

## D. Recovery & Restore Preconditions

Restoration from nightly disk backups on **`BK-01` (pre-`02:59 UTC` clean state)** must only proceed once all preconditions below are satisfied:

1. **Threat Eradication:** Confirmed removal of all malware binaries, scheduled tasks, and persistence mechanisms on `CARE-WS-07`, `FIN-WS-11`, and `FS-02`.
2. **Clean-Room Validation:** Test restoration of critical `FS-02` volumes in an isolated sandbox environment, verified clean via full antimalware/EDR scans prior to production cutover.
3. **Global Credential Reset:** Mandatory password rotation across all service accounts (`svc_backup`), domain administrators, and affected departmental users.
4. **Recovery Order & Integrity Verification:** Restore core identity/infrastructure services first, followed by `FS-02` file shares and departmental endpoints; verify file integrity checksums to confirm zero encrypted files were captured in the restore point.

---

## E. Crisis Communications Templates

### 1. Executive Leadership Update (`04:30 UTC`)
> **Subject:** `[HIGH PRIORITY]` Ransomware Incident Update — `04:30 UTC`  
> **Status:** CityWorks Utilities is actively managing a confirmed ransomware incident affecting file server `FS-02` and select workstations in the Finance and CARE departments.  
> **Operational Impact:** File server `FS-02` has been taken offline for containment, and Call Center operations scheduled for `05:00 UTC` are temporarily paused to guarantee network isolation and protect municipal service integrity.  
> **Actions Taken:** Compromised service account `svc_backup` has been disabled, and NetOps has applied network segmentation isolating the CARE and Finance VLANs to halt further spread.  
> **Next Decision Point & Update:** Forensics acquisition and integrity verification of pre-incident backups (`BK-01`, prior to `02:59 UTC`) are underway to determine the Call Center restoration timeline. Next executive briefing scheduled for **`06:00 UTC`**.  
> **Incident Commander:** Rawan Semrin

### 2. All-Staff Operational Advisory
> **Subject:** `[URGENT]` IT Systems Maintenance & Security Advisory  
> **Guidance:** CityWorks Utilities is currently experiencing a critical IT systems disruption. Effective immediately, all staff are instructed **not to log into, power off, or restart** office computers until further notice from the IT desk. If you observe an unusual text message or warning window on your screen, do not click anything or touch the keyboard—report your workstation number immediately to the IT Helpdesk. Our technical teams are actively working to secure and restore all services safely.

---

## F. Phase Transition & Incident Closure Criteria

| Gate / Milestone | Explicit Verification & Documentation Criteria |
| :--- | :--- |
| **Containment ➔ Eradication** | `svc_backup` disabled; `CARE` and `Finance` VLANs segmented; `FS-02` isolated; volatile memory (`RAM`) dumps and ransom note artifacts preserved with chain-of-custody logs. |
| **Eradication ➔ Recovery** | Infected endpoints wiped/re-imaged; all persistence mechanisms removed; clean-room test restore from `BK-01` (pre-`02:59 UTC`) validated malware-free; privileged passwords rotated. |
| **Formal Incident Closure** | **1) Normalization:** `48 hours` of post-restoration monitoring with zero anomalous file activity or unauthorized logins.<br>**2) Evidence Archiving:** Full forensic images of `CARE-WS-07` and `FS-02` logs archived securely for legal and cyber-insurance review.<br>**3) Root-Cause Patching:** Vulnerability/misconfiguration enabling `svc_backup` compromise remediated.<br>**4) Post-Incident Review (PIR):** Formal lessons-learned review completed within **`72 hours`** and Risk Register updated. |
