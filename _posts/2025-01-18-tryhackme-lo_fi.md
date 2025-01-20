---
title: "TryHackMe: Lo-Fi"
categories: [TryHackMe]
tags: [LFI]
render_with_liquid: false
img_path: /images/tryhackme_lo_fi/
image:
  path: banner.png
---

Want to hear some lo-fi beats, to relax or study to? We've got you covered! 

![](room_card.png){: width="300" height="300" .shadow}
_<https://tryhackme.com/r/room/lofi>_

## Reconnaissance

Nmap scan results reveal two open ports: port 22 for SSH and port 80 for HTTP.

```console
nmap -sC -sV -T3 -Pn --open <MACHINE_IP>
```
![](nmap.png){: width="996" height="434"}

We run a scan on the page with Feroxbuster, but after completing the scan, we don’t find any significant results.

![](ferox.png){: width="971" height="509"}

![](coffee.png){: width="1919" height="648"}

## Web Analysis

The room description hinting that we should test for potential [Local File Inclusion](https://owasp.org/www-project-web-security-testing-guide/v42/4-Web_Application_Security_Testing/07-Input_Validation_Testing/11.1-Testing_for_Local_File_Inclusion) (LFI) vulnerabilities. Upon visiting the index page, we are presented with links to various genres, each pointing to a different section of the site. These links are dynamically loaded based on user interaction, which suggests the possibility of parameter manipulation.

![](main.png){: width="1915" height="714"}

## Local File Inclusion

Testing for LFI in this scenario involves analyzing how the application handles the page parameter in the URL. Each genre link likely passes a specific value to this parameter, which the server processes to include the corresponding file. If proper validation is not implemented, this functionality can be exploited to access unintended files on the server.

By manipulating the page parameter with directory traversal techniques, we can assess whether the application is vulnerable. This setup provides an excellent opportunity to test common LFI attack strategies while understanding the impact of such vulnerabilities on real-world systems.

Looking at the source code of the page, we could see that it's possible to perform path traversal attack.

![](source.png){: width="1452" height="707"}

We test the page parameter for LFI vulnerabilities by trying directory traversal payloads **(../../../../etc/passwd)**. When we submit the payload, it confirmed that the application is indeed vulnerable.

![](lfi.png){: width="1916" height="635"}

## Retrieving root flag

By changing our focus to the `/flag.txt` file located in the root directory, as specified in the task, we attempt to include it using the same LFI technique. After submitting the payload, we successfully retrieve the flag.

```
<MACHINE_IP>/?page=../../../../../flag.txt
```

![](flag.png){: width="1919" height="670"}

## Answer

<details>
  <summary style="cursor:pointer; padding:10px; border:1px solid #ccc; background-color:#f0f0f0; user-select: none;">Answer</summary>
  <div style="padding:10px; border:1px solid #ccc;">
    <span onclick="navigator.clipboard.writeText('flag{e4478e0eab69bd642b8238765dcb7d18}')" style="cursor:pointer;">flag{e4478e0eab69bd642b8238765dcb7d18}</span>
    <i onclick="navigator.clipboard.writeText('flag{e4478e0eab69bd642b8238765dcb7d18}')" style="float:right; cursor:pointer; font-size:16px;">&#x1F4C4;</i>
  </div>
</details>

