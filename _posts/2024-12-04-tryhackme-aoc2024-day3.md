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
  Late one Christmas evening the Glitch had a feeling,<br>
  Something forgotten as he stared at the ceiling.<br>
  He got up out of bed and decided to check,<br>
  A note on his wall: ”Two days! InsnowSec”.<br><br>
</div>

<div style="text-align: center; font-style: italic;">
  With a click and a type he got his hotel and tickets,<br>
  And sank off to sleep to the sound of some crickets.<br>
  Luggage in hand, he had arrived at Frosty Pines,<br>
  “To get to the conference, just follow the signs”.<br><br>
</div>

<div style="text-align: center; font-style: italic;">
  Just as he was ready the Glitch got a fright,<br>
  An RCE vulnerability on their website ?!?<br>
  He exploited it quick and made a report,<br>
  But before he could send arrived his transport.<br><br>
</div>

<div style="text-align: center; font-style: italic;">
  In the Frosty Pines SOC they saw an alert,<br>
  This looked quite bad, they called an expert.<br>
  The request came from a room, but they couldn’t tell which,<br>
  The logs saved the day, it was the room of…the Glitch.<br><br>
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

We will need to select the collection that is relevant to us. A collection is a group of logs. For this stage of Operation Blue, we will be reviewing the logs present within the "wareville-rails" collection. To select this collection, click on the dropdown on the left of the display.

![](indexpattern.png){: width="504" height="413"}

Once you have done this, you will be greeted with a screen saying, "No results match your search criteria". This is because no logs have been ingested within the last 15 minutes. Do not panic; we will discuss how to change this shortly.

![](noresult.png){: width="1804" height="700"}

To change the date and time, click the text located on the right side of the box that has the calendar icon. Select "**Absolute**" from the dropdown, where you can now select the start date and time. Next, click on the text on the right side of the arrow to and repeat the process for the end date and time.

For the WareVille Rails collection, we will need to set the start time to **October 1 2024 00:00:00**, and the end time to **October 1 23:30:00**

If you are stuck, refer to the GIF below. Please note that the day and time in this demonstration of WareVille Rails will differ from the times required to review the FrostyPines Resorts collection in the second half of the practical.

![](date.gif){: width="2533" height="926"}

Now that we can see some entries, let's go over the basics of the Kibana Discover UI.

![](ui.png){: width="1906" height="918"}

- **Search Bar**: Here, we can place our search queries using KQL
- **Index Pattern**: An index pattern is a collection of logs. This can be from a specific host or, for example, multiple hosts with a similar purpose (such as multiple web servers). In this case, the index pattern is all logs relating to "wareville-rails"
- **Fields**: This pane shows us the fields that Elasticsearch has parsed from the logs. For example, timestamp, response type, and IP address.
- **Timeline**: This visualisation displays the event count over a period of time
- **Documents (Logs)**: These entries are the specific entries in the log file
- **Time Filter**: We can use this to narrow down a specific time frame (absolute). Alternatively, we can search for logs based on relativity. I.e. "Last 7 days".

## Kibana Query Language (KQL)

KQL, or Kibana Query Language, is an easy-to-use language that can be used to search documents for values. For example, querying if a value within a field exists or matches a value. If you are familiar with Splunk, you may be thinking of SPL (Search Processing Language).

For example, the query to search all documents for an IP address may look like **ip.address: "10.10.10.10"**. 

![](query.png){: width="241" height="130"}


| **Query/Syntax** | **Description** | **Example** |
|------------------|-----------------|-------------|
| " "              | The two quotation marks are used to search for specific values within the documents. Values in quotation marks are used for exact searches. | "TryHackMe" |
| *                | The asterisk denotes a wildcard, which searches documents for similar matches to the value provided. | United* (would return United Kingdom and United States) |
| OR               | This logical operator is used to show documents that contain either of the values provided. | "United Kingdom" OR "England" |
| AND              | This logical operator is used to show documents that contain both values. | "Ben" AND "25" |
| :                | This is used to search the (specified) field of a document for a value, such as an IP address. | ip.address: 10.10.10.10 |


## Investigating a Web Attack With ELK

Scenario: Thanks to our extensive intrusion detection capabilities, our systems alerted the SOC team to a web shell being uploaded to the WareVille Rails booking platform on Oct 1, 2024. Our task is to review the web server logs to determine how the attacker achieved this.

If you would like to follow along, ensure that you have the "**wareville-rails**" collection selected like so:

![](filter.png){: width="504" height="413"}

To investigate this scenario, let's change the time filter to show events for the day of the attack, setting the start date and time to "**Oct 1, 2024 @ 00:00:00.000**" and the end date and time to "**Oct 2, 2024 @ 00:00:00.000**".

![](month.png){: width="606" height="424"}

You will see the logs have now populated within the display. Please note that the quantity of entries (hits) in this task may differ to the amount on the practical VM.


