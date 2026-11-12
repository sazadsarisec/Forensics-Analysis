# Forensics Lab Report: Forensic Analysis of Linux System

**Course:** IT411
**Student:** Sol Azadsari
**Date:** 11/12/2026
**Instructor:** Basta

---

## 1. Objective


> Learn how to search through forensic disk image in dd format to find artifacts for intrusion on Linux Server. 
> 	Create image of compromised Linux machine w/ FTK Imager
> 	Examine compromised Linux machine w/ Autopsy
> 	Analyze common locations of compromised artifacts

---

## 2. Scenario / Background

Summarize the scenario or context provided for the lab (if applicable).

- **Case Reference:** IT411 M5 P1
- **Evidence Type:** Disk
- **Tools Used:** FTK Imager, Autopsy, auth.log, history file, /etc/shadow, /etc/passwd

---

## 3. Materials & Tools

List all hardware, software, images, and evidence files used.

| Item             | Version / Details       |
| ---------------- | ----------------------- |
| Operating System | Windows 10              |
| Forensic Tool    | Autopsy 4.14            |
| Evidence Image   | Linux_Webserver04052026 |
| Hashing Utility  | N/A                     |

---

## 4. Methodology / Procedure


### 4.1 Collection
- [ ] Identify and label data from relevant sources
- [ ] Record and acquire data
- [ ] Digital forensic imaging - no modification of contents occurs
- [ ] Document each step and stage data was found and transferred or copied to with legal name, timestamp, data label, and reasoning for purchase
*FTK Imager was used for collection

### 4.2 Examination
1. Forensically sound data collection must be used within reason
2. Preserve data integrity
3. Document each step to preserve chain of custody
*Autopsy was used for examination

### 4.3 Analysis
- Results of investigation using legally justifiable methods and techniques
*Autopsy was used for Analysis

### 4.4 Reporting
1. Document the examination and analysis of investigation
2. Explain tools, describe analysis, issues, and vulnerabilities identified
3. Recommendations
4. Document each step to preserve chain of custody


### Methods

Digital forensic imaging - copies physical storage without modification
Bit-by-bit replica of original media
Hashing ensures the image is an exact copy and forensically equivalent

#### Windows Copy / Paste
does not allow exact copies, no deleted files and folders
#### Disk Cloning
Exact copy of target drive with operating system and files - exact copy in case of main drive failure but not used for forensic copies (no built-in hashing)
#### Disk Imaging
stored as a single file unlike cloned disk, has all info on disk as well as free space and slack space - this image is analyzed with a forensic software like EnCase or Autopsy
#### FTK Imager
GUI program that allows user to create or convert disk image inside of Windows
	Windows can create issues imaging a disk due to files locked by OS
	FTK Imager will allow to image a disk or logical drive
		- FTK Imager -> Import Device -> Physical (VM disk)
	- Expand drive -> R-click metasploitable-root partition -> export
- Raw dd format from partition -> Export to: Desktop image folder -> filename .dd extension
- Image fragment size 30000
- New topic tree for research: Fragmentation and Compression on disk images, filesystem types and sizes, why fragment, why compress, how does it affect disk imaging
	- File Allocation Table (FAT) and NTFS (new tech file system) are used in Windows
	- EXT (extend file system) 2, 3, 4 and ReiserFS are used in Linux

#### Autopsy forensic browser - free, open source
can be used to capture disk image and analyze, and create reports
	Create new case - Add Data Source - Sign as examiner

#### Linux log analysis
secure log file - logs security incidents
		located in auth.log
			created accounts, group accounts, logging from remote system, changing password

/var/log/:
	auth.log
	/apache2/access.log
	.bash_history

/etc/:
	shadow (password hashes)
	passwd (password file)


VirusTotal:
	website with info on malware and viruses
	upload files that are suspicious for analysis
	


---

## 5. Observations & Results

Record what you observed during the lab. Include screenshots, command output, and key findings.

### 5.1 Key Findings
#### /var/log/auth.log:
Excessive password failure for misc port outbound ssh2
	Accepted password outbound with ssh2
	new group creation, new user creation
	user id UID:0 , only Administrator should have 0
	group ID above 900 - as 1004, admin rights

#### /home/user:
.bash_history file
	attacker performed following actions:
		viewed password hashes
		viewed website hosted on machine
		echo command defacing website to index.php
		created a ssn directory
		created creditcardno directory
		echo command visa information to numbers.txt - available for viewing on website
		echo command mastercard info as well
		copied shadow file to . ('.' is current directory)
		enabled 777 rights (read, write, execute to user and group) on the shadow file
		wget command to IP/shell.elf
		chaned rights to 777 for the shell
		ran the shell
		

Extracted shell.elf and uploaded to VirusTotal - if not detected by VirusTotal but is known payload, send to reverse engineer for further analysis

#### VirusTotal:
Compact shellcode that acts as stager - establishes TCP connection to 192.168.1.101:443 using syscalls `socketcall` - reads payload from socket into a buffer and jumps to buffer for execution (classic connect-read-execute classic for reverse shell or first-stage downloader)

#### /var/log/apache2:
logs show enumeration attemps on website via nmap
GET /creditcardno/numbers.txt, GET /creditcardno/etc/shadow, and GET creditcardno/shadow was dir and file created by attacker while inside machine, now accessing via reverse shell through the .101 address 

#### /var/www/index.php:
website contents removed and replaced with `this site has been defaced`
directory creation trail from before leads us to the /creditcardno dir with numbers.txt and shadow.txt files on webserver 


### 5.2 Command Output

N/A

### 5.3 Screenshots

![[Screenshot_20261112_135627.png|443]]

---

## 6. Analysis & Interpretation

Explain the significance of your findings. What do they mean in the context of the case or scenario?

- **Interpretation:** 
Brute force password attacks against web server permitted attacker into elevated perms account. Attacker created admin account, defaced website, uploaded credit card numbers and all user passwords to website in hidden directory. Attacker installed reverse shell and leaked customer financial data.
- **Corroboration:** Above file destinations and files show indicators or compromise. Validation performed with thorough analysis of website alterations, leaked data, new directories, and connection history to attack machine.
- **Limitations:** No gaps at this time, web server exploited and altered.

---

## 7. Chain of Custody & Integrity

Document how evidence was handled to maintain forensic integrity.

| Action            | Time / Date | Hash (if applicable) | Notes                |
| ----------------- | ----------- | -------------------- | -------------------- |
| Received evidence | 11.12.2026  | `N/A`                | Disk Image VM        |
| Imaged evidence   | 11.12.2026  | N/A                  | Write-blocked        |
| Analysis started  | 11.12.2026  | —                    | Working copy created |

---

## 8. Conclusion

Lab objective was to learn how to use forensics tools and find indicators of compromise in a Linux environment. Compromise was successfully found and detailed in a Linux web server via reverse shell payload after brute force account password attempts. FTK Imager was used to pull image data from a VM disk and Autopsy was used to analyze disk image created in dd format. 

---

## 9. References

List any course materials, vendor documentation, or external resources consulted.

1. [chmod777: What it Means and Why You Should Never Use It](https://linuxize.com/post/what-does-chmod-777-mean/)
2. [VirusTotal](https://www.virustotal.com/)
---

## 10. Appendices (Optional)


### Appendix A: Full Command Log
### Appendix B: Additional Screenshots
### Appendix C: Evidence Information

---

*Template version 1.0 — Forensics Lab Report*
