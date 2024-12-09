---
title: "TryHackMe: Advent Of Cyber 2024 Day 3"
categories: [TryHackMe]
tags: [log analysis, elastic, rce, ELK, KQL, advent of cyber]
render_with_liquid: false
img_path: /images/tryhackme_aoc2024_day3/
image:
  path: banner.png
---

Today's AoC challenge follows a rather unfortunate series of events for the Glitch. Here is a little passage which sets the scene for today's task: 

![Tryhackme Room Link](bell.png){: width="1152" height="300" .shadow }
_<https://tryhackme.com/r/room/adventofcyber2024>_

<div style="text-align: center; font-style: italic;">
  Late one Christmas evening the Glitch had a feeling,
  Something forgotten as he stared at the ceiling.
  He got up out of bed and decided to check,
  A note on his wall: ”Two days! InsnowSec”.

</div>

<div style="text-align: center; font-style: italic;">
  With a click and a type he got his hotel and tickets,
  And sank off to sleep to the sound of some crickets.
  Luggage in hand, he had arrived at Frosty Pines,
  “To get to the conference, just follow the signs”.
</div>

<div style="text-align: center; font-style: italic;">
  Just as he was ready the Glitch got a fright,
  An RCE vulnerability on their website ?!?
  He exploited it quick and made a report,
  But before he could send arrived his transport.
</div>

<div style="text-align: center; font-style: italic;">
  In the Frosty Pines SOC they saw an alert,
  This looked quite bad, they called an expert.
  The request came from a room, but they couldn’t tell which,
  The logs saved the day, it was the room of…the Glitch.
</div>



![](cabin.png){: width="1100" height="706"}

In this task, we will cover how the SOC team and their expert were able to find out what had happened (Operation Blue) and how the Glitch was able to gain access to the website in the first place (Operation Red). Let's get started, shall we?

## Learning Objectives
- Learn about Log analysis and tools like ELK.
- Learn about KQL and how it can be used to investigate logs using ELK.
- Learn about RCE (Remote Code Execution), and how this can be done via insecure file upload.

## OPERATION BLUE

In this section of the lesson, we will take a look at what tools and knowledge is required for the blue segment, that is the investigation of the attack itself using tools which enable is to analyse the logs. 

For the first part of Operation Blue, we will demonstrate how to use ELK to analyse the logs of a demonstration web app - WareVille Rails. Feel free to following along for practice. 

## Log Analysis & Introducing ELK

Log analysis is crucial to blue-teaming work, as you have likely discovered through this year's Advent of Cyber.

Analysing logs can quickly become overwhelming, especially if you have multiple devices and services. ELK, or Elasticsearch, Logstash, and Kibana, combines data analytics and processing tools to make analysing logs much more manageable. ELK forms a dedicated stack that can aggregate logs from multiple sources into one central place.

Explaining how ELK collates and processes these logs is out of the scope of today's task. However, if you wish to learn more, you can check out the Investigating with ELK 101 room. For now, it's important to note that multiple processes behind the scenes achieve this.

The first part of today's task is to investigate the attack on Frosty Pines Resort's Hotel Management System to see what it looks like to a blue teamer. You will then test your web app skills by recreating the attack.

## Using ELK

Upon loading the URL http://MACHINE_IP:5601/ within your AttackBox’s browser, you will be greeted with the ELK Home page.

For today's task, we will use Kibana's Discover interface to review Apache2 logs. To access this, simply click on the three lines located at the top left of the page to open the slide-out tray. Under the Analytics heading, click on Discover.

![](elk.gif){: width="728" height="742"}
