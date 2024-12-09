---
title: "TryHackMe: Advent Of Cyber 2024 Day 7"
categories: [TryHackMe]
tags: [log analysis, aws, advent of cyber]
render_with_liquid: false
img_path: /images/tryhackme_aoc2024_day7/
image:
  path: banner.png
---

<p align="center"><i>As SOC-mas approached, so did the need,</i><br>
<i>To provide those without, with something to read.</i><br>
<i>Care4Wares tried, they made it their mission,</i><br>
<i>A gift for all wares, a SOC-mas tradition.</i></p><br>

<p align="center"><i>Although they had some, they still needed more,</i><br>
<i>To pick up some books, they’d head to the store.</i><br>
<i>The town’s favourite books, would no doubt make them jolly,</i><br>
<i>They ticked off the list, as they filled up the trolley.</i></p><br>

<p align="center"><i>With the last book ticked off, the shopping was done,</i><br>
<i>When asked for their card, the ware handed them one.</i><br>
<i>“I’m sorry” he said, as the shop clerk reclined,</i><br>
<i>“I can’t sell you these books, as your card has declined.”</i></p><br>

<p align="center"><i>The ware put them back, as they walked in confusion,</i><br>
<i>How could this be? An attack? An intrusion?</i><br>
<i>And when they logged on, the ware got a scare,</i><br>
<i>To find the donations, they just weren’t there!</i></p>

![Tryhackme Room Link](bell.png){: width="1152" height="300" .shadow }
_<https://tryhackme.com/r/room/adventofcyber2024>_

![](book.png){: width="1140" height="800"}

## Monitoring in an AWS Environment

Care4Wares' infrastructure runs in the cloud, so they chose AWS as their Cloud Service Provider (CSP). Instead of their workloads running on physical machines on-premises, they run on virtualised instances in the cloud. These instances are (in AWS) called EC2 instances (Amazon Elastic Compute Cloud). A few members of the Wareville SOC aren't used to log analysis on the cloud, and with a change of environment comes a change of tools and services needed to perform their duties. Their duties this time are to help Care4Wares figure out what has happened to the charity's funds; to do so, they will need to learn about an AWS service called CloudWatch.

### CloudWatch

AWS CloudWatch is a monitoring and observability platform that gives us greater insight into our AWS environment by monitoring applications at multiple levels. CloudWatch provides functionalities such as the monitoring of system and application metrics and the configuration of alarms on those metrics for the purposes of today's investigation, though we want to focus specifically on CloudWatch logs. Running an application in a cloud environment can mean leveraging lots of different services (e.g. a service running the application, a service running functions triggered by that application, a service running the application backend, etc.); this translates to logs being generated from lots of different sources. CloudWatch logs make it easy for users to access, monitor and store the logs from all these various sources. A CloudWatch agent must be installed on the appropriate instance for application and system metrics to be captured.

A key feature of CloudWatch logs that will help the Warevile SOC squad and us make sense of what happened in their environment is the ability to query application logs using filter patterns. Here are some CloudWatch terms you should know before going further:

- **Log Events**: A log event is a single log entry recording an application "event"; these will be timestamped and packaged with log messages and metadata.
- **Log Streams**: Log streams are a collection of log events from a single source.
- **Log Groups**: Log groups are a collection of log streams. Log streams are collected into a log group when logically it makes sense, for example, if the same service is running across multiple hosts.

### CloudTrail

CloudWatch can track infrastructure and application performance, but what if you wanted to monitor actions in your AWS environment? These would be tracked using another service called AWS CloudTrail. Actions can be those taken by a user, a role (granted to a user giving them certain permissions) or an AWS service and are recorded as events in AWS CloudTrail. Essentially, any action the user takes (via the management console or AWS CLI) or service will be captured and stored. Some features of CloudTrail include:

- **Always On**: CloudTrail is enabled by default for all users
- **JSON-formatted**: All event types captured by CloudTrail will be in the CloudTrail JSON format
- **Event History**: When users access CloudTrail, they will see an option "Event History", event history is a record of the actions that have taken place in the last 90 days. These records are queryable and can be filtered on attributes such as "resource" type.
- **Trails**: The above-mentioned event history can be thought of as the default "trail," included out of the box. However, users can define custom trails to capture specific actions, which is useful if you have bespoke monitoring scenarios you want to capture and store beyond the **90-day event history retention period**.
- **Deliverable**:  As mentioned, CloudWatch can be used as a single access point for logs generated from various sources; CloudTrail is no different and has an optional feature enabling **CloudTrail logs to be delivered to CloudWatch**.

![](aws.png){: width="501" height="800"}

As mentioned, Cloudtrail helps capture and record actions taken. These actions could be interactions with any number of AWS services. For example, services like S3 (Amazon Simple Storage Service used for object storage) and IAM (AWS's Identity and Access Management service can be used to secure access to your AWS environment with the creation of identities and the assigning of access permissions to those identities) will have actions taken within their service recorded. These recorded events can be very helpful when performing an investigation.

## Intro to JQ

**What is JQ?**

Earlier, it was mentioned that Cloudtrail logs were JSON-formatted. When ingested in large volumes, this machine-readable format can be tricky to extract meaning from, especially in the context of log analysis. The need then arises for something to help us transform and filter that JSON data into meaningful data we can understand and use to gain security insights. That's exactly what JQ is (and does!). Similar to command line tools like sed, awk and grep, JQ is a lightweight and flexible command line processor that can be used on JSON.

![](jq.png){: width="1141" height="651"}

**How Can It Be Used?**

Now, let's take a look at how we use JQ to transform and filter JSON data. The wares being the wares, they stored their shopping list from the trip to the bookstore in JSON format. Let's take a look at that:

```json
[

{ "book_title": "Wares Wally", "genre": "children", "page_count": 20 },

{ "book_title": "Charlottes Web Crawler", "genre": "young_ware", "page_count": 120 },

{ "book_title": "Charlie and the 8 Bit Factory", "genre": "young_ware", "page_count": 108 },

{ "book_title": "The Princess and the Pcap", "genre": "children", "page_count": 48 },

{ "book_title": "The Lion, the Glitch and the Wardrobe", "genre": "young_ware", "page_count": 218 }

]
```

JQ takes two inputs: the filter you want to use, followed by the input file. We start our JQ filter with a **.** which just tells JQ we are accessing the current input. From here, we want to access the array of values stored in our JSON (with the **[]**). Making our filter a *.[]*. For example, let’s run the following command. 

```console
user@tryhackme$ jq '.[]' book_list.json
```

The command above would result in this output:

```json
{
  "book_title": "Wares Wally",
  "genre": "children",
  "page_count": 20
}
{
  "book_title": "Charlottes Web Crawler",
  "genre": "young_ware",
  "page_count": 120
}
{
  "book_title": "Charlie and the 8 Bit Factory",
  "genre": "young_ware",
  "page_count": 108
}
{
  "book_title": "The Princess and the Pcap",
  "genre": "children",
  "page_count": 48
}
{
  "book_title": "The Lion, the Glitch and the Wardrobe",
  "genre": "young_ware",
  "page_count": 218
}
```

Once we've accessed the array, we can grab elements from that array by going one step deeper. For example, we could run this JQ command:

```console
user@tryhackme$ jq  '.[] | .book_title' book_list.json
```

If we wanted to view all the book titles contained within this JSON file, this would return a nicely formatted output like this:

```json
"Wares Wally"
"Charlottes Web Crawler"
"Charlie and the 8 Bit Factory"
"The Princess and the Pcap"
"The Lion, the Glitch and the Wardrobe"
```

That's a lot nicer to look at, isn't it? It gives you an idea of what JQ is and what it does. Of course, JQ can filter and transform JSON data in many additional ways. In our upcoming investigation, we'll see the tool in action.

## The Peculiar Case of Care4Wares’ Dry Funds

Now that we have refreshed our knowledge of AWS Cloudtrail and JQ alongside McSkidy, let’s investigate this peculiar case of Care4Wares’ dry funds.

The responsible ware for the Care4Wares charity drive gave us the following info regarding this incident:

We sent out a link on the 28th of November to everyone in our network that points to a flyer with the details of our charity. The details include the account number to receive donations. We received many donations the first day after sending out the link, but there were none from the second day on. I talked to multiple people who claimed to have donated a respectable sum. One showed his transaction, and I noticed the account number was wrong. I checked the link, and it was still the same. I opened the link, and the digital flyer was the same except for the account number.

McSkidy recalls putting the digital flyer, **wareville-bank-account-qr.png**, in an Amazon AWS S3 bucket named **wareville-care4wares**. Let’s assist McSkidy and start by finding out more about that link. Before that, let’s first review the information that we currently have to start the investigation:

- The day after the link was sent out, several donations were received.
- Since the second day after sending the link, no more donations have been received.
- A donator has shown proof of his transaction. It was made 3 days after he received the link. The account number in the transaction was not correct.
- McSkidy put the digital flyer in the AWS S3 object named **wareville-bank-account-qr.png** under the bucket **wareville-care4wares**.
- The link has not been altered.

## Glitch Did It

Let’s examine the Cloudtrail logs related to the wareville-care4wares S3 bucket. For a quick example, a typical S3 log entry looks like this:

```json
{
  "eventVersion": "1.10",
  "userIdentity": {
    "type": "IAMUser",
    "principalId": "AIDAXRMKYT5O5Y2GLD4ZG",
    "arn": "arn:aws:iam::518371450717:user/wareville_collector",
    "accountId": "518371450717",
    "accessKeyId": "AKIAXRMKYT5OZCZPGNZ7",
    "userName": "wareville_collector"
  },
  "eventTime": "2024-10-21T22:13:24Z",
  "eventSource": "s3.amazonaws.com",
  "eventName": "ListObjects",
  "awsRegion": "ap-southeast-1",
  "sourceIPAddress": "34.247.218.56",
  "userAgent": "[aws-sdk-go/0.24.0 (go1.22.6; linux; amd64)]",
  "requestParameters": {
    "bucketName": "aoc-cloudtrail-wareville",
    "Host": "aoc-cloudtrail-wareville.s3.ap-southeast-1.amazonaws.com",
    "prefix": ""
  },
  "responseElements": null,
  "additionalEventData": {
    "SignatureVersion": "SigV4",
    "CipherSuite": "TLS_AES_128_GCM_SHA256",
    "bytesTransferredIn": 0,
    "AuthenticationMethod": "AuthHeader",
    "x-amz-id-2": "yqniVtqBrL0jNyGlvnYeR3BvJJPlXdgxvjAwwWhTt9dLMbhgZugkhlH8H21Oo5kNLiq8vg5vLoj3BNl9LPEAqN5iHpKpZ1hVynQi7qrIDk0=",
    "bytesTransferredOut": 236375
  },
  "requestID": "YKEKJP7QX32B4NZB",
  "eventID": "fd80529f-d0af-4f44-8034-743d8d92bdcf",
  "readOnly": true,
  "resources": [
    {
      "type": "AWS::S3::Object",
      "ARNPrefix": "arn:aws:s3:::aoc-cloudtrail-wareville/"
    },
    {
      "accountId": "518371450717",
      "type": "AWS::S3::Bucket",
      "ARN": "arn:aws:s3:::aoc-cloudtrail-wareville"
    }
  ],
  "eventType": "AwsApiCall",
  "managementEvent": false,
  "recipientAccountId": "518371450717",
  "eventCategory": "Data",
  "tlsDetails": {
    "tlsVersion": "TLSv1.3",
    "cipherSuite": "TLS_AES_128_GCM_SHA256",
    "clientProvidedHostHeader": "aoc-cloudtrail-wareville.s3.ap-southeast-1.amazonaws.com"
  }
}
```

It might be overwhelming to see the sheer amount of information in one event, but there are some elements that we can focus on for our investigation:

| **Field**            | **Description**                                                                 |
|----------------------|---------------------------------------------------------------------------------|
| `userIdentity`       | Details of the user account that acted on an object.                            |
| `eventTime`          | When did the action occur?                                                     |
| `eventType`          | What type of event occurred? (e.g., AwsApiCall, AwsConsoleSignIn, AwsServiceEvent) |
| `eventSource`        | From what service was the event logged?                                         |
| `eventName`          | What specific action occurred? (e.g., ListObjects, GetBucketObject)            |
| `sourceIPAddress`    | From what IP did the action happen?                                             |
| `userAgent`          | What user agent was used to perform the action? (e.g., Firefox, AWS CLI)      |
| `requestParameters`  | What parameters were involved in the action? (e.g., BucketName)                |

By using the guide above, we can read the example log entry as follows: 

- The IAM user, **wareville_collector**, listed all objects (ListObjects event) of the S3 bucket named **aoc-cloudtrail-wareville**.
- The IP address from which this request originated is **34.247.218.56**.
- The user agent indicates that the request was made using the **AWS SDK tool for Go**.

Now that we know where to look, let’s use JQ to filter the log for events related to the **wareville-bank-account-qr.png** S3 object. The goal is to use the same elements to filter the log file using JQ and format the results into a table to make it more readable. According to McSkidy, the logs are stored in the **~/wareville_logs** directory.

To start, click the Terminal icon on the Desktop and enter the two commands below:

```console
ubuntu@tryhackme:~/$ cd wareville_logs
ubuntu@tryhackme:~/$ ls
cloudtrail_log.json  rds.log
```

With the commands above, we initially changed our current directory to the directory McSkidy mentioned via the **cd** command, and we listed the directory's contents using the **ls** command. As you can see, two files are inside it, but we will focus first on the **cloudtrail_log.json** for this investigation. 

Now, let's start investigating the CloudTrail logs by executing the command below.

```console
ubuntu@tryhackme:~/wareville_logs$ jq -r '.Records[] | select(.eventSource == "s3.amazonaws.com" and .requestParameters.bucketName=="wareville-care4wares")' cloudtrail_log.json
```

Let's do a quick breakdown of the command we executed:

| **Command**                                                      | **Description**                                                                                                  |
|------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------|
| `jq -r 'FILTER' cloudtrail_log.json`                              | The `-r` flag tells `jq` to output the results in RAW format instead of JSON. The `FILTER` section is enclosed with single quotes. The last part of the command accepts the input file, which is `cloudtrail_log.json`. |
| `.Records[]`                                                     | Instructs `jq` to parse the events in the `Records` container element. The `Records` field is the top element in the JSON-formatted CloudTrail log. |
| `| select(.eventSource == "s3.amazonaws.com" and .requestParameters.bucketName=="wareville-care4wares")` | Uses the previous command's output, and filters it on the `eventSource` and `requestParameters.bucketName` keys. The value `s3.amazonaws.com` is used to filter events related to the Amazon AWS S3 service, and the value `wareville-care4wares` is used to filter events related to the target S3 bucket. |




