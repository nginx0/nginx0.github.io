---
title: "TryHackMe: Advent Of Cyber 2025 Day 6 (Malware Analysis - Egg-xecutable)"
categories: [TryHackMe]
tags: [Analysis, reverse engineering, sandboxes, advent of cyber]
render_with_liquid: false
media_subpath: /images/tryhackme_aoc2025_day6/
image:
  path: banner.png
---

Discover some common tooling for malware analysis within a sandbox environment.

<h2 style="color:#A64AC9; text-align:center; font-weight:700; font-size:1.8em; text-shadow: 1px 0 #A64AC9, -1px 0 #A64AC9, 0 1px #A64AC9, 0 -1px #A64AC9;">
  The Story
</h2>

[![TryHackMe Room Link](header.svg){: width="1200" height="407" }](https://tryhackme.com/room/malware-sandbox-aoc2025-SD1zn4fZQt)

## Introduction

The town of Wareville remains quiet in the middle of the night. While the residents of Wareville are nicely tucked up in bed, blissfully unaware, the SOC team at *The Best Festival Company (TBFC)* remain alert, poised and ready for whatever may face them.

Monitoring their screens, armed with a freshly poured mug of hot cocoa, the elves of the SOC watch their dashboards diligently. 

Suddenly, the elves receive an email in unison from Elf McClause, Head of Elf Affairs, in their inboxes. It reads:

<div style="border: 1px solid #cccccc; padding: 15px; border-radius: 10px;">

<strong>From:</strong> Elf McClause<br>
<strong>Subject:</strong> A new schedule awaits<br>
<strong>Attachments:</strong> HopHelper.exe<br><br>

To all Elves, going forward, we will be using a new program for creating and viewing team rotas.<br><br>

This new program will revolutionalise how rotas are made here at The Best Festival Company.<br><br>

Please contact IT support if you have issues running the program.<br><br>

Kind regards, remain the best,<br><br>

Elf McClause<br>
Head of Elf Affairs

</div>
<br>

*"Why is Elf McClause working at 3AM?"* Screams a member of the SOC team in the background. They're right, something is amiss.

Elf McBlue is immediately suspicious. Their years of experience in the SOC have given them the wisdom not to download "out of the blue" executables. Without McSkidy's wisdom, Elf McBlue takes charge, loading up their malware investigation toolkik - the investigation begins.

![](credentials.png){: width="1194" height="161"}

## Principles of Malware Analysis

Malware analysis is the process of examining a malicious file to understand its functionality, operation, and methods for defence against it. By analysing a malicious file or application, we can see exactly how it operates, and therefore, know how to prevent it. For example, could the malicious file communicate with an attacker's server? We can block that server.

Could the malicious file leave traces on the machine? We can use these to determine if the malware has ever infected another device. Instead of fearing malware, we can take a proactive approach by translating technical findings into practical defensive measures and understanding how the malware fits into an attacker's techniques.

![](malware.png){: width="600" height="600"}

There are two main branches of malware analysis: **static** and **dynamic**. Static analysis focuses on inspecting a file without executing it, whereas dynamic analysis involves execution. We will come to these shortly.

**Sandboxes**

In cyber security, sandboxes are used to execute potentially dangerous code. Think of this as disposable digital play-pens. These sandboxes are safe, isolated environments where potentially malicious applications can perform their actions without risking sensitive data or impacting other systems.

The use of sandboxes is part of the **golden rule in malware analysis: never run dangerous applications on devices you care about**.

Most of the time, sandboxes present themselves as virtual machines. Virtual machines are a popular choice for sandboxing because you can control how the system operates and benefit from features such as snapshotting, which allows you to create and restore the machine to various stages of its status. 

![](sandbox.png){: width="600" height="600"}

To reiterate, it is **imperative** to understand that potentially malicious code and applications should **only be run in a safe, isolated environment**. From now on, this room will refer to malicious code and applications as samples.

With these fundamentals covered, let's move on to the practical section of today's room.**The following will demonstrate a sample; you must apply these techniques to the** `HopHelper.exe` file presented to you within the "HopHelper" folder on the Desktop of the practical VM.

## Interactive: Static Analysis

As we alluded to previously in this room, we use static analysis to gather information about a sample without executing it and digging deep.  

Static analysis can be a quick and effective way to understand how the sample may operate, as well as how it can be identified. Some of the information that can be gathered from static analysis has been included in the table below:

| **Information** | **Explanation** | **Example** |
|-----------------|-----------------|-------------|
| **Checksums** | These checksums are used within cyber security to track and catalogue files and executables. For example, you can Google the checksum to see if this has been identified before. | `a93f7e8c4d21b19f2e12f09a5c33e48a` |
| **Strings** | "Strings" are sequences of readable characters within an executable. This could be, for example, IP addresses, URLs, commands, or even passwords! | `138.62.51.186` |
| **Imports** | "Imports" are a list of libraries and functions that the application depends upon. For example, rather than building everything from scratch, applications will use operating system functions and libraries to interact with the OS.<br><br>These are useful, especially in Windows, as they allow you to see how the application interacts with the system.<br><br>`CreateFileW` — This library is used to create a file on a Windows system. | N/A |
| **Resources** | "Resources" contain data such as the icon that is displayed to the user. This is useful to examine, especially since malware might use a Word document icon to trick the user.<br><br>Additionally, malware itself has been known to hide in this section! | N/A |

However, it's important to note that regardless of how a sample may appear or function, we don't truly know until it's executed. Attackers use techniques such as obfuscation to obscure how the sample appears, primarily to evade anti-viruses but also to evade a curious analyst.

**Demonstrating PeStudio**

This section of the room will demonstrate using PeStudio on an example called `downloader.exe`. Please note that the information you see will be from this demonstration sample. The sample you will be analysing will be different, but the techniques will still apply.

Please note, it is imperative that you do not launch the `HopHelper.exe` executable yet.

<div style="border: 1px solid #cccccc; padding: 15px; border-radius: 10px;">

<strong>Executive Summary:</strong><br><br>

First, we will use <strong>PeStudio</strong> to gather information about the executable.<br><br>

1. Launch PeStudio<br>
2. Load the executable into PeStudio<br>
3. Click on the <strong>"indicators"</strong> tab in the dropdown<br>
4. Look for the <strong>SHA256Sum</strong>

</div>
<br>

First, we will launch PeStudio and load the executable into it. The shortcut for this has been placed on the Desktop of your analyst machine. You can drag and drop the executable into the PeStudio window, or load it by selecting `File -> Open File` from the toolbar. PeStudio will display some information about the executable.

![](pestudio.png){: width="1090" height="617"}

For us, at this stage, the `file > sha256` property within the table is of interest. This value is a checksum, which is a unique identifier for the executable. We can keep a note of this SHA256 as threat intelligence

<div style="border: 1px solid #cccccc; padding: 15px; border-radius: 10px;">

Now, proceed to do this on the sample, <strong>HopHelper.exe</strong>, provided to you on the analyst machine to answer question 1.<br><br>

As a reminder, the <strong>HopHelper.exe</strong> sample can be found in the <strong>"HopHelper"</strong> folder on the Desktop.

</div>
<br>

Next, we will proceed with reviewing the "Strings" of the executable. You can do this by clicking on the "strings" indicator on the left pane of PeStudio.

![](pestudio2.png){: width="1139" height="614"}

In the context of malware analysis, strings are sequences of readable characters present within an executable. This could be, for example, IP addresses, URLs, commands, or even passwords! As a malware analyst, it's great to have a look at these, as these could reveal the attacker's command infrastructure, which we can use for our defences.

<div style="border: 1px solid #cccccc; padding: 15px; border-radius: 10px;">

Now, proceed to review the strings on the sample, <strong>HopHelper.exe</strong>, provided to you on the analyst machine to answer question 2.<br><br>

Please note that it may take a few minutes for the <strong>"strings"</strong> to calculate.

</div>
<br>

Great! This concludes the static analysis portion of the practical. There's so much more to uncover using static analysis. Feel free to explore if you'd like. Let's proceed to the dynamic analysis below.

## Interactive: Dynamic Analysis

This section of the room provides a brief introduction to dynamic analysis. As you recall, dynamic analysis involves executing the malicious sample to identify its behaviours and how it interacts with the operating system.

**Regshot**

Regshot is a widely used utility, especially when analysing malware on Windows. It works by creating two "snapshots" of the registry—one before the malware is run and another afterwards. The results are then compared to identify any changes.

Malware aims to establish persistence, meaning it seeks to run as soon as the device is switched on. A common technique for malware is to add a `Run` key into the registry, which is frequently used to specify which applications are automatically executed when the device is powered on.

<div style="border: 1px solid #cccccc; padding: 15px; border-radius: 10px;">

Before executing the malicious sample, <strong>HopHelper.exe</strong>, let's create a snapshot of our registry as it currently is, so we can compare the difference once the sample has been executed.

</div>
<br>

Let's load up Regshot and create a capture of the registry as it currently exists. The shortcut to this has also been placed on the Desktop of the analyst machine.

First, change the output directory of the capture to the user's Desktop using the box with three dots in the "Output path" section.

Then, once set, let's create our first snapshot. Press **1st shot** and then **Shot** on the dropdown. Please note that this may take a few minutes to complete.

![](regshot.png){: width="286" height="280"}

<div style="border: 1px solid #cccccc; padding: 15px; border-radius: 10px;">

Now that we have taken a snapshot of the registry, you should proceed with executing the <strong>HopHelper.exe</strong> sample and take another snapshot. We will then compare the difference.<br><br>

For you, this is the <strong>HopHelper.exe</strong> executable located in the <strong>"HopHelper"</strong> folder on your Desktop.<br><br>

Psst... now that you have executed the sample, you might see that some strange things have happened.

</div>
<br>

Once we have executed our sample, let's return to Regshot and capture our second snapshot, using the same procedure as above. Click on the **2nd shot** button and press **Shot** in the dropdown. Regshot is now capturing the registry again, and outputting the differences to a file.

![](regshot2.png){: width="282" height="292"}

And now, after a few seconds, let's press the **Compare** button that appears.

![](regshot3.png){: width="282" height="289"}

We can search for the executable within the log that opens up. 

**ProcMon**

Next, we will explore using ProcMon (Process Monitor) from the Sysinternals suite to investigate today's sample. Proccess Monitor is used to monitor and investigate how processes are interacting with the Windows operating system. It is a powerful tool that allows us to see exactly what a process is doing. For example, reading and writing registry keys, searching for files, or creating network connections.

Open **Process Monitor (ProcMon)**, the shortcut for this has been placed on the Desktop of the analyst machine. Process Monitor will automatically start capturing events of various processes on the system.

![](procmon.png){: width="1298" height="980"}

<div style="border: 1px solid #cccccc; padding: 15px; border-radius: 10px;">

Now, execute the sample <strong>HopHelper.exe</strong> again and return to <strong>Process Monitor</strong> to see how it interacts with the operating system.<br><br>

You will see a lot of information here. Do not worry, we will come onto how to filter this shortly.

</div>
<br>

After allowing a minute to pass, ensuring the sample has fully executed, we will now stop capturing. To stop capturing more events, click on the **Play** button in the toolbar of Process Monitor.

![](procmon2.png){: width="589" height="248"}

As you can see, there is a lot of information to scroll through here, with the most recent events at the bottom. Here we can see how various system processes are interacting with Windows. Nearly all of it, we don't care about. 

<div style="border: 1px solid #cccccc; padding: 15px; border-radius: 10px;">

Remember, the task content is demonstrating using <strong>Process Monitor</strong> for the demonstration sample <strong>downloader.exe</strong>.<br><br>

You will need to follow along, but doing so for the <strong>HopHelper.exe</strong> sample.

</div>
<br>

Let's apply some filters. Afterall, for this demonstration, we only care about the `downloader.exe` sample. To do so, click on the **Filter** button, and then **Filter** within the dropdown.

![](filter.png){: width="394x269" height="269"}

A new window will open.

![](filter2.png){: width="583" height="357"}

Here we can create some filters to remove some of the noise that we don't care about. Because we want to only look at this `downloader.exe` sample for this demonstration, we can apply a filter like so:

1. Apply the Process Name filter  
2. Set the condition to **is**  
3. Put in the name of the process we wish to see within the text area  
4. Press the **Add** button to apply this filter  
5. Click **OK** to save

![](filter3.png){: width="581" height="361"}

Once done, returning to the main Process Monitor window, we can already see the filter has worked.

![](procmon3.png){: width="970" height="738"}

Now it is much easier to investigate how the process is interacting with the operating system. Here are some **Operations** that may be of interest to us:

- RegOpenKey
- CreateFile
- TCP Connect
- TCP Receive

However, as you can see, there is still a lot of information. We can further apply filters to look for specific things that we want to investigate, such as these aforementioned **Operations**.

To do so, return to the **Filter** heading and create the filter we want to apply. For example, we can filter by **Operations**. Let's do so below, filtering for any TCP **Operation**:

![](filter4.png){: width="777" height="473"}

We will now see all **Operations** that include **TCP**. Remember, you can remove the filters you've previously applied by pressing the filter in the **Filter** list, and pressing **Remove**:

![](filter5.png){: width="777" height="476"}

Or, alternatively, if you wish to start over, you can simply press the **Reset Filter** option when clicking on the **Filter** heading.

![](filter6.png){: width="562" height="357"}

Phew! Well done. That concludes the demonstration for today's room. Remember, you will need to apply what you have learnt here on the **HopHelper.exe** executable that has been placed in the **HopHelper** folder on the analyst Desktop, to answer the questions below.

## Answer

### Question 1

**Static analysis:** What is the SHA256Sum of the HopHelper.exe?

<details>
  <summary style="cursor:pointer; padding:10px; border:1px solid #888; background-color:#444; color:#fff; user-select: none;">
    Answer
  </summary>
  <div style="padding:10px; border:1px solid #888; background-color:#333; color:#fff;">
    <span onclick="navigator.clipboard.writeText('F29C270068F865EF4A747E2683BFA07667BF64E768B38FBB9A2750A3D879CA33')" style="cursor:pointer;">F29C270068F865EF4A747E2683BFA07667BF64E768B38FBB9A2750A3D879CA33</span>
    <i onclick="navigator.clipboard.writeText('F29C270068F865EF4A747E2683BFA07667BF64E768B38FBB9A2750A3D879CA33')" style="float:right; cursor:pointer; font-size:16px;">&#x1F4C4;</i>
  </div>
</details>

### Question 2

**Static analysis:** Within the strings of HopHelper.exe, a flag with the format THM{XXXXX} exists. What is that flag value?

Note, this can be found towards the bottom of the strings output.

<details>
  <summary style="cursor:pointer; padding:10px; border:1px solid #888; background-color:#444; color:#fff; user-select: none;">
    Answer
  </summary>
  <div style="padding:10px; border:1px solid #888; background-color:#333; color:#fff;">
    <span onclick="navigator.clipboard.writeText('THM{STRINGS_FOUND}')" style="cursor:pointer;">THM{STRINGS_FOUND}</span>
    <i onclick="navigator.clipboard.writeText('THM{STRINGS_FOUND}')" style="float:right; cursor:pointer; font-size:16px;">&#x1F4C4;</i>
  </div>
</details>

### Question 3

**Dynamic analysis:** What registry value has the HopHelper.exe modified for persistence?

Note: Provide the full path of the key that has been modified

<details>
  <summary style="cursor:pointer; padding:10px; border:1px solid #888; background-color:#444; color:#fff; user-select: none;">
    Answer
  </summary>
  <div style="padding:10px; border:1px solid #888; background-color:#333; color:#fff;">
    <span onclick="navigator.clipboard.writeText('HKU\S-1-5-21-1966530601-3185510712-10604624-1008\Software\Microsoft\Windows\CurrentVersion\Run\HopHelper')" style="cursor:pointer;">HKU\S-1-5-21-1966530601-3185510712-10604624-1008\Software\Microsoft\Windows\CurrentVersion\Run\HopHelper</span>
    <i onclick="navigator.clipboard.writeText('HKU\S-1-5-21-1966530601-3185510712-10604624-1008\Software\Microsoft\Windows\CurrentVersion\Run\HopHelper')" style="float:right; cursor:pointer; font-size:16px;">&#x1F4C4;</i>
  </div>
</details>

### Question 4

**Dynamic analysis:** Filter the output of ProcMon for "TCP" operations. What network protocol is HopHelper.exe using to communicate?

Make sure to have executed HopHelper.exe while ProcMon was open and capturing events.

<details>
  <summary style="cursor:pointer; padding:10px; border:1px solid #888; background-color:#444; color:#fff; user-select: none;">
    Answer
  </summary>
  <div style="padding:10px; border:1px solid #888; background-color:#333; color:#fff;">
    <span onclick="navigator.clipboard.writeText('http')" style="cursor:pointer;">http</span>
    <i onclick="navigator.clipboard.writeText('http')" style="float:right; cursor:pointer; font-size:16px;">&#x1F4C4;</i>
  </div>
</details>

### Bonus wallpaper

<details>
  <summary>Wallpaper</summary>

<p>
  <img src="wm1.png" width="600" height="600">
</p>
<p>
  <img src="wm2.png" width="600" height="600">
</p>

</details>
