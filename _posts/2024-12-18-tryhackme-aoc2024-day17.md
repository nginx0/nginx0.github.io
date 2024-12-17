---
title: "TryHackMe: Advent Of Cyber 2024 Day 17"
categories: [TryHackMe]
tags: [splunk, advent of cyber]
render_with_liquid: false
img_path: /images/tryhackme_aoc2024_day17/
image:
  path: banner.png
---

<p align="center"><i>An attack now, it seems, on the town's CCTV,</i><br>
<i>There's a problem with the logs, but what could it be?</i><br>
<i>An idea put forward of a log format switch,</i><br>
<i>Not as expected, the idea of the Glitch!</i></p>

![Tryhackme Room Link](bell.png){: width="1152" height="300" .shadow }
_<https://tryhackme.com/r/room/adventofcyber2024>_

## Background Story

Marta May Ware is going crazy: someone has disconnected the main server from the Wareville network, and nobody knows who it is! As soon as she realized it, she contacted Wareville's top physical security company, WareSec&Aware, to let her view the data centre's CCTV streams. They forbade it entirely: for privacy reasons, only the camera owner can view the recordings. Not even the WareSec&Aware employees themselves are allowed to do so.

Still, they said there was no recording of anybody entering the data centre yesterday! How could that be, wondered Marta May, desperate for answers. Their first supposition was that the owner of the cameras must have deleted the recordings from their managing web page. But the data centre's camera owner surely can't be the perpetrator: it is no other than Byte, Glitch's dog! Glitch insisted with Marta to leave the ownership of the cameras to Byte precisely to avoid these kinds of happenings: Byte, the ultimate good boy, combines loyalty and sharp instincts to keep any place safe.

Marta May calls Glitch and McSkidy right away, explaining the situation in between the sobs. Glitch's eyes darken: Someone is trying to frame Byte, and he will not let anybody vex his beautiful dog!

McSkidy is perplexed: why are the people at WareSec&Aware "supposing" that Byte had deleted the recordings? Shouldn't they have some logs to prove such an accusation?

Marta May has the answer: they do have some log files that they back up every 6 hours, give or take.

But they can't search through it—or rather, they tried, but when they go and search for some keyword like the data centre's cameras' IDs or the action "delete", this is what they get:

```console
user@tryhackme$ cat cctv_logs.log| grep -i "11"         
2024-12-16 22:53:06 WatchCamera 5 byte 11 rij5uu4gt204q0d3eb7jj86okt
RecordingInfo: 1 11 rij5uu4gt204q0d3eb7jj86okt
2024-12-16 22:53:22 WatchCamera 5 byte 11 rij5uu4gt204q0d3eb7jj86okt
RecordingInfo: 1 11 rij5uu4gt204q0d3eb7jj86okt
2024-12-16 22:53:25 WatchCamera 5 byte 11 rij5uu4gt204q0d3eb7jj86okt
user@tryhackme$ 
user@tryhackme$ cat cctv_logs.log| grep -i "download"               
2024-12-16 22:52:50 DownloadRecording 5 byte 51 10 opfg6ns9khsbpq0u4us6dro2m8
```
Unreadable!

McSkidy shakes her head: they must immediately send the log file to the SOC team! Armed with a SIEM, no log is unsearchable!

## Learning Objectives

In this task, we will explore the following learning objectives while investigating the logs related to the incident scenario explained above:

- Learn how to extract custom fields in Splunk
- Learn to create a parser for the custom logs
- Filter and narrow down the search results using Search Processing Language (SPL)
- How to investigate in Splunk

## Investigation Time

It's time to fire up Splunk, where the data has been pre-ingested for us to investigate the incident. Once the lab is connected, open up the link in the browser and click on **Search & Reporting** on the left.

![](splunk1.png){: width="1364" height="792"}

<!-- HTML inside Markdown -->
<lottie-player
    src="https://lottie.host/949e2e8d-d536-432b-ac05-ba04d8ad4f57/yYkj8wQAZ4.json"
    background="transparent"
    speed="1"
    style="width: 1200px; height: 1280px;"
    loop
    autoplay>
</lottie-player>

<script src="https://unpkg.com/@lottiefiles/lottie-player@latest/dist/lottie-player.js"></script>









![](tree.png){: width="800" height="800"}
![](tree.png){: width="800" height="800"}
![](tree.png){: width="800" height="800"}
![](tree.png){: width="800" height="800"}
![](tree.png){: width="800" height="800"}
![](tree.png){: width="800" height="800"}
![](tree.png){: width="800" height="800"}
![](tree.png){: width="800" height="800"}
![](tree.png){: width="800" height="800"}
![](tree.png){: width="800" height="800"}
![](tree.png){: width="800" height="800"}
![](tree.png){: width="800" height="800"}
![](tree.png){: width="800" height="800"}
![](tree.png){: width="800" height="800"}
![](tree.png){: width="800" height="800"}
![](tree.png){: width="800" height="800"}
![](tree.png){: width="800" height="800"}
![](tree.png){: width="800" height="800"}
![](tree.png){: width="800" height="800"}
![](tree.png){: width="800" height="800"}
![](tree.png){: width="800" height="800"}
![](tree.png){: width="800" height="800"}
![](tree.png){: width="800" height="800"}
![](tree.png){: width="800" height="800"}
![](tree.png){: width="800" height="800"}
![](tree.png){: width="800" height="800"}
![](tree.png){: width="800" height="800"}
![](tree.png){: width="800" height="800"}
![](tree.png){: width="800" height="800"}
![](tree.png){: width="800" height="800"}
![](tree.png){: width="800" height="800"}
![](tree.png){: width="800" height="800"}
![](tree.png){: width="800" height="800"}
![](tree.png){: width="800" height="800"}
![](tree.png){: width="800" height="800"}
![](tree.png){: width="800" height="800"}
![](tree.png){: width="800" height="800"}
![](tree.png){: width="800" height="800"}
![](tree.png){: width="800" height="800"}
![](tree.png){: width="800" height="800"}
![](tree.png){: width="800" height="800"}
![](tree.png){: width="800" height="800"}
![](tree.png){: width="800" height="800"}
![](tree.png){: width="800" height="800"}
