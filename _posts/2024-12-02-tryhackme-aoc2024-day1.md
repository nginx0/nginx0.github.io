---
title: "TryHackMe: Advent Of Cyber 2024 Day 1"
categories: [TryHackMe]
tags: [web, vhost, subdomain, ufw, firewall, ftp, sudo, apt]
render_with_liquid: false
img_path: /images/tryhackme_aoc2024_day1/
image:
  path: main_banner.png
---

McSkidy's fingers flew across the keyboard, her eyes narrowing at the suspicious website on her screen. She had seen dozens of malware campaigns like this. This time, the trail led straight to someone who went by the name "Glitch."

"Too easy," she muttered with a smirk.

"I still have time," she said, leaning closer to the screen. "Maybe there's more."

Little did she know, beneath the surface lay something far more complex than a simple hacker's handle. This was just the beginning of a tangled web unravelling everything she thought she knew.

![Main Banner](/images/tryhackme_aoc2024_day1/main_banner.png){: .center }


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