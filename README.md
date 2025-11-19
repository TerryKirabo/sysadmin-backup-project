# sysadmin-backup-project
Measuring Effectiveness of Backup and Recovery Solutions in Virtualized Environments

A comparative Analysis of Windows vs. Linux Backup Tool Resilience

Course:BFOR419/519 - (6854;7037)

Date of project: October 11th to November 18, 2025




1. Abstract

This project analyzes the real-world effectiveness of built-in backup tools on Windows 11 and Ubuntu 22.04. The analysis evaluates data integrity (RPO), recovery speed (RTO), and, most critically, resilience against modern cybersecurity threats. The built-in Windows "File History" tool was first attempted and found to be non-functional, prompting a pivot to the "Backup and Restore (Windows 7)" utility. This was compared against the Linux rsync command. Tests included baseline performance, recovery from file deletion, and recovery from data corruption. The findings reveal a stark "apples-to-oranges" trade-off between Windows' storage efficiency and Linux's speed, a critical silent failure in the Windows restore process, and the absolute vulnerability of both tools' default configurations to modern cyber threats.

2. Methodology & Lab Setup

This project was conducted on a single host machine running Windows 11, with both Windows and Linux environment virtualized in Oracle Virtualbox.

Windows System: Windows 11 Pro

Linux System: Ubuntu 22.04 LTS

Windows Backup Tool: "Backup and Restore (Windows 7)" utility, backing up to an external USB drive.

Linux Backup Tool: rsync command-line utility, backing up to a separate BACKUP/ directory on the same virtual disk.

Verification Tool: Get-FileHash (PowerShell) and sha256sum (Linux) were used to create "ground truth" checksum files to mathematically verify data integrity (RPO) after every restore operation.

Standardized Test Dataset

To ensure a fair and direct comparison, this project intentionally avoided backing up the full operating systems, which differ greatly in size and file types. Instead, a standardized 31MB test dataset was created and identically replicated on both systems. This dataset simulated a real-world user folder, consisting of:

.png image files

.docx documents

.csv database files

.txt text files

3. Findings, Part 1: The Initial "File History" Failure

A primary objective was to test modern, default tools. The initial attempt to use the Windows 11 "File History" tool from the Control Panel was a complete failure.

Symptom: The tool would run, but the resulting backup on the external drive was only 2MB, indicating it was only saving logs.

Troubleshooting: Despite numerous attempts—including moving the target folder to default Documents library, resetting the Exclude list, and performing a "hard reset" by deleting all local AppData configuration files—the tool consistently refused to scan and back up the target data.

Finding: The modern, consumer-facing "File History" tool was found to be non-functional, buggy, and unreliable for backing up specific, user-selected data. This prompted a pivot to the older, more robust "Backup and Restore (Windows 7)" utility.

4. Findings, Part 2: Baseline Performance (Week 2)

The first test measured the performance of the initial full backup and a subsequent incremental backup (adding one new text file).
Measured the performance of an initial full backup and a subsequent incremental backup (adding one new text file).

Backup & Restore (Windows 11) – Full Backup:

Time: ~15 seconds | Size: 7.37 MB

rsync (Ubuntu) – Full Backup:

Time: ~0.5 seconds | Size: 31.0 MB

Backup & Restore (Windows 11) – Incremental Backup:

Time: ~24 seconds | Size: No change

rsync (Ubuntu) – Incremental Backup:

Time: ~0.5 seconds | Size: No change




Analysis: An "Apples to Oranges" Comparison

The results reveal a fundamental difference in philosophy between the tools:

Windows ("The Apple"): Prioritizes storage efficiency. It automatically compressed the 31MB dataset down to 7.37MB. This efficiency comes at the cost of high overhead, taking longer for an incremental check (24s) than the initial full backup (15s).

Linux ("The Orange"): Prioritizes speed and data parity. rsync is a 1:1 mirror, resulting in a larger backup (31MB) but with near-zero overhead. It was ~30x faster on the full backup and ~48x faster on the incremental backup.

5. Findings, Part 3: Recovery Scenarios (Week 3)

This phase simulated common data-loss scenarios.

Scenario 1: Accidental File Deletion

Test: One folder (/Photos/) and one file (/Database/....csv) were permanently deleted.

Tool

OS

Restore Time (RTO)

Usability

Verification (RPO)

Backup and Restore

Windows

~60 seconds

5/5 (Easy GUI)

Excellent (Match)

rsync

Ubuntu

~0.5 seconds

4/5 (Easy command)

Excellent (Match)

Finding: Both tools successfully restored the deleted files with perfect data integrity (checksums matched). rsync was dramatically faster.

Scenario 2: Data Corruption

Test: A single .png file was corrupted by forcing a save with a text editor, simulating "bit rot."

Tool

OS

Restore Time (RTO)

Usability

Verification (RPO)

Backup and Restore

Windows 11

~20 seconds*

5/5 (Easy GUI)*

CRITICAL FAILURE

rsync

Ubuntu

~0.5 seconds

5/5 (Easy command)

Excellent (Match)

Finding: This was the most critical finding of the project.

Linux rsync: rsync automatically detected that the corrupted file's data was different from the backup copy. It instantly overwrote the broken file with the clean file, "healing" the data. The final checksum matched the original.

Windows "Backup and Restore": The restore process critically and silently failed. A system-level file lock (likely OneDrive or a permission issue) caused an "Access is denied" error that blocked the restore. However, the tool did not report any error to the user. The restore looked successful, giving a false sense of security, but the file on disk remained the corrupted version.

6. Findings, Part 4: Cybersecurity Stress Test (Week 4)

This phase tested the backups against two common, modern attack vectors.

Scenario 1: Confidentiality (Stolen Backup)

Test: Assumed an attacker steals the external USB drive (Windows) or gains access to the BACKUP/ folder (Linux).

Finding: CRITICAL FAILURE for both. The Windows backup is a standard, unencrypted .zip file, and the rsync backup is an unencrypted 1:1 file mirror. An attacker would have immediate, full access to all backed-up data.

Scenario 2: Availability (Ransomware)

Test: Simulated a ransomware attack by deleting both the original project folder and the "online" backup (the plugged-in USB drive and the same-system BACKUP/ folder).

Finding: CATASTROPHIC FAILURE for both. With both the original and the only backup copy destroyed, data was 100% unrecoverable.

7. Conclusions & Recommendations

This project proved that while both tools can back up data, their real-world performance and security differ dramatically. The Linux rsync tool was superior in every metric except for storage efficiency. The Windows "Backup and Restore" tool was not only slow but suffered from a critical, silent failure, making it dangerously unreliable.

Based on these findings, the following recommendations are for any system administrator:

Prioritize Integrity over Efficiency: The silent failure of the Windows tool highlights that a reliable, verifiable restore is more important than backup size. rsync's ability to detect and heal corruption automatically makes it a more robust solution.

Beware of Integrated Systems: The Windows restore failure was likely caused by a conflict with OneDrive or another system-level process. This demonstrates a key risk in modern, tightly-integrated operating systems where file locks can prevent even administrative tools from functioning.

Assume Backups are Insecure (Confidentiality): Neither tool encrypts data by default. Any backup of sensitive information must be paired with full-disk encryption, such as BitLocker (Windows) or LUKS (Linux), to protect data-at-rest.

The 3-2-1 Rule is Not Optional (Availability): This project provided a practical, definitive demonstration of the "1" in the 3-2-1 Backup Rule (3 copies, 2 media, 1 offline). An "online" backup (a plugged-in USB drive or a same-system folder) is not a backup; it is a target. For true resilience against ransomware, a backup must be "offline" (ejected and stored in a drawer) or "immutable" (stored in a cloud repository with versioning/deletion protection).
