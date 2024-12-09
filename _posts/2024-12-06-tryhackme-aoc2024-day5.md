---
title: "TryHackMe: Advent Of Cyber 2024 Day 5"
categories: [TryHackMe]
tags: [xxe, xml, advent of cyber]
render_with_liquid: false
img_path: /images/tryhackme_aoc2024_day5/
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







