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
- Important Concept

### Extensible Markup Language (XML)

XML is a commonly used method to transport and store data in a structured format that humans and machines can easily understand. Consider a scenario where two computers need to communicate and share data. Both devices need to agree on a common format for exchanging information. This agreement (format) is known as XML. You can think of XML as a digital filing cabinet. Just as a filing cabinet has folders with labelled documents inside, XML uses tags to label and organise information. These tags are like folders that define the type of data stored. This is what an XML looks like, a simple piece of text information organised in a structured manner:

![XML act like a file drawer](drawer.png){: width="740" height="850"}
