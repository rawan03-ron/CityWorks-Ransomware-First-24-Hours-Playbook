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
