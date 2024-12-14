---
title: "TryHackMe: Advent Of Cyber 2024 Day 14"
categories: [TryHackMe]
tags: [certificate, mitm, advent of cyber]
render_with_liquid: false
img_path: /images/tryhackme_aoc2024_day14/
image:
  path: banner.png
---

It’s a quiet morning in the town of Wareville. A wholesome town where cheer and tech come together. McSkidy is charged to protect the GiftScheduler, the service elves use to schedule all the presents to be delivered in Wareville. She assigned Glitch to the case to make sure the site is secure for G-Day (Gift Day). In the meantime, Mayor Malware works tirelessly, hoping to not only ruin Christmas by redirecting presents to the wrong addresses but also to ensure that Glitch is blamed for the attack. After all, Glitch’s warnings about the same vulnerabilities Mayor Malware is exploiting make the hacker an easy scapegoat.

<p align="center"><i>
“It’s the Mayor” said the Glitch, he said it while sighing,<br>
“The people of Wareville, their browsing he’s spying!”<br>
“That sounds like him”, McSkidy then said,<br>
“Back to work then”, while scratching her head.
</i></p>

![Tryhackme Room Link](bell.svg){: width="1152" height="300" .shadow }
_<https://tryhackme.com/r/room/adventofcyber2024>_

## Learning Objectives

In today's task you will learn about:

- Self-signed certificates
- Man-in-the-middle attacks
- Using Burp Suite proxy to intercept traffic

## Certified to Sleigh

We hear a lot about certificates and their uses, but let’s start dissecting what a certificate is:

- **Public key**: At its core, a certificate contains a public key, part of a pair of cryptographic keys: a public key and a private key. The public key is made available to anyone and is used to encrypt data.
- **Private key**: The private key remains secret and is used by the website or server to decrypt the data.
- **Metadata**: Along with the key, it includes metadata that provides additional information about the certificate holder (the website) and the certificate. You usually find information about the Certificate Authority (CA), subject (information about the website, e.g. www.meow.thm), a uniquely identifiable number, validity period, signature, and hashing algorithm.

## Sign Here, Trust Me

So what is a Certificate Authority (CA)?

A CA is a trusted entity that issues certificates; for example, GlobalSign, Let’s Encrypt, and DigiCert are very common ones. The browser trusts these entities and performs a series of checks to ensure it is a trusted CA. Here is a breakdown of what happens with a certificate:

- **Handshake**: Your browser requests a secure connection, and the website responds by sending a certificate, but in this case, it only requires the public key and metadata.
- **Verification**: Your browser checks the certificate for its validity by checking if it was issued by a trusted CA. If the certificate hasn’t expired or been tampered with, and the CA is trusted, then the browser gives the green light. There are different types of checks you can do; check them [here](https://www.sectigo.com/resource-library/dv-ov-ev-ssl-certificates).
- **Key exchange**: The browser uses the public key to encrypt a session key, which encrypts all communications between the browser and the website.
- **Decryption**: The website (server) uses its private key to decrypt the session key, which is [symmetric](https://deviceauthority.com/symmetric-encryption-vs-asymmetric-encryption/). Now that both the browser and the website share a secret key (session key), we have established a secure and encrypted communication!

Ever wonder what makes HTTPS be S (secure)? Thanks to certificates, we can now have authentication, encryption, and data integrity.

**Self-Signed Certificates vs. Trusted CA Certificates**

The process of acquiring a certificate with a CA is long, you create the certificate, and send it to a CA to sign it for you. If you don’t have tools and automation in place, this process can take weeks. Self-signed certificates are signed by an entity usually the same one that authenticates. For example, Wareville owns the GiftScheduler site, and if they create a certificate and sign it with Wareville as a CA, that becomes a self-signed certificate.

- **Browsers** generally do not trust self-signed certificates because there is no third-party verification. The browser has no way of knowing if the certificate is authentic or if it’s being used for malicious purposes (like a **man-in-the-middle attack**).
- **Trusted CA certificates**, on the other hand, are verified by a CA, which acts as a trusted third party to confirm the website’s identity.

CA-issued certificates sometimes take a long time; if you want to test a development environment, it can make sense to use self-signed certificates. Ideally, this is an internal, air-gapped environment with no connection to the public Internet. Otherwise, it defeats the purpose of a certificate: the entire system of secure communication relies on the fact that both parties (the browser and the server) can trust the data being exchanged and that no one in the middle can intercept or modify it without detection.

## How Mayor Malware Disrupts G-Day

There are less than two weeks until G-Day, and Mayor Malware has been planning its disruption ever since Glitch raised the self-signed certificate vulnerability to McSkidy during a security briefing the other day.

His plan is near perfect. He will hack into the Gift Scheduler and mess with the delivery schedule. No one will receive the gift destined for them: G-Day will be ruined! [*evil laugh*]

**Preparation**

First things first: the Glitch spoke about a self-signed certificate, but Mayor Malware can’t believe that the townspeople—usually so security-savvy it’s maddening to him—would easily disregard such a critical vulnerability. Is it a trap set up by the Glitch and McSkidy to catch him red-handed? He definitely needs to check for himself.

Before that, though, he wants to make sure that his tracks are well covered. To prevent any DNS logs from alerting his enemies, he will resolve the Gift Scheduler’s FQDN locally on his machine.

To achieve this, let’s add the following line to the **/etc/hosts** file on the AttackBox: **MACHINE_IP gift-scheduler.thm**

We can use the following command:

```console
root@attackbox:~# echo "MACHINE_IP gift-scheduler.thm" >> /etc/hosts
```

To verify that the line above was added to the file, we can execute the following:

```console
root@attackbox:~# cat /etc/hosts
127.0.0.1       localhost
127.0.1.1       tryhackme.lan   tryhackme

# The following lines are desirable for IPv6 capable hosts
::1     localhost ip6-localhost ip6-loopback
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
MACHINE_IP gift-scheduler.thm
```
Now, Mayor Malware can navigate to the Gift Scheduler website without leaving a trace on Wareville’s DNS logs.

Let’s open the Firefox browser and navigate to **https://gift-scheduler.thm.** We’ll be presented with the following warning page:

![](warning1.png){: width="1244" height="598"}

We can click on the **Advanced** button to expand the warning’s details.

![](warning2.png){: width="1140" height="696"}

When we click on the **View Certificate** link marked with a 1 in the screenshot above, a new tab opens with the certificate details.

Mayor Malware can’t believe his luck! This is evidence that the Glitch was speaking the truth: the Gift Scheduler web server uses a self-signed certificate.

This means that the townspeople and all the elves will be used to clicking on the **Accept the Risk and Continue** button (marked with 2 on the screenshot above) to access the website, to the point it’s become a habit.

Mayor Malware does just that and inserts his credentials into the login form.

![](credential.png){: width="436" height="147"}

![](login.png){: width="1258" height="876"}

With his credentials, he can’t do anything but send a gift request—as if he were to ever do such a sickeningly sweet gesture. To carry out his evil plan, he will need to sniff some admin credentials. Maybe some of the elves’ passwords. Or even—if he gets lucky—Marta May Ware’s account!

![](wishlist.png){: width="1258" height="876"}

To sniff the elves’ traffic, the next step will be to start a proxy on his machine and route all of Wareville’s traffic to it. This way, the **Mayor** will be **In The Middle** between the townspeople and the Gift Scheduler. This position will allow him to sniff all requests forwarded to the sickening website.

Let’s start the Burp Suite proxy by typing **burp** in the terminal. A new window will open. We can accept the default configuration by clicking on **Next**, then **Start Burp** in the next window.

![](burp1.png){: width="1446" height="663"}

Once Burp Suite loads, we will select **Proxy** (number 1 in the screenshot above) and then toggle off the **Intercept on** option (number 2) to prevent users from noticing any delays in the website responses. Finally, let’s open the **Proxy Settings** (number 3) to set a new listener on our AttackBox IP address.

![](burp2.png){: width="1714" height="927"}

We can click on the **Add** button highlighted in the screenshot above. Burp Suite will prompt us for the new listener’s configuration.

![](burp3.png){: width="827" height="537"}

We must set the listening port to **8080** and toggle the **Specific address** option. The box next to it will automatically specify the IP address of our AttackBox, **CONNECTION_IP**. Finally, we can click on **OK** to apply the configuration.

The previous settings window will get displayed and we can see that the new listener has been added under the proxy listeners list.

![](burp4.png){: width="1714" height="927"}

Mayor Malware rubs his hands together gleefully: as we can read in the yellow box in the screenshot above, Burp Suite already comes with a self-signed certificate. The users will be prompted to accept it and continue, and Mayor Malware knows they will do it out of habit, without even thinking of verifying the certificate origin first. The G-Day disruption operation will go off without a hitch!

**Sniff From The Middle**

Now that our machine is ready to listen, we must reroute all Wareville traffic to our machine.

Mayor Malware has a wonderful idea to achieve this: he will set his own machine as a gateway for all other Wareville’s machines!

Let’s add another line to the AttackBox’s **/etc/hosts** file. Note: The **CONNECTION_IP** address in the snippet should reflect the IP of our AttackBox, which can be found at the top of the page.

```console
root@attackbox:~# echo "CONNECTION_IP wareville-gw" >> /etc/hosts
```

This will divert all of Wareville’s traffic, usually routed through the legitimate Wareville Gateway, to Mayor Malware’s machine, effectively putting him “In The Middle” of the requests. **Note**: In practice, the adversary can launch a similar attack if they can control the user’s gateway and their attack can easily succeed against websites not using properly signed certificates. This attack requires more than adding an entry into the **/etc/hosts** file; however, this task aims to emulate parts of the attack.

As a last step, we must start a custom script to simulab te the users’ requests to the Gift Scheduler. **Note**: Keep the script running so that new user requests will constantly be captured in Burp Suite.

```console
root@attackbox:~# cd ~/Rooms/AoC2024/Day14
root@attackbox:~/Rooms/AoC2024/Day14# ./route-elf-traffic.sh 
Verifying archive integrity...  100%   MD5 checksums are OK. All good.
Uncompressing Intercept Traffic  100%  
Intercepting user traffic in progress...
 User request intercepted successfully at 2024-12-11 16:05:56
 User request intercepted successfully at 2024-12-11 16:06:23
 User request intercepted successfully at 2024-12-11 16:06:36
[...]
```

**Pwn the Scheduler**

At last, everything is in place. Mayor Malware’s evil plan can finally commence! [*evil laugh*]

We can return to the open Burp Suite window and click on the **HTTP History** tab.

![](burp5.png){: width="1446" height="878"}

There is a triumphant gleam in Mayor Malware’s eyes while he stares intently at the web requests pouring on his screen. He can finally see them: the POST requests containing clear-text credentials for the Gift Scheduler website! Now, he only needs to wait and find the password to a privileged account.

![](aoc2024.png){: width="1920" height="1080"}

## Explanation

**To find the Certificate Authority (CA) in Firefox :**

- Click the padlock icon in the address bar of the site.
- Select 'Connection secure' or 'More information' (depending on your version).
- Click 'View Certificate.'
- In the certificate details, check the 'Issued by' section to identify the CA."

![](cert.png){: width="1216" height="647"}

[Download the route-elf-traffic.sh script](assets/scripts/route-elf-traffic.sh) 

With proxy listeners setup in the BurpSuite with your AttackBox IP / Local machine, execute the script and look into **Proxy > HTTP History** for credentials

![](route.png){: width="1003" height="680"}

## Answer

### Question 1

What is the name of the CA that has signed the Gift Scheduler certificate?

<details>
  <summary style="cursor:pointer; padding:10px; border:1px solid #ccc; background-color:#f0f0f0; user-select: none;">Answer</summary>
  <div style="padding:10px; border:1px solid #ccc;">
    <span onclick="navigator.clipboard.writeText('THM')" style="cursor:pointer;">THM</span>
    <i onclick="navigator.clipboard.writeText('THM')" style="float:right; cursor:pointer; font-size:16px;">&#x1F4C4;</i>
  </div>
</details>

### Question 2

Look inside the POST requests in the HTTP history. What is the password for the **snowballelf** account?

<details>
  <summary style="cursor:pointer; padding:10px; border:1px solid #ccc; background-color:#f0f0f0; user-select: none;">Answer</summary>
  <div style="padding:10px; border:1px solid #ccc;">
    <span onclick="navigator.clipboard.writeText('c4rrotn0s3')" style="cursor:pointer;">c4rrotn0s3</span>
    <i onclick="navigator.clipboard.writeText('c4rrotn0s3')" style="float:right; cursor:pointer; font-size:16px;">&#x1F4C4;</i>
  </div>
</details>

### Question 3

Use the credentials for any of the elves to authenticate to the Gift Scheduler website. What is the flag shown on the elves’ scheduling page?

<details>
  <summary style="cursor:pointer; padding:10px; border:1px solid #ccc; background-color:#f0f0f0; user-select: none;">Answer</summary>
  <div style="padding:10px; border:1px solid #ccc;">
    <span onclick="navigator.clipboard.writeText('THM{AoC-3lf0nth3Sh3lf}')" style="cursor:pointer;">THM{AoC-3lf0nth3Sh3lf}</span>
    <i onclick="navigator.clipboard.writeText('THM{AoC-3lf0nth3Sh3lf}')" style="float:right; cursor:pointer; font-size:16px;">&#x1F4C4;</i>
  </div>
</details>

### Question 4

What is the password for Marta May Ware’s account?

<details>
  <summary style="cursor:pointer; padding:10px; border:1px solid #ccc; background-color:#f0f0f0; user-select: none;">Answer</summary>
  <div style="padding:10px; border:1px solid #ccc;">
    <span onclick="navigator.clipboard.writeText('H0llyJ0llySOCMAS!')" style="cursor:pointer;">H0llyJ0llySOCMAS!</span>
    <i onclick="navigator.clipboard.writeText('H0llyJ0llySOCMAS!')" style="float:right; cursor:pointer; font-size:16px;">&#x1F4C4;</i>
  </div>
</details>

### Question 5

Mayor Malware finally succeeded in his evil intent: with Marta May Ware’s username and password, he can finally access the administrative console for the Gift Scheduler. G-Day is cancelled!
What is the flag shown on the admin page?

<details>
  <summary style="cursor:pointer; padding:10px; border:1px solid #ccc; background-color:#f0f0f0; user-select: none;">Answer</summary>
  <div style="padding:10px; border:1px solid #ccc;">
    <span onclick="navigator.clipboard.writeText('THM{AoC-h0wt0ru1nG1ftD4y}')" style="cursor:pointer;">THM{AoC-h0wt0ru1nG1ftD4y}</span>
    <i onclick="navigator.clipboard.writeText('THM{AoC-h0wt0ru1nG1ftD4y}')" style="float:right; cursor:pointer; font-size:16px;">&#x1F4C4;</i>
  </div>
</details>
