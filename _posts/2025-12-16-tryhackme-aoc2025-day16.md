---
title: "TryHackMe: Advent Of Cyber 2025 Day 16 (Forensics - Registry Furensics)"
categories: [TryHackMe]
tags: [dfir, windows, advent of cyber]
render_with_liquid: false
media_subpath: /images/tryhackme_aoc2025_day16/
image:
  path: banner.png
---

Learn what the Windows Registry is and how to investigate it.

<h2 style="color:#A64AC9; text-align:center; font-weight:700; font-size:1.8em; text-shadow: 1px 0 #A64AC9, -1px 0 #A64AC9, 0 1px #A64AC9, 0 -1px #A64AC9;">
  The Story
</h2>

[![TryHackMe Room Link](header.svg){: width="1200" height="407" }](https://tryhackme.com/room/registry-forensics-aoc2025-h6k9j2l5p8)

## Introduction

TBFC is under attack. Systems are exhibiting weird behavior, and the company is now feeling the absence of its lead defender, McSkidy. However, McSkidy made sure the legacy continues.

McSkidy’s team, determined and well-trained, is fully confident in securing all the systems and regaining control before the big event, SOCMAS.

They have now decided to conduct a detailed forensic analysis on one of the most critical systems of TBFC, `dispatch-srv01`. The `dispatch-srv01` coordinates the drone-based gifts delivery during SOCMAS. However, recently it was compromised by King Malhare’s bandits of bunnies.

TBFC’s defenders have decided to split into specialized teams to uncover the attack on this system through detailed forensics. While some of the other team members investigate logs, memory dumps, file systems, and other artefacts, you will work to investigate the registry of this compromised system.

![](alert.png){: width="1080" height="1080"}

## Windows Registry

Your brain stores all the information that you need to function effectively. This includes:

- How should you behave?
- What would be the first thing you would do after waking up?
- How would you dress yourself?
- What are your habits?
- What happened in the recent past?

These are just a few things. Your brain knows pretty much everything about you. It's just like a database storing the human configuration.

Windows OS is not a human, but it also needs a brain to store all its configurations. This brain is known as the Windows Registry. The registry contains all the information that the Windows OS needs for its functioning. 

Now, this Windows brain (Registry) is not stored in one single place, unlike a human brain, which is situated in one single place inside the head. It is made up of several separate files, each storing information on different configuration settings. These files are known as **Hives**.

![](hives.png){: width="1080" height="1080"}

Let's take a look at all these hives in the table below. The first column contains the hive names, the second column contains the type of configuration settings that each hive stores, and the third column contains the location of each hive on the disk. 

| **Hive Name**      | **Contains**                                           | **Location**                                    |
|-------------------|-------------------------------------------------------|------------------------------------------------|
| SYSTEM            | Services<br>Mounted Devices<br>Boot Configuration<br>Drivers<br>Hardware | `C:\Windows\System32\config\SYSTEM`          |
| SECURITY          | Local Security Policies<br>Audit Policy Settings     | `C:\Windows\System32\config\SECURITY`        |
| SOFTWARE          | Installed Programs<br>OS Version and other info<br>Autostarts<br>Program Settings | `C:\Windows\System32\config\SOFTWARE`       |
| SAM               | Usernames and their Metadata<br>Password Hashes<br>Group Memberships<br>Account Statuses | `C:\Windows\System32\config\SAM`            |
| NTUSER.DAT        | Recent Files<br>User Preferences<br>User-specific Autostarts | `C:\Users\username\NTUSER.DAT`              |
| USRCLASS.DAT      | Shellbags<br>Jump Lists                               | `C:\Users\username\AppData\Local\Microsoft\Windows\USRCLASS.DAT` |

**Note:** The configuration settings stored in each hive listed above are just a few examples. Each hive stores more than these.

Now that you understand where these Registry Hives are stored, you may be tempted to double-click and open these files from their respective locations to view the data they contain. But here's the twist. These Registry Hives contain binary data that cannot be opened directly from the file. So, double-clicking them would only display things you won't ever understand. Then how can we possibly view the registry data?

The Windows OS has a built-in tool known as the Registry Editor, which allows you to view all the registry data available in these hives. You can go ahead and open this tool by simply typing Registry Editor in your search bar.

![](regedit.png){: width="1106" height="384"}

As you can see in the Registry Editor screenshot above, there are some folders named `HKEY_LOCAL_MACHINE`, `HKEY_CURRENT_USER`, and more. But didn't we expect `SYSTEM`, `SECURITY`, `SOFTWARE`, etc., to be seen so we can view their data? Don't worry, Windows organizes all the Registry Hives into these structured Root Keys. Instead of seeing the Registry Hives, you would always get these registry root keys whenever you open the registry. Now, which registry key contains which registry hive's data? To answer this question, we have a table below that maps the registry keys with their respective Registry Hives.

| **Hive on Disk**  | **Where You See It in Registry Editor**                     |
|------------------|------------------------------------------------------------|
| SYSTEM           | `HKEY_LOCAL_MACHINE\SYSTEM`                                |
| SECURITY         | `HKEY_LOCAL_MACHINE\SECURITY`                              |
| SOFTWARE         | `HKEY_LOCAL_MACHINE\SOFTWARE`                              |
| SAM              | `HKEY_LOCAL_MACHINE\SAM`                                   |
| NTUSER.DAT       | `HKEY_LOCAL_MACHINE\<SID>`                                  |
| USRCLASS.DAT     | `HKEY_LOCAL_MACHINE\<SID>\Software\Classes`                |

In the table above, you can see that most of the Registry Hives are located under the `HKEY_LOCAL_MACHINE (HKLM)` key. We can verify this by clicking on the little toggle arrow at the on the left side of the `HKLM` key in the Registry Editor, as shown in the screenshot below:

![](regedit2.png){: width="1786" height="596"}

As you can see, the `SYSTEM`, `SOFTWARE`, `SECURITY`, and `SAM` hives are under the `HKLM` key. `NTUSER.DAT` and `USRCLASS.DAT` are located under `HKEY_USERS (HKU)` and `HKEY_CURRENT_USER (HKCU)`. 

**Note:** The other two keys `(HKEY_CLASSES_ROOT (HKCR)` and `HKEY_CURRENT_CONFIG (HKCC)` are not part of any separate hive files. They are dynamically populated when Windows is running.

So far, we have learned what the registry is, where it is located (in separate Registry Hives), and how to view the registry through the Registry Editor, which displays the registry keys backed by these Registry Hives. 

Now, let's see how we can actually extract information from the registry. Think of it like navigating to different files in the file explorer. If you know the path where the file is stored, you can go directly to that location to find the file. Let's take a look at a few examples:

## Example 1: View Connected USB Devices

**Note:** The registry key contents explained in this example are not available in the attached VM.

The registry stores information on the USB devices that have been connected to the system. This information is present in the `SYSTEM` hive. To view it:

1. Open the Registry Editor.
2. Navigate to the following path: `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Enum\USBSTOR`.
3. Here you will see the USB devices' information (make, model, and device ID).
4. Each device will have the following:
- A main subkey that is the identification of the type and manufacturer of the USB device.
- A subkey under the above (for example) that represents the unique devices under this model.

![](regedit3.png){: width="2056" height="1068"}

## Example 2: View Programs Run by the User

The registry stores information on the programs that the user ran using the Run dialog `Win + R`. This information is present in the `NTUSER.DAT` hive. To view it:

1. Open the Registry Editor.
2. Navigate to the following path: `HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Explorer\RunMRU`.
3. Here you will see the list of commands typed by the user in the Run dialog to run applications.

![](regedit4.png){: width="2050" height="1076"}

## Registry Forensics

Since the registry contains a wide range of data about the Windows system, it plays a crucial role in forensic investigations. Registry forensics is the process of extracting and analyzing evidence from the registry. In Windows digital forensic investigations, investigators analyze registry, event logs, file system data, memory data, and other relevant data to construct the whole incident timeline. 

The table below lists some registry keys that are particularly useful during forensic investigations.

| Registry Key | Importance |
|-------------|------------|
| `HKCU\Software\Microsoft\Windows\CurrentVersion\Explorer\UserAssist` | Stores information on recently accessed applications launched via the GUI. |
| `HKCU\Software\Microsoft\Windows\CurrentVersion\Explorer\TypedPaths` | Stores all paths and locations typed by the user in the Explorer address bar. |
| `HKLM\Software\Microsoft\Windows\CurrentVersion\App Paths` | Stores the paths of installed applications. |
| `HKCU\Software\Microsoft\Windows\CurrentVersion\Explorer\WordWheelQuery` | Stores all search terms typed by the user in the Explorer search bar. |
| `HKLM\Software\Microsoft\Windows\CurrentVersion\Run` | Stores information on programs configured to automatically start when the user logs in. |
| `HKCU\Software\Microsoft\Windows\CurrentVersion\Explorer\RecentDocs` | Stores information on files recently accessed by the user. |
| `HKLM\SYSTEM\CurrentControlSet\Control\ComputerName\ComputerName` | Stores the computer’s name (hostname). |
| `HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall` | Stores information about installed programs. |

Numerous other registry keys can be used for extracting important evidence from a Windows system during an incident investigation. The investigation of these registry keys during forensics cannot be done via the built-in Registry Editor tool. It is because the Registry analysis cannot be done on the system under investigation (due to the chance of modification), so we collect the Registry Hives and open them offline into our forensic workstation. However, the Registry Editor does not allow opening offline hives. The Register editor also displays some of the key values in binary which are not readable.

To solve this problem, there are some tools built for registry forensics. In this task you will use the [Registry Explorer](https://ericzimmerman.github.io/) tool which is a registry forensics tool. It is open source and can parse the binary data out of the registry, and we can analyze it without the fear of modification.

## Practical

In this practical example, we will use the Registry Explorer tool to analyze the Registry Hives from the compromised system, `dispatch-srv01`. The Registry Hives have been collected and are available in the folder `C:\Users\Administrator\Desktop\Registry Hives` on the machine attached to this task.

**Step 1: Launch Registry Explorer**

Click on the Registry Explorer icon pinned to the taskbar of the target machine to launch it.

![](reg_explorer.png){: width="2964" height="1826"}

**Step 2: Load the Registry Hives**

Once Registry Explorer opens with an empty interface, follow these steps to load the hives:

1. Click the **File** option from the top menu
2. Select **Load hive** from the dropdown

![](hive.png){: width="3216" height="1844"}

**Step 3: Handling Dirty Hives**

While loading Registry Hives, it is important to know that these Registry Hives can sometimes be "dirty" when collected from live systems, meaning they may have incomplete transactions. To ensure clean loading:

1.On the **Load hives** pop-up, navigate to `C:\Users\Administrator\Desktop\Registry Hives`
2. Select the desired hive file (e.g., SYSTEM)
3. **Hold SHIFT**, then press **Open** to load associated transaction log files. This ensures you get a clean, consistent hive state for analysis.

![](reg_explorer2.png){: width="2560" height="1616"}

4. You'll be prompted with a message indicating successful replay for transaction logs
5. Repeat the same process for all the other hives you want to load

![](reg_explorer3.png){: width="2564" height="1610"}

**Step 4: Investigating Registry Keys**

After loading the `SYSTEM` hive, you can navigate to specific registry keys for investigation. Let's practice by finding the computer name:

- Navigate to: `ROOT\ControlSet001\Control\ComputerName\ComputerName`. Or you can also just type "ComputerName" in the search bar to quickly locate the key, as shown below.

![](reg_explorer4.png){: width="3506" height="938"}

- Alternatively, you can click the **Available Bookmarks** tab and navigate to the **ComputerName** key from there.
- Examine the values to identify the system's hostname. Under the **Data** value, you'll find `DISPATCH-SRV01`.

![](reg_explorer5.png){: width="2456" height="1068"}

Now that you understand how to load hives and navigate in Registry Explorer, you're ready to begin your forensic investigation and uncover evidence of the TBFC intrusion on the Dispatch server!

**Note:** The abnormal activity on the `dispatch-srv01` started on 21st October, 2025.

**Tip:** The table given in the Registry Forensics explanation is going to be your friend.

## Answer

### Question 1

What application was installed on the `dispatch-srv01` before the abnormal activity started?

<details>
  <summary style="cursor:pointer; padding:10px; border:1px solid #888; background-color:#444; color:#fff; user-select: none;">
    Answer
  </summary>
  <div style="padding:10px; border:1px solid #888; background-color:#333; color:#fff;">
    <span onclick="navigator.clipboard.writeText('DroneManager Updater')" style="cursor:pointer;">DroneManager Updater</span>
    <i onclick="navigator.clipboard.writeText('DroneManager Updater')" style="float:right; cursor:pointer; font-size:16px;">&#x1F4C4;</i>
  </div>
</details>

### Question 2

What is the full path where the user launched the application (found in question 1) from?

<details>
  <summary style="cursor:pointer; padding:10px; border:1px solid #888; background-color:#444; color:#fff; user-select: none;">
    Answer
  </summary>
  <div style="padding:10px; border:1px solid #888; background-color:#333; color:#fff;">
    <span onclick="navigator.clipboard.writeText('C:\Users\dispatch.admin\Downloads\DroneManager_Setup.exe')" style="cursor:pointer;">C:\Users\dispatch.admin\Downloads\DroneManager_Setup.exe</span>
    <i onclick="navigator.clipboard.writeText('C:\Users\dispatch.admin\Downloads\DroneManager_Setup.exe')" style="float:right; cursor:pointer; font-size:16px;">&#x1F4C4;</i>
  </div>
</details>

### Question 3

Which value was added by the application to maintain persistence on startup?

<details>
  <summary style="cursor:pointer; padding:10px; border:1px solid #888; background-color:#444; color:#fff; user-select: none;">
    Answer
  </summary>
  <div style="padding:10px; border:1px solid #888; background-color:#333; color:#fff;">
    <span
      onclick="navigator.clipboard.writeText('&quot;C:\\Program Files\\DroneManager\\dronehelper.exe&quot; --background')"
      style="cursor:pointer;">
      "C:\Program Files\DroneManager\dronehelper.exe" --background
    </span>
    <i
      onclick="navigator.clipboard.writeText('&quot;C:\\Program Files\\DroneManager\\dronehelper.exe&quot; --background')"
      style="float:right; cursor:pointer; font-size:16px;">
      &#x1F4C4;
    </i>
  </div>
</details>