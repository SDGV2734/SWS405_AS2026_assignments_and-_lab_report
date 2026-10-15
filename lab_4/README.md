# Lab 4

# Lab 04 - The Sleuth Kit Report

## 1. Introduction

This lab shows that files deleted from an NTFS volume can still be found and recovered from a disk image with The Sleuth Kit (TSK), and that each recovered file can be proven identical to the original by a SHA-256 hash match.

**Scenario.** A USB drive was seized from a Finance employee suspected of leaking "Project Falcon" data. Before seizure, the suspect deleted a report, a photo, a contract, a password list and other files. The task is to analyse the drive image, find the deleted files, examine their metadata, recover them and prove the recovered files are unaltered.

**Objectives.**

1. Install and verify TSK on Kali Linux.
2. Explain the TSK layer model: image → partition → file system → file name → metadata → content.
3. Use `img_stat`, `mmls`, `fsstat`, `fls`, `istat`, `icat`, `ffind` and `tsk_recover`.
4. Find the partition offset and pass it with `o` to every file-system command.
5. Recover deleted files and verify them with SHA-256.
6. Build a file-activity timeline with `fls -m` and `mactime`.

## 2. Environment and Method

The work was done on Kali Linux in a virtual machine, with TSK 4.14.0 with the command `mmls -V`. The lab ran in two roles, with one rule separating them.

| Role | Parts | What was done |
| --- | --- | --- |
| Suspect | 2–4 | Built a 200 MiB NTFS image, planted 9 files, deleted 8 of them |
| Investigator | 5–7 | Read the sealed image with TSK only; never mounted or modified it |

**Evidence handling.** The image was hashed after the files were deleted and then made read-only with `chmod 444`. The hash was re-checked before analysis and again at the end. Every investigator command read the image directly, with no mounting and no `sudo`.

**Screenshot S1:** `mmls -V` and `which …` output showing TSK installed.

## 3. Building the Evidence Image

The suspect's drive was simulated with an ordinary file, then prepared and filled like a real disk.

**Step 3.1 – Create the blank image.** `dd if=/dev/zero of=evidence.dd bs=1M count=200 status=progress` created a 209,715,200-byte file of zeros.

![image.png](Lab%204/image.png)

*Figure S2: Blank 200 MiB image created.*

**Step 3.2 – Partition table.** `parted` wrote an MBR table and one NTFS-type partition starting at sector 2048 (1 MiB ÷ 512 bytes).

![image.png](Lab%204/image%201.png)

*Figure S3: Partition table showing the partition starting at sector 2048.*

**Step 3.3 – Loop device, NTFS format and mount.** `losetup -fP --show` attached the image and created the partition node `${LOOP}p1`. `parted` only defines the partition, so `mkfs.ntfs -Q -L EVIDENCE` created the NTFS file system. It was then mounted at `/mnt/evidence` with `ntfs-3g`.

**[ INSERT SCREENSHOT S4 HERE ]** Commands: `echo $LOOP`, `ls ${LOOP}*`, `sudo mkfs.ntfs …`, `df -h /mnt/evidence`

*Figure S4: Loop device, formatted NTFS partition and mount.*

**Step 3.4 – Plant the files.** `make_files.py` created 9 files (text, CSV, PDF, PNG, DOCX, HTML, JSON, ZIP) plus `readme.txt`, the control file that is not deleted. `file` confirmed each type by content.

![image.png](Lab%204/image%202.png)

*Figure S5: Nine sample files and their detected types.*

**Step 3.5 – Record the original hashes.** SHA-256 hashes of the 8 files to be deleted were saved to `original_hashes.txt`, outside the image.

![image.png](Lab%204/image%203.png)

*Figure S6: Original hashes (8 lines).*

**Step 3.6 – Delete the files.** The 8 files and the `Confidential` folder were removed with `rm`, leaving only `readme.txt`.

![image.png](Lab%204/image%204.png)

*Figure S7: Only readme.txt remains.*

**Step 3.7 – Seal the image.** The image was unmounted, the loop device detached, the image hashed to `evidence.dd.sha256` and made read-only.

![image.png](Lab%204/image%205.png)

*Figure S8: No loop device attached, image hash recorded, permissions -r--r--r--.*

**Image SHA-256 (before analysis):** [`48b95932eae6a3d01ae6fb419a183432b0196ba36c0a8d801b2a9168089d52fa`]

Deleting a file on NTFS only marks its MFT entry and clusters as free. The bytes stay on disk until something overwrites them, which is why the later recovery works.

## 4. Investigation Findings

The image hash was verified first (`evidence.dd: OK`, **S9**). TSK then answered one question per step, each handing a value to the next: image → partition offset → file system → deleted names → MFT address → content.

![image.png](Lab%204/image%206.png)

*Figure S9: Image hash verified (OK) before analysis.*

| Item | Finding | Command | Screenshot |
| --- | --- | --- | --- |
| Image type and size | raw, 209,715,200 bytes, 512-byte sectors (409,600 sectors) | `img_stat` | S10 |
| Partition start sector | 2048 (= 1,048,576 bytes) | `mmls` | S11 |
| Sectors 0–2047 | Unallocated alignment gap before the partition | `mmls` | S11 |
| File system, volume name | NTFS, EVIDENCE | `fsstat` | S12 |
| Cluster size, MFT entry size | 4,096 bytes, 1,024 bytes | `fsstat` | S12 |
| Deleted files found | 8 files plus the deleted `Confidential` folder | `fls -r -d -p` | S14 |
| MFT entry of report.pdf | [your number, expected 67] | `fls` | S13 |
| report.pdf created time (UTC) | [from `istat`, "Created:" line] | `istat` | S15 |
| report.pdf data cluster | [last line of `istat`, expected 25984] | `istat` | S15 |

![image.png](Lab%204/image%207.png)

*Figure S10: Image type, size and sector size.*

![image.png](Lab%204/image%208.png)

*Figure S11: Partition layout with the NTFS partition starting at sector 2048.*

![image.png](Lab%204/image%209.png)

*Figure S12: File system type, volume name, cluster size and MFT entry size.*

**Why `-o 2048` is needed.** Sector 0 holds the partition table, not a file system, so `fsstat evidence.dd` fails with "Cannot determine file system type". The NTFS file system starts at the offset `mmls` reports.

**Deleted files and their MFT entries** (use your own numbers; expected values shown):

| MFT entry | File |
| --- | --- |
| 65 | secret.txt |
| 66 | Confidential/passwords.csv |
| 67 | report.pdf |
| 68 | photo.png |
| 69 | Confidential/contract.docx |
| 70 | notes.html |
| 71 | Confidential/contacts.json |
| 72 | backup.zip |

![image.png](Lab%204/image%2010.png)

*Figure S13: Root directory listing; deleted entries are marked with * and readme.txt is not.*

![image.png](Lab%204/image%2011.png)

*Figure S14: All deleted files with their MFT entry numbers.*

In `fls` output, `*` marks a deleted entry and `-/r` means the directory-entry type is unknown while the MFT type is a regular file. `readme.txt` has no `*`, so it is still allocated. The `OrphanFile-16` to `OrphanFile-23` lines are unused reserved entries on a new NTFS volume, not evidence.

**Resident vs non-resident data.** `istat` on `secret.txt` (72 bytes) shows `$DATA … Resident`, so its content sits inside the MFT entry (**S16**). `istat` on `report.pdf` (4,030 bytes) shows `Non-Resident` with a cluster number, so its content sits in separate clusters (**S15**).

![image.png](Lab%204/image%2012.png)

*Figure S15: report.pdf metadata: Not Allocated, MACB times, Non-Resident, data cluster.*

![image.png](Lab%204/image%2013.png)

*Figure S16: secret.txt is Resident (content stored inside the MFT entry).*

## 5. Recovery and Verification

All 8 deleted files were recovered with `icat -r`, and all 8 SHA-256 hashes matched the originals. [Confirm this against your own output before submitting.]

`icat` takes only the MFT entry number and writes the content it points to; the `-r` option makes it follow the data of an unallocated (deleted) entry. File names were chosen by redirecting output (`> recovered/report.pdf`); the names themselves came from `fls`.

**Step 5.1 – Preview.** `icat -r -o 2048 evidence.dd 65` displayed the text of the deleted `secret.txt` on screen.

![image.png](Lab%204/image%2014.png)

*Figure S17: Deleted secret.txt content readable.*

**Step 5.2 – Raw cluster (optional).** `blkcat` on report.pdf's cluster showed the `%PDF-1.4` signature, so the bytes were still on disk.

![image.png](Lab%204/image%2015.png)

*Figure: Raw cluster starting with %PDF-1.4.*

**Step 5.3 – Recover all 8 files and check their types.** One `icat -r` per MFT entry wrote the files to `recovered/`. Sizes matched the Part 3 listing, and `file recovered/*` returned the same types as before deletion.

![image.png](Lab%204/image%2016.png)

*Figure S18: Eight recovered files with matching sizes and types.*

![image.png](Lab%204/image%2017.png)

*Figure: A recovered file opened normally.*

**Step 5.4 – Hash check.** `sha256sum -c` against `original_hashes.txt` printed 8 × OK.

![image.png](Lab%204/image%2018.png)

*Figure S19: All 8 recovered files match their original hashes.*

**Step 5.5 – Bulk recovery.** `tsk_recover -o 2048 evidence.dd tsk_recovered` reported `Files Recovered: 8` and rebuilt the `Confidential` folder tree; the hash check against `original_hashes.txt` again gave 8 × OK.

![image.png](Lab%204/image%2019.png)

![image.png](Lab%204/image%2020.png)

*Figure S20: tsk_recover output and 8 × OK.*

`tsk_recover` exports only unallocated files by default, so `readme.txt` was not included.

## 6. Timeline Analysis

All files were created within seconds of each other, and the last event on the `Confidential` folder marks when its contents were deleted. [Fill the times from your own `timeline.csv`.]

`fls -r -m / -o 2048 evidence.dd > bodyfile.txt` wrote a body file of every timestamp, and `mactime -b bodyfile.txt -d -z UTC` sorted it into `timeline.csv`, with NTFS system files filtered out. **(S21)**

![image.png](Lab%204/image%2021.png)

*Figure S21: Timeline showing creation times and the final m.c. line for Confidential.*

## 7. Integrity Verification

The final `sha256sum -c evidence.dd.sha256` returned `evidence.dd: OK`, so no analysis command changed the evidence. **(S22)** [Confirm from your output.]

![image.png](Lab%204/image%2022.png)

*Figure S22: Image hash still OK after analysis.*

| Check | Result |
| --- | --- |
| Image SHA-256 before analysis | [hash from S8] |
| Image SHA-256 after analysis | [matches? OK] |
| Image permissions | `-r--r--r--` (read-only) |

The hash matches the recorded reference, which shows the image was not altered during analysis. It does not by itself prove the image was acquired correctly in the first place. The read-only permission is a software safeguard; real laboratories also use hardware write-blockers.

## 8. Review Questions

1. **Why does `fsstat evidence.dd` fail without `o`, and where does 2048 come from?** Sector 0 holds the partition table, not a file system, so TSK cannot detect one there. The NTFS partition starts at sector 2048 (1 MiB ÷ 512 bytes), which `mmls` reports and `parted` was told to use in Part 2.
2. **What do  and `/r` mean in `/r * 67-128-2: report.pdf`?**  means the entry is deleted. `/r` means the directory entry's type is unknown () and the MFT type is a regular file (`r`).
3. **What is the difference between resident and non-resident files?** A resident file's content is stored inside its 1,024-byte MFT entry; a non-resident file's content is stored in separate clusters. `istat` showed `Resident` for the 72-byte `secret.txt` and `Non-Resident` with a cluster number for `report.pdf`.
4. **How did `icat` recover files without their names?** It needs only the MFT entry number. The entry holds the file's data location, and the name is a separate label stored in the directory entry.
5. **Why compare SHA-256 hashes instead of just opening the files?** A file can open normally and still be partly overwritten or altered. A matching hash proves the recovered file is bit-for-bit identical to the original.
6. **Give two reasons a deleted file might not be recoverable on a real drive.** New files can overwrite the freed clusters, new files can reuse the MFT entry (losing the name and data location), and SSD TRIM can erase freed blocks.
7. **What does the final `sha256sum -c` prove, and why does it matter in court?** It proves the image is unchanged from the acquisition hash, so analysis did not alter the evidence. This supports its integrity and chain of custody.

## 9. Conclusion and Limitations

TSK identified all 8 deleted files and the deleted folder from the NTFS metadata, and `icat` and `tsk_recover` recovered them with matching SHA-256 hashes, while the image hash stayed unchanged. [Adjust if any of your results differ.]

Recovery was complete because nothing was written to the volume after deletion. On a drive in use, overwritten clusters, a reused MFT entry or SSD TRIM can make recovery partial or impossible, and a failed hash would then be reported as partly overwritten content. This lab used a small, simulated 200 MiB drive and fictional data, so it shows the method rather than real-world recovery rates.

The key lessons are to find the partition offset first, to read evidence only through read-only tools, and to prove every claim with hashes.