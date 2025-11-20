# SysAdmin Backup and Recovery Evaluation

**A Comparative Analysis of Windows and Linux Backup Tool Reliability**

**Course:** BFOR419/519
**Project Dates:** October 11 – November 18, 2025

---

## Introduction

This project evaluates the practical reliability of native backup tools in Windows 11 and Ubuntu 22.04. The goal was to measure how well each system performs during normal backups, recoveries, and failure scenarios, including deletion, corruption, and basic cybersecurity threats.

Rather than comparing full operating system images (which differ drastically in structure and size), the study focused on a controlled 31MB dataset designed to resemble a realistic user directory. The project emphasizes not just performance, but *truthfulness* of restores—whether the recovered data is actually correct, verifiable, and trustworthy.

View:
- sysadmin project.pdf to see some of the screenshots of the project as I was doing it.
- Windows evidence scripts to see a list of scripts I used for windows vm.
- Linux evidence scripts to see a list of scripts I used for Ubuntu vm
- project overview summary for a quick look and understanding of what I did in the project.
- Project log to see real time step by step what I was doing.
---

## Dataset and Integrity Baseline

To ensure a fair comparison, a standardized dataset was manually generated and replicated on both systems. It included:

* A **Documents** folder (Word documents, text files)
* A **Photos** folder (PNG files)
* A **Database** folder (a CSV dataset)

Before any backups were performed, **cryptographic checksums** were generated using:

* `Get-FileHash` on Windows
* `sha256sum` on Linux

These checksums served as the “ground truth” for measuring data integrity during all restore tests.

---

## Tools and Environment

All experiments were performed on a Windows 11 host using **Oracle VirtualBox**, with the following configuration:

| Component              | Configuration                                 |
| ---------------------- | --------------------------------------------- |
| Windows VM             | Windows 11 Pro                                |
| Linux VM               | Ubuntu 22.04 LTS                              |
| Windows Backup Tool    | Backup and Restore (Windows 7) → External USB |
| Linux Backup Tool      | rsync → Local BACKUP/ directory               |
| Integrity Verification | SHA-256 checksum comparison                   |

Two separate types of failures were tested: deletion-based and corruption-based, followed by cybersecurity-focused scenarios.

---

## Phase 1: Attempting Windows File History

The initial plan was to use Windows **File History**, the recommended user-facing backup solution. However, during testing, File History repeatedly failed to produce usable backups:

### Symptoms

* The backup output was consistently ~2MB (logs only), instead of ~31MB.
* The tool refused to scan or include the project directory correctly.

### Troubleshooting Attempts

The following remediation steps were performed:

1. Reset File History’s Exclude list
2. Moved the dataset into different Windows libraries
3. Re-initialized File History configuration
4. Deleted all related `%appdata%` and `AppData\Local` configuration folders
5. Reformatted and reselected the backup drive

None of these attempts resolved the issue.

### Outcome

File History was deemed **non-functional for this project**, and the study pivoted to the legacy **Backup and Restore (Windows 7)** utility, which immediately worked.

---

## Phase 2: Establishing Baseline Backup Performance

### Backup Execution Steps

For consistency, the following steps were taken on both systems:

1. Create the dataset
2. Generate SHA-256 checksums
3. Perform initial full backup
4. Add a single new text file
5. Perform incremental backup
6. Measure backup time and output size

### Results

#### Full Backup

| Tool                       | Time         | Resulting Size       |
| -------------------------- | ------------ | -------------------- |
| Windows Backup and Restore | ~15 seconds  | 7.37 MB (compressed) |
| rsync                      | ~0.5 seconds | 31 MB (1:1 mirror)   |

#### Incremental Backup

| Tool                       | Time         | Size Change |
| -------------------------- | ------------ | ----------- |
| Windows Backup and Restore | ~24 seconds  | No change   |
| rsync                      | ~0.5 seconds | No change   |

### Analysis

* **Windows** emphasizes storage efficiency through automatic compression but suffers from high overhead, especially during incremental scans.
* **Linux rsync** emphasizes exact replication and minimal overhead, producing near-instant backups at the cost of larger storage usage.

---

## Phase 3: Recovery from Minor Failure (Deletion Test)

### Testing Steps

1. Permanently delete the **Photos** folder
2. Permanently delete the **Database CSV file**
3. Attempt restore using native tools
4. Compare restored files with original checksums

### Results

| Metric         | Windows Backup and Restore    | Linux rsync                   |
| -------------- | ----------------------------- | ----------------------------- |
| Restore Time   | ~60 seconds                   | ~0.5 seconds                  |
| Usability      | GUI-based, straightforward    | Single command                |
| Data Integrity | Excellent (matched checksums) | Excellent (matched checksums) |

### Summary

Both systems handled deletion recovery effectively. Windows offered a strong GUI experience, while rsync was dramatically faster.

---

## Phase 4: Recovery from Critical Failure (Corruption Test)

This phase simulated file-level corruption (e.g., bit rot).

### Corruption Steps

1. Open a PNG file in a hex editor
2. Overwrite the first bytes with zeros
3. Attempt to restore the valid file from backup
4. Verify integrity using SHA-256 checksums

### Linux Result

* rsync immediately recognized the change
* Restored the clean backup file
* Final checksum matched the original
* Outcome: **Successful automatic self-healing**

### Windows Result

* Restoration prompted “Replace the file,” implying success
* However, checksum comparison showed the file remained corrupted
* The restore was silently blocked due to a file lock (likely OneDrive or system permissions)
* No error message was shown
* Outcome: **Silent restore failure**

### Critical Conclusion

The Windows backup appeared successful to the user but **did not restore the file**, representing the most severe finding of the project.

---

## Phase 5: Cybersecurity Stress Tests

### Scenario 1: Confidentiality (Stolen Backup)

A simulated attacker was given direct access to the backup storage.

| System                        | Result                      |
| ----------------------------- | --------------------------- |
| Windows Backup (.zip archive) | Fully readable, unencrypted |
| Linux rsync directory         | Fully readable, unencrypted |

**Conclusion:**
Neither system encrypts backups by default. Additional measures (BitLocker, LUKS, encrypted containers) are required for confidentiality.

---

### Scenario 2: Availability (Ransomware Simulation)

With the backup drive plugged in (Windows) or mounted (Linux), a simulated ransomware event deleted both:

* The original dataset
* The connected backup storage

**Outcome for Both:**
100% data loss. No recovery possible.

**Conclusion:**
An “online” backup is not a true backup. An **offline**, **air-gapped**, or **immutable** backup is required to survive ransomware.

---

## Final Conclusions

This project demonstrated that:

1. **rsync** consistently provided fast, predictable, and verifiable backups.
2. **Windows Backup and Restore** was functional but suffered a silent failure in the corruption test, undermining trust.
3. Neither system offered built-in encryption or ransomware-resistant backups.
4. The **3-2-1 Backup Rule** is essential for real-world resilience.

---

## Key Recommendations

* Always verify restores using cryptographic hashes.
* Avoid relying solely on consumer-facing tools (e.g., File History).
* Use encrypted drives for backup media.
* Maintain at least one offline or immutable backup copy.

References

AI was used to help me make the tables in this read me file for organization and to help with neat formatting as I was unsure how to do it.

