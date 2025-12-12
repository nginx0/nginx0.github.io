---
title: "TryHackMe: Advent Of Cyber 2024 Day 21"
categories: [TryHackMe]
tags: [reverse engineering, debugger, advent of cyber]
render_with_liquid: false
media_subpath: /images/tryhackme_aoc2024_day21/
image:
  path: banner.png
---

*McSkidy’s alert dashboard lit up with an unusual alert. A file-sharing web application built by Glitch had triggered a security warning. Glitch had been working hard during this season's SOC-mas after the last scare with the Yule Log Exploit, but this new alert caused McSkidy to question his intentions.*

*McSkidy began to investigate. It seemed the source of the alert came from a binary file that made its way to the web app’s backend. It did not belong there and had some anomalous activity. The binary was compiled with .NET. This whole setup seemed quite unusual, and with Glitch working on the latest security updates, McSkidy was filled with suspicion.*

*As McSkidy continued her investigation, Glitch rushed into the room: “I swear I did not put it there! I was testing defences, but I wouldn’t go that far!*

*McSkidy reassured him, “This doesn’t look like your work. Let's get to the bottom of this. Put on your decompiling hat, and let’s see what we are dealing with.”*

![Tryhackme Room Link](bell.png){: width="1152" height="300" .shadow }
_<https://tryhackme.com/r/room/adventofcyber2024>_

## Learning Objectives

- Understanding the structure of a binary file 
- The difference between Disassembly vs Decompiling
- Familiarity with multi-stage binaries
- Practically reversing a  multi-stage binary

![](credentials.png){: width="439" height="149"}

## Introduction to Reverse Engineering

Reverse Engineering (RE) is the process of breaking something down to understand its function. In cyber security, reverse engineering is used to analyse how applications (binaries) function. This can be used to determine whether or not the application is malicious or if there are any security bugs present.

![](wannacry.png){: width="600" height="453"}

For example, cyber security analysts reverse engineer malicious applications distributed by attackers to understand if there are any attributable indicators to associate the binary with an attacker and any potential ways to defend against the malicious binary. A famous example of this is the [WannaCry](https://en.wikipedia.org/wiki/WannaCry_ransomware_attack) ransomware in May 2017. Security researcher [Marcus Hutchins](https://en.wikipedia.org/wiki/Marcus_Hutchins) reverse-engineered the ransomware application and discovered a specific function within the application where the malware wouldn't run if a particular domain were registered and available.

Marcus then registered this domain, stopping the global WannaCry attack. This is just one of many famous cases of reverse engineering being used in cyber security defence.

## Binaries

In computing, binaries are files compiled from source code. For example, you run a binary when launching an executable file on your computer. At one point in time, this application would've been programmed in a programming language such as C#. It is then compiled, and the compiler translates the code into machine instructions.

Binaries have a specific structure depending on the operating system they are designed to run. For example, Windows binaries follow the Portable Executable (PE) structure, whereas on Linux, binaries follow the Executable and Linkable Format (ELF). This is why, for example, you cannot run a **.exe** file on MacOS. With that said, all binaries will contain at least:

- **A code section**: This section contains the instructions that the CPU will execute
- **A data section**: This section contains information such as variables, resources (images, other data), etc
- **Import/Export tables**: These tables reference additional libraries used (imported) or exported by the binary. Binaries often rely on libraries to perform functions. For example, interacting with the Windows API to manipulate files

The binaries in today's task follow the PE structure. This structure will be explained throughout the task.

## Disassembly Vs. Decompiling

When reverse engineering binaries, you will employ two primary techniques. This task section will introduce you to disassembly and decompiling, explaining the key differences and their pros and cons.

Disassembling a binary shows the low-level machine instructions the binary will perform (you may know this as assembly). Because the output is translated machine instructions, you can see a detailed view of how the binary will interact with the system at what stage. Tools such as IDA, Ghidra, and GDB can do this.

![](debugger.png){: width="1588" height="1176"}

Decompiling, however, converts the binary into its high-level code, such as C++, C#, etc., making it easier to read. However, this translation can often lose information such as variable names. This method of reverse engineering a binary is useful if you want to get a high-level understanding of the application's flow.

![](ilspy.png){: width="1234" height="617"}

There are specific circumstances where you would choose one method over the other. For example, decompiling is sometimes a "best guess" based on the tooling you've used and does not provide the actual full source code.

A table outlining the key differences between the two has been provided below.

| **Aspect**      | **Disassembly**                                                                            | **Decompiling**                                                                         |
|------------------|-------------------------------------------------------------------------------------------|-----------------------------------------------------------------------------------------|
| **Readability**  | Requires knowing assembly and low-level knowledge of computing concepts.                  | Requires familiarity with programming and logic.                                        |
| **Level of output** | The translated output is the exact instructions that the machine will perform.            | The translated output is often a "best guess". The output may not be accurate, and useful information, such as variables, function names, etc., will likely be lost. |
| **Difficulty**   | The difficulty can be considered higher as the machine instructions are translated into assembly. | The machine instructions are translated into a high-level language, which makes them easier to understand if you are familiar with the language the binary is written in. |
| **Usefulness**   | The entire behavior of the binary can be studied given enough time.                        | Decompiling is a quick way to understand some of the logic of the binary.               |

## Multi-Stage Binaries

Recent trends in cyber security have seen the rise of attackers using what's known as "Multi-stage binaries" in their campaigns - especially malware. These attacks involve using multiple binaries responsible for different actions rather than one performing the entire attack. Usually, an attack involving various binaries will look like the following:

1. **Stage 1 - Dropper**: This binary is usually a lightweight, basic binary responsible for actions such as enumerating the operating system to see if the payload will work. Once certain conditions are verified, the binary will download the second - much more malicious - binary from the attacker's infrastructure.
2. **Stage 2 - Payload**: This binary is the "meat and bones" of the attack. For example, in the event of ransomware, this payload will encrypt and exfiltrate the data.
Sophisticated attackers may further split actions of the attack chain (e.g., lateral movement) into additional binaries. Using multiple stages helps evade detection and makes the analysis process more difficult.

For example, a small, more "harmless" initial binary is likelier to evade detection via email filtering than a fully-fledged binary that performs malicious actions such as encryption. Additionally, splitting these functions into multiple stages gives the attacker much more control (i.e. only downloading specific stages once conditions such as time have been met).

The diagram below shows what an attack involving multiple staged binaries may look like.

![](stage.png){: width="1140" height="800"}

## Jingle .NET all the way

For today's task, you will be reverse engineering two .NET binaries using the decompiler ILSpy. You can follow the walkthrough below in reverse engineering using an example application named **demo.exe**. Then, you will reverse an application on your own at the end of this task.

Before analysing our target, we need to learn and find a way to identify the original binary file, modify it, or use it as evidence. Also, it is good practice to have a big picture of the file we are dealing with so that we can choose the proper tools we will need.

Let's start by navigating to the file location in the **demo** folder on the machine's Desktop by right-clicking on the file named **demo** and clicking on **Properties**.

![](properties.png){: width="641" height="519"}

We can observe that the file's extension is .exe, indicating that it is a Windows executable.

![](typeoffile.png){: width="538" height="521"}

Since it's a Windows file, we'll use [PEStudio](https://www.winitor.com/download), a software designed to investigate potentially malicious files and extract information from them without execution. This will help us focus on the static analysis side of the investigation. Let's open PEstudio from the taskbar and then click on  **File > Open** and select the file **demo.exe** located in **C:\Users\Administrator\Desktop\demo\demo.exe** as shown below.

![](pestudio.png){: width="1005" height="467"}

As shown below, PEStudio will display information about the file, so let's start enumerating some of the most important aspects we can get from it. Using the left panel, we can navigate through different sections that will share different types of information about the file. In the general information output displayed when opening the file, we can see the hash of the file in the form of **SHA-256**, The architecture type, in this case, **x64**, the file type, and the signature of the language used to compile the executable, in this case, **.NET framework** that uses the C# language.

![](pestudio2.png){: width="1006" height="476"}

Let's focus on some critical data we can obtain. First, if we want to identify the file and provide evidence of its alteration, we need to take note of the file’s SHA-256 hash, as we mentioned above, as well as the hash of each section on the PE file. PE stands for Portable Executable, and it's the format in which Windows executables are arranged; you can learn more about it [here](https://learn.microsoft.com/en-us/windows/win32/debug/pe-format). 

[The sections](https://learn.microsoft.com/en-us/windows/win32/debug/pe-format#section-table-section-headers) represent a memory space with different content of the Windows executable within the PE format. We can calculate the hash of these sections in order to identify the executable properly. We'll focus this time on two hashes: the one from the [.text](https://learn.microsoft.com/en-us/windows/win32/debug/pe-format#:~:text=in%20that%20module.-,.text,Executable%20code%20(free%20format),-IMAGE_SCN_CNT_CODE%20%7C%20IMAGE_SCN_MEM_EXECUTE%20%7C%20IIMAGE_SCN_MEM_READ) section, which is the section on the file containing the executable code; this section is often marked as Read and executable, so it should not have any alterations when the file is being copied. We can observe them in the screenshot below:

![](pestudio3.png){: width="1005" height="475"}

Another essential section we can use to obtain information about the binary is the "indicators" section. This section tells us about the potential indicators like URLs or other suspicious attributes of a binary.

![](pestudio4.png){: width="1005" height="505"}

The screenshot above shows several strings on the file, like file names, URLs, and function names. This can be very important depending on the file's execution flow. Additionally, looking for artefacts such as IP addresses, URLs, and crypto wallets can be a "quick win" for gathering some identifying intelligence. We'll learn about that in the next section.

Now that we have information about the file we are investigating, let's try to understand what the executable is doing. We need to understand its flow. If we try to read the file by opening it, we cannot do it since it's in binary format. In the previous section, we learned that the file is compiled using the **.NET** framework used by the **C#** language; we can decompile the binary code into C# using a decompilation tool like [ILSpy](https://github.com/icsharpcode/ILSpy).

This tool will decompile the code, providing us with readable information we can use to determine the flow of execution. Let's start by opening ILSpy from the taskbar and then click on **File > Open** and then navigate to **C:\Users\Administrator\Desktop\demo** and select the file **demo.exe**. The tool **ILSpy** may take up to 30 seconds to appear on the screen.

![](ilspy2.png){: width="1275" height="775"}

As we can observe from above, the left panel contains the libraries used by the framework, and the actual decompiled code is in the section with the file name **demo**. Let's click on it to expand and see what it contains.

![](ilspy3.png){: width="1162" height="670"}

As the screenshot above shows, ILSpy can provide much information, like metadata and references. However, the actual show is displayed on the brackets symbols {}, in this case, under **DemoBinary > Program > Main**, which is actually the Main function of the executable. Now that we have access to the code running on the binary, let's analyse it.

```c#
private static void Main(string[] args)
{
  Console.WriteLine("Hello THM DEMO Binary");
  Thread.Sleep(5000);
  string address = "http://10.10.10.10/img/tryhackme_connect.png";
  string text = Path.Combine(Environment.GetFolderPath(Environment.SpecialFolder.Desktop), "thm-demo.png");
  using (WebClient webClient = new WebClient())
  {
    try
    {
      Console.WriteLine("Downloading file...");
      webClient.DownloadFile(address, text);
      Console.WriteLine("File downloaded to: " + text);
      Process.Start(new ProcessStartInfo(text)
      {
        UseShellExecute = true
      });
      Console.WriteLine("Image opened successfully.");
    }
    catch (Exception ex)
    {
      Console.WriteLine("An error occurred: " + ex.Message);
    }
  }
  Console.WriteLine("Bye Bye leaving Demo Binary");
  Thread.Sleep(5000);
}
```
The code above displays the main function and its code. We can observe that first, it prints to the screen the message "Hello THM DEMO Binary" using the [Console.Writeline](https://learn.microsoft.com/en-us/dotnet/api/system.console.writeline?view=net-8.0) method.

```c#
Console.WriteLine("Hello THM DEMO Binary");
```
It then uses the [Sleep](https://learn.microsoft.com/en-us/dotnet/api/system.threading.thread.sleep?view=net-8.0) method to wait for 5 seconds.

```c#
Thread.Sleep(5000);
```
Then, it assigns a value to two string variables: address and text, the first one with a URL accessing a PNG file, and the second one with a file name on the user's Desktop named **thm-demo.png**.

```c#
string address = "http://10.10.10.10/img/tryhackme_connect.png";
string text = Path.Combine(Environment.GetFolderPath(Environment.SpecialFolder.Desktop), "thm-demo.png");
```
Then, it will try to connect to the URL on the address variable and save the content to a file on the Desktop using the [WebClient class](https://learn.microsoft.com/en-us/dotnet/api/system.net.webclient?view=net-8.0), and it will then execute the downloaded file path assigned to the text variable using the [Process class](https://learn.microsoft.com/en-us/dotnet/api/system.diagnostics.process?view=net-8.0)  and the [Start method](https://learn.microsoft.com/en-us/dotnet/api/system.diagnostics.process.start?view=net-8.0#system-diagnostics-process-start(system-string), as displayed below.

```c#
using (WebClient webClient = new WebClient())
  {
    try
    {
      Console.WriteLine("Downloading file...");
      webClient.DownloadFile(address, text);
      Console.WriteLine("File downloaded to: " + text);
      Process.Start(new ProcessStartInfo(text)
      {
        UseShellExecute = true
      });
      Console.WriteLine("Image opened successfully.");
    }
    catch (Exception ex)
    {
      Console.WriteLine("An error occurred: " + ex.Message);
    }
  }
```
Finally, it prints the message **"Bye Bye, leaving THM DEMO Binary"** again to the console and waits for 5 seconds using the **Sleep** method before closing.

```c#
Console.WriteLine("By bye leaving THM Demo Binary");
Thread.Sleep(5000);
```
Great! We now understand what the binary is doing. It will download a PNG file to the user's Desktop from the URL: **http://MACHINE_IP/img/tryhackme_connect.png**. Let's execute the file and see if this is true. Once the binary starts, wait for the text **"Hello THM DEMO Binary"** to appear, then press **Enter** to download the file.

**Note**: We are only executing the binary because we are in a sandbox environment; execution of an unknown binary on a host machine is NOT recommended.

![](demo.png){: width="1223" height="806"}

After executing the file, we can observe that it was downloaded to the Desktop, and the messages print to the screen as expected. Also, the downloaded file is executed and opened using the default app for PNG **Paint**. Excellent, we successfully Reverse-Engineered the flow of the code.

Now that we have some practice, join McSkidy and help investigate the alerts coming from the file WarevilleApp.exe. Put on your reverse engineering hat and help decipher the mystery behind this suspicious binary.

**Note**: To answer the question, you will need to reverse the application **WarevilleApp.exe**, located at <strong>C:\Users\Administrator\Desktop\.</strong>

## Explanation

- Open **ILSpy** in VM, go to **File > Open > Desktop > WarevilleApp**
- On left side assemblies, go to **WarevilleApp > FancyApp > Form1 > DownloadAndExecutefile() : void > Hover to DownloadAndExecuteFile() text > Decompile** to gather the details

![](Q1_2_3.png){: width="861" height="535"}

Next open the browser in VM > **http://mayorc2.thm:8080/dw/explorer.exe**

In **ILSpy** import explorer.exe for analyzing
Navigate to dropdown **Explorer > FileCollector > Program > Main(string[] : void**

![](explorer.png){: width="957" height="837"}

**Explanation of the Explorer.exe**

- **Initialization**  
  - Defines a list of file extensions: `.docx`, `.pptx`, `.png`, `.gif`, and `.jpeg`.
  - Identifies the user's "My Pictures" folder using `Environment.GetFolderPath(Environment.SpecialFolder.MyPictures)`.

- **Search for Files**  
  - Logs the folder being searched.
  - Creates a temporary folder named `CollectedFiles` in the system's temp directory.
  - Searches all files in the "My Pictures" folder (including subdirectories) using `Directory.GetFiles`.

- **Filter and Copy Files**  
  - Iterates through each file found:
    - Checks if its extension matches the predefined list.
    - If it matches, copies the file to the `CollectedFiles` temporary directory.

- **Create a Zip Archive**  
  - After copying files:
    - If no matching files are found, logs that no files were collected and skips further actions.
    - Otherwise:
      - Deletes any existing `CollectedFiles.zip` in the temp folder to avoid conflicts.
      - Compresses the collected files into a ZIP archive named `CollectedFiles.zip` in the temp folder using `ZipFile.CreateFromDirectory`.

- **Copy the ZIP File**  
  - Copies the ZIP file from the temporary directory to the user's "My Pictures" folder.
  - If a ZIP file already exists in "My Pictures", deletes it before copying.

- **Upload the ZIP File**  
  - Logs the start of an upload process (calls `UploadFileToServer`, which is assumed to be anonymousc2.thm).

- **Cleanup**  
  - Deletes:
    - The `CollectedFiles` temporary directory and its contents.
    - The temporary `CollectedFiles.zip` from the system's temp folder.

- **Error Handling**  
  - Catches any exceptions that occur during the process and logs the error message.

In the same dropdown, click on **UploadFileToServer(string) : void** for the final answer.

![](Q5.png){: width="955" height="837"}

## Answer

### Question 1

What is the function name that downloads and executes files in the WarevilleApp.exe?

<details>
  <summary style="cursor:pointer; padding:10px; border:1px solid #ccc; background-color:#f0f0f0; user-select: none;">Answer</summary>
  <div style="padding:10px; border:1px solid #ccc;">
    <span onclick="navigator.clipboard.writeText('DownloadAndExecuteFile')" style="cursor:pointer;">DownloadAndExecuteFile</span>
    <i onclick="navigator.clipboard.writeText('DownloadAndExecuteFile')" style="float:right; cursor:pointer; font-size:16px;">&#x1F4C4;</i>
  </div>
</details>

### Question 2

Once you execute the WarevilleApp.exe, it downloads another binary to the Downloads folder. What is the name of the binary?

<details>
  <summary style="cursor:pointer; padding:10px; border:1px solid #ccc; background-color:#f0f0f0; user-select: none;">Answer</summary>
  <div style="padding:10px; border:1px solid #ccc;">
    <span onclick="navigator.clipboard.writeText('explorer.exe')" style="cursor:pointer;">explorer.exe</span>
    <i onclick="navigator.clipboard.writeText('explorer.exe')" style="float:right; cursor:pointer; font-size:16px;">&#x1F4C4;</i>
  </div>
</details>

### Question 3

What domain name is the one from where the file is downloaded after running WarevilleApp.exe?

<details>
  <summary style="cursor:pointer; padding:10px; border:1px solid #ccc; background-color:#f0f0f0; user-select: none;">Answer</summary>
  <div style="padding:10px; border:1px solid #ccc;">
    <span onclick="navigator.clipboard.writeText('mayorc2.thm')" style="cursor:pointer;">mayorc2.thm</span>
    <i onclick="navigator.clipboard.writeText('mayorc2.thm')" style="float:right; cursor:pointer; font-size:16px;">&#x1F4C4;</i>
  </div>
</details>

### Question 4

The stage 2 binary is executed automatically and creates a zip file comprising the victim's computer data; what is the name of the zip file?

<details>
  <summary style="cursor:pointer; padding:10px; border:1px solid #ccc; background-color:#f0f0f0; user-select: none;">Answer</summary>
  <div style="padding:10px; border:1px solid #ccc;">
    <span onclick="navigator.clipboard.writeText('CollectedFiles.zip')" style="cursor:pointer;">CollectedFiles.zip</span>
    <i onclick="navigator.clipboard.writeText('CollectedFiles.zip')" style="float:right; cursor:pointer; font-size:16px;">&#x1F4C4;</i>
  </div>
</details>

### Question 5

What is the name of the C2 server where the stage 2 binary tries to upload files?

<details>
  <summary style="cursor:pointer; padding:10px; border:1px solid #ccc; background-color:#f0f0f0; user-select: none;">Answer</summary>
  <div style="padding:10px; border:1px solid #ccc;">
    <span onclick="navigator.clipboard.writeText('anonymousc2.thm')" style="cursor:pointer;">anonymousc2.thm</span>
    <i onclick="navigator.clipboard.writeText('anonymousc2.thm')" style="float:right; cursor:pointer; font-size:16px;">&#x1F4C4;</i>
  </div>
</details>

