---
title: "TryHackMe: Advent Of Cyber 2025 Day 5 (IDOR - Santa’s Little IDOR)"
categories: [TryHackMe]
tags: [IDOR, advent of cyber]
render_with_liquid: false
media_subpath: /images/tryhackme_aoc2025_day5/
image:
  path: banner.png
---

Learn about IDOR while helping pentest the TrypresentMe website.

<h2 style="color:#A64AC9; text-align:center; font-weight:700; font-size:1.8em; text-shadow: 1px 0 #A64AC9, -1px 0 #A64AC9, 0 1px #A64AC9, 0 -1px #A64AC9;">
  The Story
</h2>

[![TryHackMe Room Link](header.svg){: width="1200" height="407" }](https://tryhackme.com/room/idor-aoc2025-zl6MywQid9)

## Introduction

The elves of Wareville are on high alert since McSkidy went missing. Recently, the support team has been receiving many calls from parents who can't activate vouchers on the TryPresentMe website. They also mentioned they are receiving many targeted phishing emails containing information that is not public. The support team is wary and has enlisted the help of the TBFC staff. When looking into this peculiar case, they discovered a suspiciously named account named Sir Carrotbane, which has many vouchers assigned to it. For now, they have deleted the account and retrieved the vouchers. But something is going on. Can you help the TBFC staff investigate the TryPresentMe website and fix the vulnerabilities?

## It’s Dangerously Obvious, Really

Have you ever seen a link that looks like this: `https://awesome.website.thm/TrackPackage?packageID=1001`?

When you saw a link like this, have you ever wondered what would happen if you simply changed the packageID to 11 or 12? In its simplest form, this can be a potential case for IDOR. 

IDOR stands for **Insecure Direct Object Reference** and is a type of access control vulnerability. Web applications often use references to determine what data to return when you make a request. However, if the web server doesn't perform checks to ensure you are allowed to view that data before sending it, it can lead to serious sensitive information disclosure. A good question to ask then is:

`Why does this happen so often?`

We need to understand references and web development a bit more to answer this. Let's take a look at what a table storing these package numbers from our link example could look like:

| packageID | person        | address                         | status           |
|-----------|---------------|---------------------------------|----------------|
| 1001      | Alice Smith   | 123 Main St, Springfield        | Delivered       |
| 1002      | Bob Johnson   | 42 Elm Ave, Shelbyville         | In Transit      |
| 1003      | Carol White   | 9 Oak Rd, Capital City          | Out for Delivery|
| 1004      | Daniel Brown  | 77 Pine St, Ogdenville          | Pending         |
| 1005      | Eve Martinez  | 5 Maple Ln, North Haverbrook    | Returned        |

If the user wants to know the status of their package and makes a web request, the simplest method is to allow the user to supply their packageID. We recover data from the database using the simplest SQL query of:

`SELECT person, address, status FROM Packages WHERE packageID = value;`

However, since packageID is a sequential number, it becomes pretty obvious to guess the packageIDs of other customers, and since the web application isn't verifying that the person making the request `is the same` person as the one who owns the package, an IDOR vulnerability appears, allowing attackers to recover the details for packages belonging to other users. Even worse is when a feature like this doesn't require a user to authenticate, then there would be no way to even tell who is making the request! To dive a bit deeper, we need to understand authentication, authorization, and privilege escalation.

A note from one of the co-authors of this task: I am not a fan of the vulnerability name <strong>IDOR</strong>. I prefer the name <strong>Authorization Bypass</strong>.

<details>
  <summary><strong>A note from one of the co-authors of this task:&nbsp;</strong>
    I am not a fan of the vulnerability name <span data-testid="glossary-term" class="glossary-term">IDOR</span>. I prefer the name Authorization Bypass. If you want to understand my reasoning, expand here, but you don't have to be bored with the details!
  </summary>
  <h2>It Doesn’t Obviously Relate</h2>
  <p>The full term, Insecure Direct Object Reference, sounds fancy, but it doesn’t really describe what’s going wrong. The “Direct Object Reference” part just means that a system uses an ID (like <code>/user/1</code>) to point to something. That’s not the problem. The real issue is that the system doesn’t check whether the person making the request is allowed to access it.</p>
  <p>A lot of people try to “fix” IDORs by hiding or encoding <span data-testid="glossary-term" class="glossary-term">IDs</span>. For example, changing <code>/user/1</code> to <code>/user/ea21f09b2</code>. That might make it look harder to guess, but if the server still isn’t checking permissions, it’s just as insecure. The vulnerability isn’t about how the object is referenced, it’s about missing authorization checks.</p>
  <p>That’s why I prefer to call it an Authorization Bypass instead. It explains exactly what’s happening: someone is bypassing the rules that decide who can see or change something. Whether the ID is a number, a hash, or a random string, the risk stays the same if the server doesn’t verify access. You can read more <a href="https://www.mwrcybersec.com/whats-the-deal-with-idor" target="_blank" rel="noopener noreferrer">here</a> if you want.</p>
</details>

## Identity Defines Our Reach

To understand the root cause of IDOR, it is important to understand the basic principles of authentication and authorization:

- **Authentication**: The process by which you verify who you are. For example, supplying your username and password.  
- **Authorization**: The process by which the web application verifies your permissions. For example, are you allowed to visit the admin page of a web application, or are you allowed to make a payment using a specific account?

You may think that authentication only happens once when you supply your username and password, but that is actually not the case! After providing your credentials, you receive a cookie or a token, called session information. Every subsequent request you make to the application includes this session information, which is verified by the application. This initial verification process is still authentication and happens for each request. This is why websites will often redirect you back to the login page. It means your session information has expired, and thus, you need to reauthenticate with your credentials to receive new session information.

Authorization cannot happen before authentication. If the application doesn't know who you are, it cannot verify what permissions your user has. This is very important to remember. If your IDOR doesn't require you to authenticate (login or provide session information), such as in our package tracking example, we will have to fix authentication first before we can fix the authorization issue of making sure that users can only get information about packages they own.

The last bit of theory to cover is privilege escalation types:

- **Vertical privilege escalation**: This refers to privilege escalation where you gain access to more features. For example, you may be a normal user on the application, but can perform actions that should be restricted for an administrator.

- **Horizontal privilege escalation**: This refers to privilege escalation where you use a feature you are authorized to use, but gain access to data that you are not allowed to access. For example, you should only be able to see your accounts, not someone else's accounts.

IDOR is usually a form of horizontal privilege escalation. You are allowed to make use of the track package functionality. But you should be restricted to only performing that tracking action for packages you own. Now that we understand the theory, let's look at how to exploit IDOR practically!

## Iterate Digits, Observe Responses

Let's start with the simplest example of IDOR. On the web application, let's authenticate to the application using the details below.

![](credentials.png){: width="1203" height="126"}

Once authenticated, you should see a dashboard like this:
 
![](dashboard.png){: width="1248" height="958"}

Let's start by using the `Developer Tools` of our browser to better understand what is happening in the background. Right-click on the page and click `Inspect`, then click on the `Network` tab as shown below:

![](network.png){: width="1439" height="191"}

Now let's refresh the page and see what requests are being made. It should look something like this:

![](network2.png){: width="1440" height="303"}

Let's take a closer look at the `view_accountinfo` request. Click on it and you will see the following:

![](headers.png){: width="688" height="752"}

In the above image we can see that the `user_id` with the value of 10 was used for the request. If we click and expand the Response tab, we can see that this `user_id` corresponds to our user:

![](response.png){: width="459" height="359"}

This tells us that the application is using our `user_id` as the reference for getting details. Let's see what happens when we change this. In the Developer Tools, navigate to the `Storage` tab and expand the `Local Storage` dropdown on the left side and click the URL inside it:

![](storage.png){: width="1437" height="219"}

Let's change the `user_id` to 11 and see what happens. Double-click on the Value field of the `auth_user` data entry, update the `user_id` to 11 and save it by pressing **Enter**. Now refresh the page. All of a sudden it seems like you are a completely different user!

This is the simplest form of IDOR. Simply changing the `user_id` to something else means we can see other users' data. Some IDORs might be slightly more hidden. Just because you don't see a direct number doesn't mean it doesn't exist! Let's dive deeper. To continue onto the next challenges of the task, **go and change the id back to 10 using the same steps you followed above**. Alternatively, you can log out of the application and log back in using the username and password.

## In Disguise: Obvious References

Sometimes, IDOR may not be as simple as just a number. In certain cases, encoding may have been used. On the loaded profile, click the eye icon next to the first child as shown on the image below.

![](eye.png){: width="1386" height="583"}

Now go back to the **Network** tab and take a look at the requests being made; you should see a request like this:

![](network3.png){: width="1440" height="495"}

Simply put, the `Mg==` is just the base64 encoded version of the number `2`. You could still perform IDOR using this request, but you would have to base64 encode the number first.

## In Digests, Objects Remain

Encoding isn't the only thing that can be used to hide potential IDOR vulnerabilities. Sometimes the values may look like a hash, such as MD5 or SHA1. If you want to see what that would look like, take a look at the request that happens when you click the edit icon next to a child.

![](network4.png){: width="1437" height="469"}

While the string may look random, upon further investigation, you can see that it is a type of hash. If we understand what value was used to generate the hash, we can perform an IDOR attack by simply replicating the hashing function. Using something like a [hash identifier](https://hashes.com/en/tools/hash_identifier) can help you quickly understand what hashing algorithm is being used and can often tell you what data was hashed.

## It's Deterministic, Obviously Reproducible

Sometimes you have to dig quite deep for IDOR. Sometimes IDOR is not as clear. Sometimes the IDOR stems from the actual algorithm being used. In this last case, let's take a look at our vouchers. While the values may look random, we need to investigate what algorithm was used to generate them. Their format looks like a UUID, so let's use a website such as [UUID Decoder](https://www.uuidtools.com/decode) to try to understand what UUID format was used. Copy one of the vouchers to the website for decoding, and you should see something like this:

![](uuid.png){: width="1144" height="686"}

While these look completely random, we can see that the UUID version 1 was used. The issue with UUID 1 is that if we know the exact date when the code was generated, we can recover the UUID. For example, suppose we knew the elves always generated vouchers between 20:00 - 21:00. In that case, we can create UUIDs for that entire time period (3600 UUIDs since we have 60 minutes, and 60 seconds in a minute), which we could use in a brute force attack to aim to recover a valid voucher and get more gifts.

Now that we have seen the various IDORs that can be found, let's discuss how to fix them and avoid them!

## Improve Design, Obliterate Risk

Now that we learned about what IDOR is, let's discuss how to fix it. The best way to stop IDOR is to make sure the server checks who is asking for the data every time. It's not enough to hide or change the ID number; the system must confirm that the logged-in user is authorized to see or change that information.

Don't rely on tricks like Base64 or hashing the IDs; those can still be guessed or decoded. Instead, keep all the real permission checks on the server. Whenever a request comes in, check: *"Does this user own or have permission to view this item?"*

Use random or hard-to-guess IDs for public links, but remember that random IDs alone don't make your app safe. Always test your app by trying to open another user's data and making sure it's blocked. Finally, record and monitor failed access attempts; they can be early signs of someone trying to exploit an IDOR.

## Answer

### Question 1

What does IDOR stand for?

<details>
  <summary style="cursor:pointer; padding:10px; border:1px solid #888; background-color:#444; color:#fff; user-select: none;">
    Answer
  </summary>
  <div style="padding:10px; border:1px solid #888; background-color:#333; color:#fff;">
    <span onclick="navigator.clipboard.writeText('Insecure Direct Object Reference')" style="cursor:pointer;">Insecure Direct Object Reference</span>
    <i onclick="navigator.clipboard.writeText('Insecure Direct Object Reference')" style="float:right; cursor:pointer; font-size:16px;">&#x1F4C4;</i>
  </div>
</details>

### Question 2

What type of privilege escalation are most IDOR cases?

<details>
  <summary style="cursor:pointer; padding:10px; border:1px solid #888; background-color:#444; color:#fff; user-select: none;">
    Answer
  </summary>
  <div style="padding:10px; border:1px solid #888; background-color:#333; color:#fff;">
    <span onclick="navigator.clipboard.writeText('Horizontal')" style="cursor:pointer;">Horizontal</span>
    <i onclick="navigator.clipboard.writeText('Horizontal')" style="float:right; cursor:pointer; font-size:16px;">&#x1F4C4;</i>
  </div>
</details>

### Question 3

Exploiting the IDOR found in the `view_accounts` parameter, what is the `user_id` of the parent that has 10 children?

<details>
  <summary style="cursor:pointer; padding:10px; border:1px solid #888; background-color:#444; color:#fff; user-select: none;">
    Answer
  </summary>
  <div style="padding:10px; border:1px solid #888; background-color:#333; color:#fff;">
    <span onclick="navigator.clipboard.writeText('15')" style="cursor:pointer;">15</span>
    <i onclick="navigator.clipboard.writeText('15')" style="float:right; cursor:pointer; font-size:16px;">&#x1F4C4;</i>
  </div>
</details>





