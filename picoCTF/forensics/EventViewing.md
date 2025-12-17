# Windows Malware Shutdown Persistence - Forensics CTF Write-Up

**Category:** Forensics
**Difficulty:** Medium
**Platform:** PicoCTF
**Artifacts provided:** Windows EVTX Logs

---

# Challenge Overview

An employee's Windows machine became unstable after installing and running software downloaded from the internet. Following execution, the system began shutting down immediately after each user login.

The objective was to analyze Windows event logs to identify evidence for each stage of the incident and recover a three-part flag hidden across the relevant logs.

---

## Investigation Summary

The investigation focused on correlating log artifacts with the following events:

1. Software installation
2. Software execution and persistence
3. Forced system shutdown on login

Windows EVTX logs were converted to JSONL format and analyzed using command-line tools.

---

## Tools Used

- `evtx_dump`
- `grep (-i)`
- macOS Terminal

---

## Analysis & Findings

---

## Software Installation

To identify evidence of software installation, the Application logs were examined for Windows Installer activity.

Installer-related events were located by filtering for the `MsiInstaller` provider:

```bash
grep -i "MsiInstaller" Windows_Logs.jsonl
```

These events confirmed that a software package had been successfully installed shortly before the system began exhibiting abnormal behavior. One of the installer log entries contained the first fragment of the flag: "cGljb0NURntFdjNudF92aTN3djNyXw==." This was converted from Base64 to reveal: "picoCTF{Ev3nt_vi3wv3r_."

The malware entered the system via a legitimate-looking installer downloaded from the internet.

---

## Software Execution & Persistence

Although the employee reported that running the software appeared to do nothing, security logs revealed that the program made persistent system changes.

Registry modification events were identified using Event ID 4657, which corresponds to registry value changes: `grep '"EventID:4657' Windows_Logs.jsonl`
This event showed that a new registry value was added to the following key: "HKLM\Software\Microsoft\Windows\CurrentVersion\Run"
The registry value pointed to the executable: "C:\Program Files (x86)\Totally_Legit_Software\custom_shutdown.exe"
This indicates that the malware configured itself to run automatically every time a user logged in.

The registry value name contained a Base64 encoded string ("MXNfYV9wcjN0dHlfdXMzZnVsXw="), which decoded to reveal the second fragment of the flag: "1s_a_pr3tty_us3ful_."

The software executed successfully and established persistence by creating a malicious autorun registry entry.

---

## Forced System Shutdown on Login

To determine the cause of the immediate shutdowns, the logs were searched for shutdown-related activity: `grep -i "shutdown" Windows_Logs.jsonl`

These results revealed execution of "shutdown.exe" shortly after user login events. The shutdown activity directly correlated with the persistent executable identified in the registry autorun key.

One of the shutdown-related log entries contained the third and final fragment of the flag: "dDAwbF84MWJhM2ZlOX0=." This was decoded to reveal: "t00l_81ba3fe9}."

The malware forced system shutdowns by executing shutdown.exe on login through a registry-based persistence mechanism.

---

## Flag

picoCTF{Ev3nt_vi3wv3r_1s_a_pr3tty_us3ful_t00l_81ba3fe9}

---

## Key Takeaways

- Malware may appear inactive while silently establishing persistence
- Registry Run keys are a common and effective persistence mechanism
- Effective forensic analysis requires correlating multiple log sources

---

## Files included

- `README.md` - Full forensic analysis and challenge write-up
- `Windows_Logs.jsonl` - EVTX logs converted to JSONL format for analysis
