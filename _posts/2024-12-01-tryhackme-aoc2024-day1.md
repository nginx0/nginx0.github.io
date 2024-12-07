---
title: 'TryHackMe: Advent Of Cyber 2024 : Day 1'
categories: [TryHackMe]
tags: [web, vhost, subdomain, ufw, firewall, ftp, sudo, apt]
render_with_liquid: false
img_path: /images/tryhackme_aoc2024day1/
image:
  path: main_banner.png
---

Dodge started by inspecting the certificate of a https webserver to get a list of subdomains and enumerating these subdomains to find a PHP endpoint that allowed disabling the UFW firewall. After disabling the firewall, it was possible to access a FTP server and get a SSH key for a user, which allowed us to get a shell on the machine. After this, using port forwarding to access an internal website and logging in with the credentials found in the comments of the same website gave us credentials for another user. With this new user, we were able to abuse sudo privileges and get a shell as root.

![Tryhackme Room Link](bell.svg){: width="1152" height="300" .shadow }
_<https://tryhackme.com/room/dodge>_

