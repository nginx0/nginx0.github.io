---
title: "TryHackMe: Advent Of Cyber 2025 Day 17 (CyberChef - Hoperation Save McSkidy)"
categories: [TryHackMe]
tags: [base64, cryptography, advent of cyber]
render_with_liquid: false
media_subpath: /images/tryhackme_aoc2025_day17/
image:
  path: banner.png
---

The story continues, and the elves mount a rescue and will try to breach the Quantum Fortress's defenses and free McSkidy.

<h2 style="color:#A64AC9; text-align:center; font-weight:700; font-size:1.8em; text-shadow: 1px 0 #A64AC9, -1px 0 #A64AC9, 0 1px #A64AC9, 0 -1px #A64AC9;">
  The Story
</h2>

[![TryHackMe Room Link](header.png){: width="1200" height="407" }](https://tryhackme.com/room/encoding-decoding-aoc2025-s1a4z7x0c3)

## Introduction

*McSkidy is imprisoned in King Malhare's Quantum Warren. Sir BreachBlocker III was put in charge of securing the fortress and implemented several access controls to prevent any escape. His defenses are worthy of his name.*

*However, McSkidy managed to send vital clues to his team using harmless bunny pictures. One message revealed that five locks needed to be disabled to secure an escape route. The locks can be broken by examining their logic and leveraging the system's built-in chat for the guards. They can be eluded in revealing vital details or even passwords. However, you will need to speak their language.*

## Encoding and Decoding

Encoding is a method to transform data to ensure compatibility between different systems. It differs from encryption in purpose and process.

| Aspect       | Encoding                | Encryption         |
|-------------|------------------------|------------------|
| **Purpose** | Compatibility, Usability | Security, Confidentiality |
| **Process** | Standardized           | Algorithm + Key  |
| **Security**| No                     | Yes              |
| **Speed**   | Fast                   | Slow             |
| **Examples**| Base64                 | TLS              |

Decoding is the process of converting encoded data back to its original, readable, and usable form.

## CyberChef Overview

[CyberChef](https://cyberchef.io/) is also known as the Cyber Swiss Army Knife. Ready to cook some recipes?

| Area      | Description                                 |
|-----------|---------------------------------------------|
| **Operations** | Repository of diverse CyberChef capabilities |
| **Recipe**     | Fine-tune and chain the operations area     |
| **Input**      | Here you provide the input for your recipe |
| **Output**     | Here is the output of your recipe          |

## Simple Example

Try your first recipe:

- Open either the online [CyberChef](https://cyberchef.io/) version in your regular browser, or use the offline CyberChef version available in the bookmarks section of the AttackBox. Drag and drop the `To Base64` operation from the **Operations** area on the left side to the **Recipe** area in the center, and add `IamRoot` into the **Input** area.

- Add another operation, `From Base64`, to show the initial input again, showcasing chain operations.

**Note:** You can enable/disable an operation in the recipe by toggling the middle button on the right of the operation.

![](cyberchef.png){: width="1080" height="1080"}

Congratulations! You took the first steps to become a master Chef.

## Inspecting Web Pages

Besides the rendered content of a web page, your browser usually receives and can show additional information.

For this challenge, you will get the chance to have a deeper look at that information and put it to good use.

To do this, depending on your browser, you can access the functionality as shown below:

| Browser         | Menu Path                                                                                  |
|-----------------|--------------------------------------------------------------------------------------------|
| **Chrome**      | `More tools > Developer tools`                                                             |
| **Firefox**     | `Menu (☰) > More tools > Web Developer Tools`                                             |
| **Microsoft Edge** | `Settings and more (...) > More tools > Developer tools`                               |
| **Opera**       | `Developer > Developer tools`                                                              |
| **Safari**      | `Develop > Show Web Inspector` (Requires enabling the "Develop" menu in Preferences > Advanced) |

**Note:** For a better experience, you can reposition the console on the right side of the browser. Look for the three dots on the right side of the console.

![](dot.png){: width="240" height="274"}

## Key Information

If not already, start the target machine, give it a few minutes to boot up, and then, from the AttackBox, you can access the web app at `http://MACHINE_IP:8080`.

McSkidy revealed some vital clues in his message. You will have to leverage any useful piece of information in order to break the locks.

Below are key points to look out for:

- **Chat is Base64 encoded**. Try decoding this in CyberChef. This will be leveraged to extract useful information from the guards. Be aware that from Lock 3 onwards, the guards will take a longer time to respond.

![](bunnygram.png){: width="420" height="137"}

- **Guard name**. This logic will persist throughout the levels. Make sure to note down the guard’s name for each level.

![](outergate.png){: width="433" height="534"}

- **Headers**. Again, inspecting the page but switching to the ‘Network’ tab this time. Make sure to refresh the page once after switching to this tab and select the first response.

![](network.png){: width="847" height="825"}

- **Login Logic**. You will inspect the page and switch to the ‘Debugger’ tab. Match the lock with the respective logic. You can also find helpful comments that explain what you need to cook in CyberChef.

![](debugger.png){: width="925" height="942"}

## First Lock - Outer Gate

![](firstlock.png){: width="256" height="256"}

Ok, it’s time to siege the fortress. Ready?

1. First, identify the guard name and encode it to Base64. You will use this as the username input.
2. Next, using the information from the page headers, identify the magic question and encode it in Base64 as well.

![](outer_gate.png){: width="1781" height="839"}

3. Use the encode magic question in the chat. The guard will answer with the encoded level password.

4. Now, switch to the ‘Debugger’ tab and identify the login logic. In this case, the password is encoded to Base 64.

![](outer_gate2.png){: width="1783" height="769"}

5. By decoding the answer from the guard, you will have the plaintext password.

6. Use the encoded username and plaintext password to log in.

Excellent work! One lock is down, and only four remain to be broken.

## Second Lock - Outer Wall

![](secondlock.png){: width="256" height="256"}

Excellent job breaking that first level.

This level nudges the difficulty up a little bit, but don’t worry, you will figure it out. Let’s go!

1. Again, identify the guard's name and save the encoded output for later.
2. Then, extract and encode the magic question and retrieve the encoded password from the guard.

![](outer_wall.png){: width="1789" height="803"}

3. Looking again at the login logic, you see that the encoding is applied twice this time. That means you have to decode from Base64 twice.

4. Go ahead and log in with the newfound password and the saved username.

![](outer_wall2.png){: width="1788" height="765"}

You are getting closer to securing an escape route; only three locks remain. Keep up the good work.

## Third Lock - Guard House

![](thirdlock.png){: width="256" height="256"}

So far, so good. As you saw in the previous level, the login logic begins to use chained operations.

This will be the trend for this and the following levels.

1. As always, collect all the needed information (encoded username, encoded password from the guard, XOR key).

![](guard_house.png){: width="1799" height="825"}

**Note:** From this lock onwards, there is no magic question, but sometimes you can ask the guard nicely to give you the password. It will still need to be decoded as per the login logic. Be aware that the guard may sometimes fall asleep or take a long time to respond (~2-3 minutes) so keeping the message short will help get the answer. Even a simple 'Password please.' will go a long way.

2. If you look at the login logic, there is a slight twist. The password is first XOR’ed with a key and then encoded to Base64.

**Theory Time**

XOR is a popular operation that, besides the input data, also uses a key. The process involves a bitwise exclusive OR between the data and key.

![](xor.png){: width="512" height="512"}

You might ask, *“Ok, but how do I reverse this?”*. Well, skipping the long math explanation, XOR has a magic property: when you XOR the result with the key again, the new result will be the initial data. Go ahead, try this in CyberChef. Put two XOR operations one after another, use the same key for both, and the output should be identical.

![](xor_decode.png){: width="709" height="571"}

3. With this newfound knowledge, build the needed recipe to find the plaintext password.

![](guard_house2.png){: width="1785" height="770"}

![](base64.png){: width="1462" height="1200"}

4. Use the credentials and unlock the next level.

## Fourth Lock - Inner Castle

![](fourthlock.png){: width="256" height="256"}

We are almost there. In this level, Sir BreachBlocker III throws you a curveball. Let’s see how to tackle this.

1. But first, go ahead and look at the login logic as before. We will not be needing header information for this one.

![](inner_castle.png){: width="1781" height="787"}

2. After asking the guard for the password and looking at it's reply, it seems a bit odd. At the same time, the login logic shows the use of a MD5 hash.

![](pass.png){: width="1238" height="551"}

**Theory Time**

MD5, or Message-Digest Algorithm 5, is a cryptographic algorithm that produces a fixed-size hash value. While this is supposed to be a one-way function, meaning you cannot reverse it, precomputed hashes can be leveraged to identify the input.

3. Putting the two together, the plaintext password is passed through MD5, and you have the hash. This looks like a job for [CrackStation](https://crackstation.net/).

4. Go ahead and open the site and paste the hash to retrieve the password.

![](crackstation.png){: width="1023" height="386"}

5. Use the credentials and advance to the final level.

Fantastic. One more lock and you will ensure McSkidy has safe passage and escapes.

## Fifth Lock - Prison Tower

![](fifthlock.png){: width="256" height="256"}

Ready for the final hurdle?

As the defenses weaken, you receive another hidden message from McSkidy:

*“I can see you are ready to break the last lock. Be aware that Sir BreachBlocker III implemented different mechanisms for the last lock, which change occasionally. Make sure you match the correct approach when decoding the password.”*

That sounds tricky, but do not despair. You will find a way.

1. Let’s start. Extract the information as before, noting down the encoded guard name.

![](prison_tower.png){: width="1786" height="740"}

2. Additionally, note the recipe ID from the header and match the corresponding login logic. Below is a quick cheat sheet for decoding each recipe.

| Recipe ID | Reverse Logic                                  |
|-----------|------------------------------------------------|
| 1         | From Base64 ⇒ Reverse ⇒ ROT13                  |
| 2         | From Base64 ⇒ From Hex ⇒ Reverse               |
| 3         | ROT13 ⇒ From Base64 ⇒ XOR (extracted key)      |
| 4         | ROT13 ⇒ From Base64 ⇒ ROT47                    |

3. Build the reverse recipe with CyberChef and extract the final password.

Finally, the last lock has been breached, and you provided a safe path for McSkidy to escape.

*As McSkidy passed by the Inner Castle, she heard a thunderous voice: “Why should Christmas have all the fun?”*

*McSkidy managed to get back to Wareville just in time as TBFC was about to be hit by another disaster.*

![](mcskidy.png){: width="800" height="480"}

## Lock 1: Outer Gate 

1. Locate the **guard’s name** on **Bunnygram**:  

2. **Base64-encode** the guard’s name using [CyberChef](https://gchq.github.io/CyberChef/).  
   - This encoded value will be used as the **Outer Gate username**.

3. Open **Developer Tools** (`F12`) and navigate to the **Network** tab.
4. Refresh the page and select the **first HTTP request**.
5. Inspect the **Response Headers** to find the **magic question**:

   > **What is the password for this level?**

6. **Base64-encode** the magic question.
7. Send the encoded question through **Bunnygram**  
   *(the Bunnygram chat protocol uses Base64)*.

8. The guard responds with a **Base64-encoded password**.
9. Open [CyberChef](https://gchq.github.io/CyberChef/) and apply the **From Base64** operation.
10. Paste the guard’s response to decode it.

**The decoded result is the plaintext password for this level.**

## Lock 2: Outer Wall

This stage builds on the previous lock, but introduces an extra layer of encoding. Instead of a single Base64 operation, the credentials are encoded **twice**.

Follow these steps carefully:

1.   Identify the **guard’s name** on **Bunnygram**
   - Encode the name using **Base64**
   - Use the encoded value as the **username**

2. **Inspect the request headers**
   - Look for the predefined security question:
     - *“Did you change the password?”*

3. **Encode the question**
   - Convert the question to **Base64**
   - Send it through the chat interface

4. **Receive the response**
   - The guard replies with an **encoded string**

5. **Analyze the client-side logic**
   - Open the **Debugger** tab in the browser
   - Review the JavaScript handling the login process
   - You’ll notice the password is validated after being **Base64-encoded twice**

**Recovering the Password**

To extract the real password:

1. Take the encoded response from the guard
2. Open **CyberChef**
3. Build the following recipe:
   - `From Base64`
   - `From Base64`
4. Run the recipe to reveal the **plaintext password**

Log in with the same username pattern **(guard name in Base64)** + this password.

> This lock reinforces the importance of checking client-side logic—multiple layers of encoding can often be reversed if no additional protection is applied.
{: .prompt-info }

## Lock 3: Guard House

Starting with this lock, the challenge begins to resemble **real-world authentication logic**, where multiple transformations are applied in sequence rather than a single encoding step.

The guard’s chat responses are still **Base64-encoded**, but the login verification process now follows this flow:

1. Take the user-supplied password
2. Apply an **XOR operation** using a fixed key
3. **Base64-encode** the result
4. Compare it against the stored value

The XOR key is conveniently provided in the UI:

```console
cyberchef
```

Unlike earlier locks, there’s no longer a special “magic question” header. Instead, you simply ask the guard something straightforward, such as: `Can I have the password?`

In return we get a message (Encoded in **Base64**)

The guard replies with an encoded string that represents the **stored, transformed password**.

XOR has a useful reversible property:

- If `A XOR KEY = B`
- Then `B XOR KEY = A`

This means the original transformation:

**Plaintext → XOR (key) → Base64**

Can be reversed by applying the steps in the opposite order:

**Encoded string → From Base64 → XOR (same key) → Plaintext**

**CyberChef Recipe (Reverse Logic)**

To recover the password:

1. Copy the guard’s encoded reply
2. Open **CyberChef**
3. Create the following recipe:
   - **From Base64**
   - **XOR**
     - Key: `cyberchef`
4. Paste the encoded string as input and run the recipe

CyberChef will output the original plaintext password.

## Lock 4: Inner Castle

This stage changes the approach once again, shifting from reversible transformations to **hash-based validation**.

After requesting the password from the guard (still via **Base64-encoded chat**), the response no longer resembles readable text. Instead, it appears to be a **hash value**.

Looking at the client-side JavaScript confirms what’s happening:
- The login logic applies an **MD5 checksum** before comparison.

MD5 is a **one-way hashing algorithm** that produces a fixed-length output. Unlike encoding or XOR operations, hashes cannot be reversed directly.

The authentication process works like this:

1. The original password is hashed using **MD5**
2. The resulting hash is **Base64-encoded**
3. That encoded hash is stored or transmitted by the system

While you can’t decode MD5, you *can* identify weak or common passwords using precomputed databases.

Follow these steps:

1. Take the guard’s chat response
2. Decode it using **CyberChef**
   - Operation: **From Base64**
3. The output will be an **MD5 hash**
4. Paste the hash into an online cracking service such as [CrackStation](https://crackstation.net/)
5. If the hash exists in their dataset, the original password will be revealed

## Lock 5: Prison Tower

The final lock is the most interesting one, as it introduces **dynamic decoding logic** instead of a fixed transformation chain.

McSkidy explains that **Sir BreachBlocker III** added variability to the final authentication process. The correct decoding method depends on a **Recipe ID** found in the HTTP headers.

The general pattern remains familiar, but with an added decision step:

1. **Retrieve the guard’s name**
   - Encode it using **Base64**
   - Use the result as the **username**

2. **Request the password**
   - Ask the guard for the password via chat
   - The message must be **Base64-encoded**

3. **Inspect HTTP headers**
   - Look for a **Recipe ID** value

4. **Select the correct reverse recipe**
   - The Recipe ID determines which decoding pipeline to use

5. **Decode the response**
   - Apply the matching transformation sequence in **CyberChef**
   - The final output will be the plaintext password

**Recipe ID Cheat Sheet**

Use the following table to determine the correct reverse logic:

| Recipe ID | Reverse Logic (CyberChef) |
|----------|---------------------------|
| `1` | From Base64 → Reverse → ROT13 |
| `2` | From Base64 → From Hex → Reverse |
| `3` | ROT13 → From Base64 → XOR (using extracted key) |
| `4` | ROT13 → From Base64 → ROT47 |

Paste the guard’s encoded response into CyberChef, apply the recipe, and the final output will reveal the password. After identifying the correct recipe and reversing it successfully,log in with the credential completes the final lock thus retrieving the flag.

## Answer

### Question 1

What is the password for the first lock?

<details>
  <summary style="cursor:pointer; padding:10px; border:1px solid #888; background-color:#444; color:#fff; user-select: none;">
    Answer
  </summary>
  <div style="padding:10px; border:1px solid #888; background-color:#333; color:#fff;">
    <span onclick="navigator.clipboard.writeText('Iamsofluffy')" style="cursor:pointer;">Iamsofluffy</span>
    <i onclick="navigator.clipboard.writeText('Iamsofluffy')" style="float:right; cursor:pointer; font-size:16px;">&#x1F4C4;</i>
  </div>
</details>

### Question 2

What is the password for the second lock?

<details>
  <summary style="cursor:pointer; padding:10px; border:1px solid #888; background-color:#444; color:#fff; user-select: none;">
    Answer
  </summary>
  <div style="padding:10px; border:1px solid #888; background-color:#333; color:#fff;">
    <span onclick="navigator.clipboard.writeText('Itoldyoutochangeit!')" style="cursor:pointer;">Itoldyoutochangeit!</span>
    <i onclick="navigator.clipboard.writeText('Itoldyoutochangeit!')" style="float:right; cursor:pointer; font-size:16px;">&#x1F4C4;</i>
  </div>
</details>

### Question 3

What is the password for the third lock?

<details>
  <summary style="cursor:pointer; padding:10px; border:1px solid #888; background-color:#444; color:#fff; user-select: none;">
    Answer
  </summary>
  <div style="padding:10px; border:1px solid #888; background-color:#333; color:#fff;">
    <span onclick="navigator.clipboard.writeText('BugsBunny')" style="cursor:pointer;">BugsBunny</span>
    <i onclick="navigator.clipboard.writeText('BugsBunny')" style="float:right; cursor:pointer; font-size:16px;">&#x1F4C4;</i>
  </div>
</details>

### Question 4

What is the password for the fourth lock?

<details>
  <summary style="cursor:pointer; padding:10px; border:1px solid #888; background-color:#444; color:#fff; user-select: none;">
    Answer
  </summary>
  <div style="padding:10px; border:1px solid #888; background-color:#333; color:#fff;">
    <span onclick="navigator.clipboard.writeText('passw0rd1')" style="cursor:pointer;">passw0rd1</span>
    <i onclick="navigator.clipboard.writeText('passw0rd1')" style="float:right; cursor:pointer; font-size:16px;">&#x1F4C4;</i>
  </div>
</details>

### Question 5

What is the password for the fifth lock?

<details>
  <summary style="cursor:pointer; padding:10px; border:1px solid #888; background-color:#444; color:#fff; user-select: none;">
    Answer
  </summary>
  <div style="padding:10px; border:1px solid #888; background-color:#333; color:#fff;">
    <span onclick="navigator.clipboard.writeText('51rBr34chBl0ck3r')" style="cursor:pointer;">51rBr34chBl0ck3r</span>
    <i onclick="navigator.clipboard.writeText('51rBr34chBl0ck3r')" style="float:right; cursor:pointer; font-size:16px;">&#x1F4C4;</i>
  </div>
</details>

### Question 6

What is the retrieved flag?

<details>
  <summary style="cursor:pointer; padding:10px; border:1px solid #888; background-color:#444; color:#fff; user-select: none;">
    Answer
  </summary>
  <div style="padding:10px; border:1px solid #888; background-color:#333; color:#fff;">
    <span onclick="navigator.clipboard.writeText('THM{M3D13V4L_D3C0D3R_4D3P7}')" style="cursor:pointer;">THM{M3D13V4L_D3C0D3R_4D3P7}</span>
    <i onclick="navigator.clipboard.writeText('THM{M3D13V4L_D3C0D3R_4D3P7}')" style="float:right; cursor:pointer; font-size:16px;">&#x1F4C4;</i>
  </div>
</details>