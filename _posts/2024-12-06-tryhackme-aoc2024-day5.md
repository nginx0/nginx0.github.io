---
title: "TryHackMe: Advent Of Cyber 2024 Day 5"
categories: [TryHackMe]
tags: [xxe, xml, advent of cyber]
render_with_liquid: false
media_subpath: /images/tryhackme_aoc2024_day5/
image:
  path: banner.png
---

The days in Wareville flew by, and Software's projects were nearly complete, just in time for Christmas. One evening, after wrapping up work, Software was strolling through the town when he came across a young boy looking dejected. Curious, Software asked, "What would you like for Christmas?" The boy replied with a sigh, "I wish for a teddy bear, but I know that my family can't afford one."

This brief conversation sparked an idea in Software's mind—what if there was a platform where everyone in town could share their Christmas wishes, and the Mayor's office could help make them come true? Excited by the potential, Software introduced the idea to Mayor Malware, who embraced it immediately. The Mayor encouraged the team to build the platform for the people of Wareville.

Through the developers' dedication and effort, the platform was soon ready and became an instant hit. The townspeople loved it! However, in their rush to meet the holiday deadline, the team had overlooked something critical—thorough security testing. Even Mayor Malware had chipped in to help develop a feature in the final hours. Now, it's up to you to ensure the application is secure and free of vulnerabilities. Can you guarantee the platform runs safely for the people of Wareville?

![Tryhackme Room Link](bell.png){: width="1152" height="300" .shadow }
_<https://tryhackme.com/r/room/adventofcyber2024>_

## Learning Objectives

- Understand the basic concepts related to XML
- Explore XML External Entity (XXE) and its components
- Learn how to exploit the vulnerability
- Understand remediation measures

## Extensible Markup Language (XML)

XML is a commonly used method to transport and store data in a structured format that humans and machines can easily understand. Consider a scenario where two computers need to communicate and share data. Both devices need to agree on a common format for exchanging information. This agreement (format) is known as **XML**. You can think of XML as a digital filing cabinet. Just as a filing cabinet has folders with labelled documents inside, XML uses **tags** to label and organise information. These tags are like folders that define the type of data stored. This is what an XML looks like, a simple piece of text information organised in a structured manner:

![XML act like a file drawer](drawer.png){: width="740" height="850"}

```xml
<people>
   <name>Glitch</name>
   <address>Wareville</address>
   <email>glitch@wareville.com</email>
   <phone>111000</phone>
</people>
```

In this case, the tags **<people>, <name>, <address>**, etc are like folders in a filing cabinet, but now they store data about Glitch. The content inside the tags, like "**Glitch**," "**Wareville**," and "**123-4567**" represents the actual data being stored. Like before, the key benefit of XML is that it is easily shareable and customisable, allowing you to create your own tags. 

### Document Type Definition (DTD)

Now that the two computers have agreed to share data in a common format, what about the structure of the format? Here is when the DTD comes into play. A DTD is a set of rules that defines the structure of an XML document. Just like a database scheme, it acts like a blueprint, telling you what elements (tags) and attributes are allowed in the XML file. Think of it as a guideline that ensures the XML document follows a specific structure.

For example, if we want to ensure that an XML document about people will always include a **name, address, email, and phone number**, we would define those rules through a DTD as shown below:

```xml
Document Type Definition (DTD)

Now that the two computers have agreed to share data in a common format, what about the structure of the format? Here is when the DTD comes into play. A DTD is a set of rules that defines the structure of an XML document. Just like a database scheme, it acts like a blueprint, telling you what elements (tags) and attributes are allowed in the XML file. Think of it as a guideline that ensures the XML document follows a specific structure.

For example, if we want to ensure that an XML document about people will always include a name, address, email, and phone number, we would define those rules through a DTD as shown below:
```

In the above DTD, **<!ELEMENT>** ** defines the elements (tags) that are allowed, like name, address, email, and phone, whereas **#PCDATA** stands for parsed people data, meaning it will consist of just plain text.

### Entities

So far, both computers have agreed on the format, the structure of data, and the type of data they will share. Entities in XML are placeholders that allow the insertion of large chunks of data or referencing internal or external files. They assist in making the XML file easy to manage, especially when the same data is repeated multiple times. Entities can be defined internally within the XML document or externally, referencing data from an outside source. 

For example, an external entity references data from an external file or resource. In the following code, the entity **&ext**; could refer to an external file located at **"http://tryhackme.com/robots.txt"**, which would be loaded into the XML, if allowed by the system:

```xml
<!DOCTYPE people [
   <!ENTITY ext SYSTEM "http://tryhackme.com/robots.txt">
]>
<people>
   <name>Glitch</name>
   <address>&ext;</address>
   <email>glitch@wareville.com</email>
   <phone>111000</phone>
</people>
```

We are specifically discussing external entities because it is one of the main reasons that XXE is introduced if it is not properly managed.

### XML External Entity (XXE)

After understanding XML and how entities work, we can now explore the XXE vulnerability. XXE is an attack that takes advantage of **how XML parsers handle external entities**. When a web application processes an XML file that contains an external entity, the parser attempts to load or execute whatever resource the entity points to. If necessary sanitisation is not in place, the attacker may point the entity to any malicious source/code causing the undesired behaviour of the web app.

For example, if a vulnerable XML parser processes this external entity definition:

```xml
<!DOCTYPE people[
   <!ENTITY thmFile SYSTEM "file:///etc/passwd">
]>
<people>
   <name>Glitch</name>
   <address>&thmFile;</address>
   <email>glitch@wareville.com</email>
   <phone>111000</phone>
</people>
```

Here, the entity **&thmFile**; refers to the sensitive file /etc/passwd on a system. When the XML is processed, the parser will try to load and display the contents of that file, exposing sensitive information to the attacker.

## Practical 

Now that you understand the basic concepts related to XML and XXE, we will analyse an application that allows users to view and add products to their carts and perform the checkout activity. You can access the Wareville application hosted on http://MACHINE_IP. This application allows users to request their Christmas wishes.

**Flow of the Application**

As a penetration tester, it is important to first analyse the flow of the application. First, the user will browse through the products and add items of interest to their wishlist at **http://MACHINE_IP/product.php**. Click on the **Add to Wishlist** under **Wareville's Jolly Cap**, as shown below:

![](productlist.png){: width="1919" height="712"}

After adding products to the wishlist, click the **Cart** button or visit **http://MACHINE_IP/cart.php** to see the products added to the cart. On the **Cart** page, click the **Proceed to Checkout** button to buy the items as shown below:

![](wishcart.png){: width="1914" height="356"}

On the checkout page, the user will be prompted to enter his name and address as shown below:

![](checkout1.png){: width="1913" height="436"}

Enter any name of your choice and address, and click on **Complete Checkout** to place the wish. Once you complete the wish, you will be shown the message "**Wish successful. Your wish has been saved as Wish #21**", as shown below:

![](checkout2.png){: width="1187" height="454"}

**Wish #21** indicates the wishes placed by a user on the website. Once you click on **Wish #21**, you will see a forbidden page because the details are only accessible to admins. But can we try to bypass this and access other people's wishes? This is what we will try to perform in this task.

![](error.png){: width="1913" height="229"}

**Intercepting the Request**

Before discussing exploiting XXE on the web, let's learn how to intercept the request. First, we need to configure the environment so that, as a pentester, all web traffic from our browser is routed through Burp Suite. This allows us to see and manipulate the requests as we browse. 

We will use Burp Suite, a powerful web vulnerability scanner, to intercept and modify requests for this exploitation. You can access Burp Suite in the AttackBox. On the desktop of the AttackBox, you will see a Burp Suite icon as shown below:

![](burp.png){: width="1041" height="632"}

Once you click the icon, Burp Suite will open with an introductory screen. You will see a message like "**Welcome to Burp Suite**". Click on the **Next** button.

![](burp1.png){: width="742" height="490"}

On the next screen, you will have the option to **Start Burp**. Click on the **Start Burp** button to start the tool.

![](burp2.png){: width="743" height="492"}

Once Burp Suite has started, you will see its main interface with different tabs, such as **Proxy, Intruder, Repeater** and others.

![](burp3.png){: width="898" height="537"}

Inside Burp Suite, click the **Settings** tab at the top right. You will see Burp's browser option available under the **Tools** section. Enable **Allow Burp's browser to run without a sandbox option** and click on the close icon on the top right corner of the **Settings** tab as shown below:

![](burp4.png){: width="1297" height="477"}

After allowing the browser to run without a sandbox, we would now be able to start the browser with pre-configured Burp Suite's proxy. Navigate to the **Open browser** option located at the **Proxy -> Intercept** section of Burp.  Open the browser by clicking the **Open browser** as shown below and browse the URL **http://MACHINE_IP**, so that all requests are intercepted: 

![](burp5.png){: width="1357" height="312"}

Once you browse the URL, all the requests are intercepted and can be seen under the **Proxy->HTTP** history tab.

![](burp6.png){: width="1183" height="582"}

**What is Happening in the Backend?**

Now, when you visit the URL, **http://MACHINE_IP/product.php**, and click **Add to Wishlist**, an AJAX call is made to **wishlist.php** with the following XML as input. 

```xml
<wishlist>
  <user_id>1</user_id>
     <item>
       <product_id>1</product_id>
     </item>
</wishlist>
```

In the above XML, **<product_id>** tag contains the ID of the product, which is 1 in this case. Now, let's review the **Add to Wishlist** request logged in Burp Suite's **HTTP History** option under the proxy tab. As discussed above, the request contains XML being forwarded as a **POST** request, as shown below:

![](burp7.png){: width="1184" height="742"}

This **wishlist.php** accepts the request and parses the request using the following code:

```php
<?php
..
...
libxml_disable_entity_loader(false);
$wishlist = simplexml_load_string($xml_data, "SimpleXMLElement", LIBXML_NOENT);

...
..
echo "Item added to your wishlist successfully.";
?>
```

**Preparing the Payload**

When a user sends specially crafted XML data to the application, the line **libxml_disable_entity_loader(false)** allows the XML parser to load external entities. This means the XML input can include external file references or requests to remote servers. When the XML is processed by **simplexml_load_string** with the **LIBXML_NOENT** option, the web app resolves external entities, allowing attackers access to sensitive files or allowing them to make unintended requests from the server.

What if we update the XML request to include references for external entities? We will use the following XML instead of the above XML:

```xml
<!--?xml version="1.0" ?-->
<!DOCTYPE foo [<!ENTITY payload SYSTEM "/etc/hosts"> ]>
<wishlist>
  <user_id>1</user_id>
     <item>
       <product_id>&payload;</product_id>
     </item>
</wishlist>
```

When we send this updated XML payload, the first two lines introduce an external entity called payload. The line **<!ENTITY payload SYSTEM "/etc/hosts">** tells the XML parser to replace the **&payload**; reference with the contents of the file /etc/hosts on the server. When the XML is processed, instead of a normal **product_id**, the application will try to load and include the contents of the file specified in the entity **(/etc/hosts)**.

**Exploitation**

Now, let's perform the exploitation by repeating the request we captured earlier. The Burp Suite tool has a feature known as **Repeater** that allows you to send multiple HTTP requests. We will use this feature to duplicate our **HTTP POST** request and send it multiple times to exploit the vulnerability. Right-click on the **wishlist.php** POST request and click on **Send to Repeater**.

![](burp8.png){: width="1562" height="721"}

Now, switch to the **Repeater** tab, where you'll find the **POST** request that needs to be modified. We will update the XML payload with the new data as shown below and then send the modified request:

![](burp9.png){: width="746" height="561"}

Place the mouse cursor inside the request in the **Repeater** tab in Burp Suite and press **Ctrl+V**  or paste the payload in the above-highlighted area.

![](burp10.png){: width="1246" height="580"}

When we clicked **Send**, the server processed the malicious XML payload, which included the external entity reference to **/etc/hosts**. As a result, the **wishlist.php** responded with the contents of the **/etc/hosts** file, leading to an XXE vulnerability.

## Time for Some Action

Now that you've identified a vulnerability in the application, it's time to see it in action! McSkidy Software has tasked us with finding loopholes, and we've successfully uncovered one in the **wishlist.php** endpoint. But our work doesn't end there—let's take it a step further and assess the potential impact this vulnerability could have on the application.

Earlier, we discovered a page accessible only by administrators, which seems like an exciting target. What if we could use the vulnerability we've found to access sensitive information, like the wishes placed by the townspeople?

Now that our objective is clear, let's leverage the vulnerability we discovered to read the contents of each wishes page and demonstrate the full extent of this flaw to help McSkidy secure the platform. To get started, let's recall the page that is only accessible by admins - **/wishes/wish_1.txt**. Using this path, we just need to guess the potential absolute path of the file. Typically, web applications are hosted on **/var/www/html**. With that in mind, let's build our new payload to read the wishes while leveraging the vulnerability.

**Note: Not all web applications use the path /var/www/html, but web servers typically use it.**

```xml
<!--?xml version="1.0" ?-->
<!DOCTYPE foo [<!ENTITY payload SYSTEM "/var/www/html/wishes/wish_1.txt"> ]>
<wishlist>
  <user_id>1</user_id>
  <item>
         <product_id>&payload;</product_id>
  </item>
</wishlist>
```

![](payload1.png){: width="2158" height="738"}

Surprisingly, we got lucky that our assumption worked. The next thing to do is see whether we can view more wishes using our discovery. To do this, let's try replacing the **wish_1.txt** with **wish_2.txt**.

![](payload2.png){: width="2114" height="748"}

As a result, we were able to view the next wish. You may observe that we just incremented the number by one. Given this, you may continue checking the other wishes and see all the wishes stored in the application.

After iterating through the wishes, we have proved the potential impact of the vulnerability, and anyone who leverages this could read the wishes submitted by the townspeople of Wareville.

## Conclusion

It was confirmed that the application was vulnerable, and the developers were not at fault since they only wanted to give the townspeople something before Christmas. However, it became evident that bypassing security testing led to an application that did not securely handle incoming requests.

As soon as the vulnerability was discovered, McSkidy promptly coordinated with the developers to implement the necessary mitigations. The following proactive approach helped to address the potential risks against XXE attacks:

- **Disable External Entity Loading**: The primary fix is to disable external entity loading in your XML parser. In PHP, for example, you can prevent XXE by setting **libxml_disable_entity_loader(true)** before processing the XML.
- **Validate and Sanitise User Input**: Always validate and sanitise the XML input received from users. This ensures that only expected data is processed, reducing the risk of malicious content being included in the request. For example, remove suspicious keywords like **/etc/host**, **/etc/passwd**, etc, from the request.

After discovering the vulnerability, McSkidy immediately remembered that a CHANGELOG file exists within the web application, stored at the following endpoint: http://MACHINE_IP/CHANGELOG. After checking, it can be seen that someone pushed the vulnerable code within the application after Software's team.

![](discovery.png){: width="754" height="372"}

With this discovery, McSkidy still couldn't confirm whether the Mayor intentionally made the application vulnerable. However, the Mayor had already become suspicious, and McSkidy began to formulate theories about his possible involvement.

## Answers

### Question 1

What is the flag discovered after navigating through the wishes?

<details>
  <summary style="cursor:pointer; padding:10px; border:1px solid #ccc; background-color:#f0f0f0; user-select: none;">Answer</summary>
  <div style="padding:10px; border:1px solid #ccc;">
    <span onclick="navigator.clipboard.writeText('THM{Brut3f0rc1n6_mY_w4y}')" style="cursor:pointer;">THM{Brut3f0rc1n6_mY_w4y}</span>
    <i onclick="navigator.clipboard.writeText('THM{Brut3f0rc1n6_mY_w4y}')" style="float:right; cursor:pointer; font-size:16px;">&#x1F4C4;</i>
  </div>
</details>

### Question 2

What is the flag seen on the possible proof of sabotage?

<details>
  <summary style="cursor:pointer; padding:10px; border:1px solid #ccc; background-color:#f0f0f0; user-select: none;">Answer</summary>
  <div style="padding:10px; border:1px solid #ccc;">
    <span onclick="navigator.clipboard.writeText('THM{m4y0r_m4lw4r3_b4ckd00rs}')" style="cursor:pointer;">THM{m4y0r_m4lw4r3_b4ckd00rs}</span>
    <i onclick="navigator.clipboard.writeText('THM{m4y0r_m4lw4r3_b4ckd00rs}')" style="float:right; cursor:pointer; font-size:16px;">&#x1F4C4;</i>
  </div>
</details>