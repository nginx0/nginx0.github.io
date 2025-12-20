---
title: "TryHackMe: Advent Of Cyber 2025 Day 9 (Passwords - A Cracking Christmas)"
categories: [TryHackMe]
tags: [hashcat, john, advent of cyber]
render_with_liquid: false
media_subpath: /images/tryhackme_aoc2025_day9/
image:
  path: banner.png
---

Learn how to crack password-based encrypted files.

<h2 style="color:#A64AC9; text-align:center; font-weight:700; font-size:1.8em; text-shadow: 1px 0 #A64AC9, -1px 0 #A64AC9, 0 1px #A64AC9, 0 -1px #A64AC9;">
  The Story
</h2>

[![TryHackMe Room Link](header.svg){: width="1200" height="407" }](https://tryhackme.com/room/attacks-on-ecrypted-files-aoc2025-asdfghj123)

## Introduction

With time between Easter and Christmas being destabilised, the once-quiet systems of *The Best Festival Company* began showing traces of encrypted data buried deep within their servers. Sir Carrotbane, stumbled upon a series of locked PDF and ZIP files labelled *“North Pole Asset List.”* Rumours spread that these could contain fragments of **Santa’s master gift registry**, critical information that could help Malhare control the festive balance between both worlds.

Sir Carrotbane sets out to crack the encryption, learning how weak passwords can expose even the most guarded secrets. Can the Elves adapt fast and prevent their secrets from being discovered?

## How Attackers Recover Weak Passwords

Attackers don't usually try to "break" the encryption itself because that would take far too long with modern cryptography. Instead, they focus on guessing the password that protects the file. The two most common ways of doing this are dictionary attacks and brute-force (or mask) attacks.

**Dictionary Attacks**

In a dictionary attack, the attacker uses a predefined list of potential passwords, known as a wordlist, and tests each one until the correct password is found. These wordlists often contain leaked passwords from previous breaches, common substitutions like **password123**, predictable combinations of names and dates, and other patterns that people frequently use. Because many users choose weak or common passwords, dictionary attacks are usually fast and highly effective.

**Mask Attacks**

Brute-force and mask attacks go one step further. A brute-force attack systematically tries every possible combination of characters until it finds the right one. While this guarantees success eventually, the time it takes grows exponentially with the length and complexity of the password.

![](split.png){: width="1920" height="1080"}

Mask attacks aim to reduce that time by limiting guesses to a specific format. For example, trying all combinations of three lowercase letters followed by two digits.

By narrowing the search space, mask attacks strike a balance between speed and thoroughness, especially when the attacker has some idea of how the password might be structured.

Practical tips attackers use (and defenders should know about):

- Start with a wordlist (fast wins). Common lists: `rockyou.txt`, `common-passwords.txt`.
- If the wordlist fails, move to targeted wordlists (company names, project names, or data from the target).
- If that fails, try mask or incremental attacks on short passwords (e.g. `?l?l?l?d?d` = three lowercase letters + two digits, which is used as a password mask format by password cracking tools).
- Use GPU-accelerated cracking when possible; it dramatically speeds up attacks for some algorithms.
- Keep an eye on resource use: cracking is CPU/GPU intensive. That behaviour can be detected on a monitored endpoint.

## Exercise

You will find the files for this section in the `Desktop` directory of the machine. Switch to it by running `cd Desktop` in your terminal.

**1. Confirm the File Type**

Use the `file` command or open the file with a hex viewer. This helps pick the right tool.

```console
ubuntu@tryhackme:~/Desktop$ file flag.pdf
ubuntu@tryhackme:~/Desktop$ file flag.zip
```

If it's a PDF, proceed with PDF tools. If it's a ZIP, proceed with ZIP tools.

**2. Tools to Use (pick one based on file type)**

- PDF: `pdfcrack`, `john` (via pdf2john)
- ZIP: `fcrackzip`, `john` (via zip2john)
- General: `john` (very flexible) and `hashcat` (GPU acceleration, more advanced)

**3. Try a Dictionary Attack First (fast, often successful)**

Example: PDF with `pdfcrack` and `rockyou.txt`:

```console
ubuntu@tryhackme:~/Desktop$ pdfcrack -f flag.pdf -w /usr/share/wordlists/rockyou.txt 
PDF version 1.7
Security Handler: Standard
V: 2
R: 3
P: -1060
Length: 128
Encrypted Metadata: True
FileID: 3792b9a3671ef54bbfef57c6fe61ce5d
U: c46529c06b0ee2bab7338e9448d37c3200000000000000000000000000000000
O: 95d0ad7c11b1e7b3804b18a082dda96b4670584d0044ded849950243a8a367ff
Average Speed: 29538.5 w/s. Current Word: 's8t8a8r8'
Average Speed: 31068.0 w/s. Current Word: 'toby344'
Average Speed: 30443.7 w/s. Current Word: 'erikowa'
Average Speed: 29519.3 w/s. Current Word: '0848845800'
Average Speed: 29676.4 w/s. Current Word: 'u3904041'
Average Speed: 29339.8 w/s. Current Word: 'spider0187'
Average Speed: 30156.8 w/s. Current Word: 'roermond7'
Average Speed: 30364.8 w/s. Current Word: 'papaopass2'
found user-password: 'XXXXXXXXXX'
```

## Detection of Indicators and Telemetry

Offline cracking does not hit login services, so lockouts and failed logon dashboards stay quiet. We can detect the work where it runs, on endpoints and jump boxes. The important signals to monitor include:

**Process creation:** Password cracking has a small set of well-known binaries and command patterns that we can look out for. A mix of process events, file activity, GPU signals, and network touches tied to tooling and wordlists. Our goal is to make the activity obvious without drowning in noise.

- Binaries and aliases: `john`, `hashcat`, `fcrackzip`, `pdfcrack`, `zip2john`, `pdf2john.pl`, `7z`, `qpdf`, `unzip`, `7za`, `perl` invoking `pdf2john.pl`.
- Command‑line traits: `--wordlist`, `-w`, `--rules`, `--mask`, `-a 3`, `-m` in Hashcat, references to `rockyou.txt`, `SecLists`, `zip2john`, `pdf2john`.
- Potfiles and state: `~/.john/john.pot`, `.hashcat/hashcat.potfile`, `john.rec`.

It's worth noting that on Windows systems, Sysmon Event ID 1 captures process creation with full command line properties, while on Linux, `auditd`, `execve`, or EDR sensors capture binaries and arguments.

**GPU and Resource Artefacts**

GPU cracking is loud. Sudden high utilisation on hosts can be picked up and would need to be investigated.

- `nvidia-smi` shows long-running processes named `hashcat` or `john`.  
- High, steady GPU utilization and power draw while the fan speed spikes.  
- Libraries loaded include: `nvcuda.dll`, `OpenCL.dll`, `libcuda.so`, `amdocl64.dll`.

![](soc.jpg){: width="1920" height="1080"}

**Network Hints, Light but Useful**

Offline cracking does not need the network once wordlists are present. Yet most operators fetch lists and tools first.

- Downloads of large text files such as `rockyou.txt`, or Git clones of common wordlist repositories.  
- Package installations (e.g., `apt install john hashcat`) detected through EDR or package telemetry.  
- Tool updates and driver downloads for GPU runtimes.

**Unusual File Reads**

Repeated reads of files such as wordlists or encrypted files would need analysis.

**Detections**

Below are some examples of detection rules and hunting queries we can put to use across various environments.

*Sysmon:*

```console
(ProcessName="C:\Program Files\john\john.exe" OR
 ProcessName="C:\Tools\hashcat\hashcat.exe" OR
 CommandLine="*pdf2john.pl*" OR
 CommandLine="*zip2john*")
```

*Linux audit rules, temporary for an investigation:*

```console
auditctl -w /usr/share/wordlists/rockyou.txt -p r -k wordlists_read
auditctl -a always,exit -F arch=b64 -S execve -F exe=/usr/bin/john -k crack_exec
auditctl -a always,exit -F arch=b64 -S execve -F exe=/usr/bin/hashcat -k crack_exec
```

*Sigma style rule, Windows process create for cracking tools:*

```console
title: Password Cracking Tools Execution
id: 9f2f4d3e-4c16-4b0a-bb3a-7b1c6c001234
status: experimental
logsource:
  product: windows
  category: process_creation
detection:
  selection_name:
    Image|endswith:
      - '\john.exe'
      - '\hashcat.exe'
      - '\fcrackzip.exe'
      - '\pdfcrack.exe'
      - '\7z.exe'
      - '\qpdf.exe'
  selection_cmd:
    CommandLine|contains:
      - '--wordlist'
      - 'rockyou.txt'
      - 'zip2john'
      - 'pdf2john'
      - '--mask'
      - ' -a 3'
  condition: selection_name or selection_cmd
level: medium
```

**Response Playbook**

As security analysts in Wareville, it is important to have a playbook to follow when such incidents occur. The immediate actions to take are:

- Isolate the host if malicious activity is detected. If it is a lab, tag and suppress.
- Capture triage artefacts such as process list, process memory dump, `nvidia-smi` sample output, open files, and the encrypted file.
- Preserve the working directory, wordlists, hash files, and shell history.
- Review which files were decrypted. Search for follow‑on access, lateral movement or exfiltration.
- Identify the origin and intent of the activity. Was this authorised? If not, escalate to the IR team.
- Remediate the activity, rotate affected keys and passwords, and enforce MFA for accounts.
- Close with education and correct placement of tools into approved sandboxes.

**PDF cracking**

```console
pdfcrack -f flag.pdf -w /usr/share/wordlists/rockyou.txt
```

**ZIP cracking**

```console
zip2john flag.zip > ziphash.txt
```

```console
john --wordlist=/usr/share/wordlists/rockyou.txt ziphash.txt
```

## Answer

### Question 1

What is the flag inside the encrypted PDF?

<details>
  <summary style="cursor:pointer; padding:10px; border:1px solid #888; background-color:#444; color:#fff; user-select: none;">
    Answer
  </summary>
  <div style="padding:10px; border:1px solid #888; background-color:#333; color:#fff;">
    <span onclick="navigator.clipboard.writeText('THM{Cr4ck1ng_PDFs_1s_34$y}')" style="cursor:pointer;">THM{Cr4ck1ng_PDFs_1s_34$y}</span>
    <i onclick="navigator.clipboard.writeText('THM{Cr4ck1ng_PDFs_1s_34$y}')" style="float:right; cursor:pointer; font-size:16px;">&#x1F4C4;</i>
  </div>
</details>

### Question 2

What is the flag inside the encrypted zip file?

<details>
  <summary style="cursor:pointer; padding:10px; border:1px solid #888; background-color:#444; color:#fff; user-select: none;">
    Answer
  </summary>
  <div style="padding:10px; border:1px solid #888; background-color:#333; color:#fff;">
    <span onclick="navigator.clipboard.writeText('THM{Cr4ck1n6_z1p$_1s_34$yyyy}')" style="cursor:pointer;">THM{Cr4ck1n6_z1p$_1s_34$yyyy}</span>
    <i onclick="navigator.clipboard.writeText('THM{Cr4ck1n6_z1p$_1s_34$yyyy}')" style="float:right; cursor:pointer; font-size:16px;">&#x1F4C4;</i>
  </div>
</details>