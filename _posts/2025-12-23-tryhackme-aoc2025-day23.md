---
title: "TryHackMe: Advent Of Cyber 2025 Day 23 (AWS Security - S3cret Santa)"
categories: [TryHackMe]
tags: [aws, cloud, advent of cyber]
render_with_liquid: false
media_subpath: /images/tryhackme_aoc2025_day23/
image:
  path: banner.png
---

Learn the basics of AWS enumeration.

<h2 style="color:#A64AC9; text-align:center; font-weight:700; font-size:1.8em; text-shadow: 1px 0 #A64AC9, -1px 0 #A64AC9, 0 1px #A64AC9, 0 -1px #A64AC9;">
  The Story
</h2>

[![TryHackMe Room Link](header.png){: width="1200" height="407" }](https://tryhackme.com/room/cloudenum-aoc2025-y4u7i0o3p6)

## Introduction

One of our stealthiest infiltrated elves managed to hop their way into Sir Carrotbane’s office and, lo and behold, discovered a bundle of cloud credentials just lying around on his desktop like forgotten carrots. The agent suspects these could be the key to regaining access to TBFC’s cloud network. If only the poor hare had the faintest clue what “the cloud” is, he’d burrow in himself. Let's help the elf utilise these credentials to try to regain access to TBFC's cloud network.

AWS accounts can be accessed programmatically by using an Access Key ID and a Secret Access Key. For this room, both of those will be automatically configured for you. The AWS CLI will look for credentials at `~/.aws/credentials`, where you should see something similar to the following:

Amazon Security Token Service (STS) allows us to utilise the credentials of a user that we have saved during our AWS CLI configuration. We can use the `get-caller-identity` call to retrieve information about the user we have configured for the AWS CLI. Let's run the following command:

`aws sts get-caller-identity`

We will see the following output when we run this command.

```console
user@machine$ aws sts get-caller-identity
{
    "UserId": "AIDAU2VYTBGYOHNOCJMX3",
    "Account": "332173347248",
    "Arn": "arn:aws:iam::332173347248:user/sir.carrotbane"
}
```

Seeing the output, the elf was overjoyed. The credentials work, and as can be seen by the name at the end, they belong to Sir Carrotbane. The elf can now attempt to regain access to TBFC's cloud network using these credentials.

## IAM Overview

Amazon Web Services utilises the Identity and Access Management (IAM) service to manage users and their access to various resources, including the actions that can be performed against those resources. Therefore, it is crucial to ensure that the correct access is assigned to each user according to the requirements. Misconfiguring IAM has led to several high-profile security incidents in the past, giving attackers access to resources they were not supposed to access. Companies like Toyota, Accenture and Verizon have been victims of such attacks in the past, often exposing customer data or sensitive documents. Below, we will discuss the different aspects of IAM that can lead to sensitive data exposure if misconfigured.

## IAM Users

A user represents a single identity in AWS. Each user has a set of credentials, such as passwords or access keys, that can be used to access resources. Furthermore, permissions can be granted at a user level, defining the level of access a user might have.

![](carrotbane.png){: width="391" height="488"}

## IAM Groups

Multiple users can be combined into a group. This can be done to ease the access management for multiple users. For example, in an organisation employing hundreds of thousands of people, there might be a handful of people who need write access to a certain database. Instead of granting access to each user individually, the admin can grant access to a group and add all users who require write access to that group. When a user no longer needs access, they can be removed from the group.

![](army.png){: width="640" height="640"}

## IAM Roles

An IAM Role is a temporary identity that can be assumed by a user, as well as by services or external accounts, to get certain permissions. Think of Sir Carrotbane, and how, depending on the battle ahead, he might need to assume the role of an attacker or a defender. When becoming an attacker, he will get permission to wield his shiny swords, but when assuming the role of a defender, he will instead get permission to carry a shield to better defend King Malhare.

![](role.png){: width="900" height="600"}

## IAM Policies

Access provided to any user, group or role is controlled through IAM policies. A policy is a JSON document that defines the following:

- What action is allowed (Action)
- On which resources (Resource)
- Under which conditions (Condition)
- For whom (Principal)

Consider the following hypothetical policy

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowSpecificUserReadAccess",
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::123456789012:user/Alice"
      },
      "Action": [
        "s3:GetObject"
      ],
      "Resource": "arn:aws:s3:::my-private-bucket/*"
    }
  ]
}
```

This policy grants access to the AWS user Alice (Principal) to get an object from an S3 bucket (Action) for the S3 bucket named my-private-bucket (Resource).

## Enumerating Users

Alright, let's see what we can do with the credentials we got from Sir Carrotbane's office, since we have already configured them in our environment. We can start interacting with the AWS CLI to find more information. Let's begin by enumerating users. We can do so by running the following command in the terminal:

`aws iam list-users`

We will see an output that lists all the users, as well as some other useful information such as their creation date. 

## Enumerating User Policies

Policies can be inline or attached. **Inline policies** are assigned directly in the user (or group/role) profile and hence will be deleted if the identity is deleted. These can be considered as hard-coded policies as they are hard-coded in the identity definitions. **Attached policies**, also called managed policies, can be considered reusable. An attached policy requires only one change in the policy, and every identity that policy is attached to will inherit that change automatically.

Let's see what inline policies are assigned to Sir Carrotbane by running the following command.

`aws iam list-user-policies --user-name sir.carrotbane`

Great! We can see an inline policy in the results. Let's take note of its name for later.

Maybe, Sir Carrotbane has some policies attached to their account. We can find out by running the following command.

`aws iam list-attached-user-policies --user-name sir.carrotbane`

Hmmm, not much here. Perhaps we can check if Sir Carrotbane is part of a group. Let's run this command to do that.

`aws iam list-groups-for-user --user-name sir.carrotbane`

Looks like sir.carrotbane is not a part of any group.

Let's get back to the inline policy we found for Sir Carrotbane's account. Let's see what permissions this policy grants by running the following command (replace `POLICYNAME` with the actual policy name you found):

`aws iam get-user-policy --policy-name POLICYNAME --user-name sir.carrotbane`

```json
{
    "UserName": "sir.carrotbane",
    "PolicyName": "POLICYNAME",
    "PolicyDocument": {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Action": [
                    "iam:ListUsers",
                    "iam:ListGroups",
                    "iam:ListRoles",
                    "iam:ListAttachedUserPolicies",
                    "iam:ListAttachedGroupPolicies",
                    "iam:ListAttachedRolePolicies",
                    "iam:GetUserPolicy",
                    "iam:GetGroupPolicy",
                    "iam:GetRolePolicy",
                    "iam:GetUser",
                    "iam:GetGroup",
                    "iam:GetRole",
                    "iam:ListGroupsForUser",
                    "iam:ListUserPolicies",
                    "iam:ListGroupPolicies",
                    "iam:ListRolePolicies",
                    "sts:AssumeRole"
                ],
                "Effect": "Allow",
                "Resource": "*",
                "Sid": "ListIAMEntities"
            }
        ]
    }
}
```

So, it looks like Sir Carrotbane has access to enumerate all the different kinds of users, groups, roles and policies (IAM entities), but that is about it. That is not a lot of help getting TBFC's access back. We might need to try something different to do that. If you look carefully, you'll notice sir.carrotbane can perform the `sts:AssumeRole` action. Maybe there's still hope!

## Enumerating Roles

The `sts:AssumeRole` action we previously found allows sir.carrotbane to assume roles. Perhaps we can try to see if there's any interesting ones available. Let's start by listing the existing roles in the account.

`aws iam list-roles`

```json
{
    "Roles": [
        {
            "Path": "/",
            "RoleName": "bucketmaster",
            "RoleId": "AROARZPUZDIKJJZ6OWN27",
            "Arn": "arn:aws:iam::123456789012:role/bucketmaster",
            "CreateDate": "2025-11-26T01:54:01.342203+00:00",
            "AssumeRolePolicyDocument": {
                "Statement": [
                    {
                        "Action": "sts:AssumeRole",
                        "Effect": "Allow",
                        "Principal": {
                            "AWS": "arn:aws:iam::123456789012:user/sir.carrotbane"
                        }
                    }
                ],
                "Version": "2012-10-17"
            },
            "MaxSessionDuration": 3600
        }
    ]
}
```

Bingo! There's a role named `bucketmaster`, and it can be assumed by `sir.carrotbane`. Let's find out what policies are assigned to this role. Just as users, roles can have inline policies and attached policies. To check the inline policies, we can run the following command.

`aws iam list-role-policies --role-name bucketmaster`

There is one policy assigned to this role. Before checking that policy, let's see if there are any attached policies assigned to the role as well.

`aws iam list-attached-role-policies --role-name bucketmaster`

Looks like we only have the inline policy assigned. Let's see what permissions we can get from the policy.

`aws iam get-role-policy --role-name bucketmaster --policy-name BucketMasterPolicy`

```json
{
    "RoleName": "bucketmaster",
    "PolicyName": "BucketMasterPolicy",
    "PolicyDocument": {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Action": [
                    "s3:ListAllMyBuckets"
                ],
                "Effect": "Allow",
                "Resource": "*",
                "Sid": "ListAllBuckets"
            },
            {
                "Action": [
                    "s3:ListBucket"
                ],
                "Effect": "Allow",
                "Resource": [
                    "arn:aws:s3:::easter-secrets-123145",
                    "arn:aws:s3:::bunny-website-645341"
                ],
                "Sid": "ListBuckets"
            },
            {
                "Action": [
                    "s3:GetObject"
                ],
                "Effect": "Allow",
                "Resource": "arn:aws:s3:::easter-secrets-123145/*",
                "Sid": "GetObjectsFromEasterSecrets"
            }
        ]
    }
}
```

Well, what do we have here? We can see that the `bucketmaster` role can perform three different actions (ListAllBuckets, ListBucket and GetObject) on some resources of a service named S3. This might just be the breakthrough we were looking for. More on this service later.

Assuming Role
To gain privileges assigned by the `bucketmaster` role, we need to assume it. We can use AWS STS to obtain the temporary credentials that enable us to assume this role. 

`aws sts assume-role --role-arn arn:aws:iam::123456789012:role/bucketmaster --role-session-name TBFC`

This command will ask STS, the service in charge of AWS security tokens, to generate a temporary set of credentials to assume the `bucketmaster` role. The temporary credentials will be referenced by the session-name "TBFC" (you can set any name you want for the session). Let's run the command:

```json
{
    "Credentials": {
        "AccessKeyId": "{REDACTED}",
        "SecretAccessKey": "{REDACTED}",
        "SessionToken": "{REDACTED}",
        "Expiration": "2025-11-26T03:40:11.117460+00:00"
    },
    "AssumedRoleUser": {
        "AssumedRoleId": "AROARZPUZDIKJJZ6OWN27:TBFC",
        "Arn": "arn:aws:sts::{REDACTED}:assumed-role/bucketmaster/TBFC"
    },
    "PackedPolicySize": 6
}
```

The output will provide us the credentials we need to assume this role, specifically the `AccessKeyID`, `SecretAccessKey` and `SessionToken`. To be able to use these, run the following commands in the terminal, replacing with the exact credentials that you received on running the `assume-role` command.

```console
user@machine$ export AWS_ACCESS_KEY_ID="{REDACTED}"
user@machine$ export AWS_SECRET_ACCESS_KEY="{REDACTED}"
user@machine$ export AWS_SESSION_TOKEN="{REDACTED}"
```

Once we have done that, we can officially use the permissions granted by the `bucketmaster` role. To check if you have correctly assumed the role, you can once again run:

`aws sts get-caller-identity`

This time, it should show you are now using the `bucketmaster` role.

## What Is S3?

Before we continue, we need to know what exactly is S3. Amazon S3 stands for **Simple Storage Service**. It is an object storage service provided by Amazon Web Services that can store any type of object such as images, documents, logs and backup files. Companies often use S3 to store data for various reasons, such as reference images for their website, documents to be shared with clients, or files used by internal services for internal processing. Any object you store in S3 will be put into a "Bucket". You can think of a bucket as a directory where you can store files, but in the cloud.

![](bucket.png){: width="600" height="900"}

Now, let's see what our newly gained access to Sir Carrotbane's S3 bucket provides us.

## Listing Contents From a Bucket

Since our profile has permission to ListAllBuckets, we can list the available S3 buckets by running the following command:

`aws s3api list-buckets`

There is one interesting bucket in there that references easter. Let's check out the contents of this directory.

`aws s3api list-objects --bucket easter-secrets-123145`

Hmmm, let's copy the file in this directory to our local machine. This might have a secret message.

`aws s3api get-object --bucket easter-secrets-123145 --key cloud_password.txt cloud_password.txt`

Hooray! We have successfully infiltrated Sir Carrotbane's S3 bucket and exfiltrated some sensitive data.

## Walkthrough

**Identifying the Current AWS Identity**

Once the AWS CLI is configured using the exposed credentials, the first step is determining who you are within the AWS account.

```console
aws sts get-caller-identity
```

This command returns:

- AWS account ID
- IAM user or role ARN
- User ID

**AWS IAM Fundamentals**

It is important to understand the IAM components involved in this challenge.

  **IAM Users**
- Long-term identities
- Use access keys
- Commonly over-privileged in misconfigured environments

  **IAM Roles**
- Temporary identities
- Assumed using AWS STS
- Frequently abused for privilege escalation

  **IAM Policies**
- Written in JSON
- Define allowed or denied actions
- Permissions must be explicitly granted

The challenge relies heavily on **role assumption**, a common AWS attack vector.

**Enumerating User Permissions**

Next, you examine what actions your IAM user is allowed to perform. Enumeration reveals that the user has:

- Limited read‑only permissions
- Permission to use `sts:AssumeRole`

Although this permission may appear harmless, it is extremely powerful if a trusted role exists.
If a role trusts your user, you can become that role.

**Discovering IAM Roles**

```console
aws iam list-roles
```

Among the results, a role named `bucketmaster` stands out.
Reviewing the role trust policy shows that your current IAM user is allowed to assume it. This is a serious security misconfiguration.

**Assuming the Privileged Role**

Since the role trusts your user, you assume it using AWS STS:

```console
aws sts assume-role \
  --role-arn arn:aws:iam::<ACCOUNT_ID>:role/bucketmaster \
  --role-session-name s3session
```

AWS responds with temporary credentials:

- Access Key ID
- Secret Access Key
- Session Token

```console
ubuntu@tryhackme:~$ export AWS_ACCESS_KEY_ID={REDACTED}
ubuntu@tryhackme:~$ export AWS_SECRET_ACCESS_KEY={REDACTED}
ubuntu@tryhackme:~$ export AWS_SESSION_TOKEN={REDACTED}
```

After exporting these values, your identity changes from a low‑privileged user to the bucketmaster role.

**Enumerating S3 Buckets**

```console
ubuntu@tryhackme:~$ aws s3api get-object --bucket easter-secrets-123145 --key cloud_password.txt cloud_password.txt
```

Open cloud_password.txt to retrieve the flag.

## Answer

### Question 1

Run `aws sts get-caller-identity`. What is the number shown for the "Account" parameter?

<details>
  <summary style="cursor:pointer; padding:10px; border:1px solid #888; background-color:#444; color:#fff; user-select: none;">
    Answer
  </summary>
  <div style="padding:10px; border:1px solid #888; background-color:#333; color:#fff;">
    <span onclick="navigator.clipboard.writeText('123456789012')" style="cursor:pointer;">123456789012</span>
    <i onclick="navigator.clipboard.writeText('123456789012')" style="float:right; cursor:pointer; font-size:16px;">&#x1F4C4;</i>
  </div>
</details>

### Question 2

What IAM component is used to describe the permissions to be assigned to a user or a group?

<details>
  <summary style="cursor:pointer; padding:10px; border:1px solid #888; background-color:#444; color:#fff; user-select: none;">
    Answer
  </summary>
  <div style="padding:10px; border:1px solid #888; background-color:#333; color:#fff;">
    <span onclick="navigator.clipboard.writeText('policy')" style="cursor:pointer;">policy</span>
    <i onclick="navigator.clipboard.writeText('policy')" style="float:right; cursor:pointer; font-size:16px;">&#x1F4C4;</i>
  </div>
</details>

### Question 3

What is the name of the policy assigned to `sir.carrotbane`?

<details>
  <summary style="cursor:pointer; padding:10px; border:1px solid #888; background-color:#444; color:#fff; user-select: none;">
    Answer
  </summary>
  <div style="padding:10px; border:1px solid #888; background-color:#333; color:#fff;">
    <span onclick="navigator.clipboard.writeText('SirCarrotbanePolicy')" style="cursor:pointer;">SirCarrotbanePolicy</span>
    <i onclick="navigator.clipboard.writeText('SirCarrotbanePolicy')" style="float:right; cursor:pointer; font-size:16px;">&#x1F4C4;</i>
  </div>
</details>

### Question 4

Apart from GetObject and ListBucket, what other action can be taken by assuming the bucketmaster role?

<details>
  <summary style="cursor:pointer; padding:10px; border:1px solid #888; background-color:#444; color:#fff; user-select: none;">
    Answer
  </summary>
  <div style="padding:10px; border:1px solid #888; background-color:#333; color:#fff;">
    <span onclick="navigator.clipboard.writeText('ListAllMyBuckets')" style="cursor:pointer;">ListAllMyBuckets</span>
    <i onclick="navigator.clipboard.writeText('ListAllMyBuckets')" style="float:right; cursor:pointer; font-size:16px;">&#x1F4C4;</i>
  </div>
</details>

### Question 5

What are the contents of the cloud_password.txt file?

<details>
  <summary style="cursor:pointer; padding:10px; border:1px solid #888; background-color:#444; color:#fff; user-select: none;">
    Answer
  </summary>
  <div style="padding:10px; border:1px solid #888; background-color:#333; color:#fff;">
    <span onclick="navigator.clipboard.writeText('THM{more_like_sir_cloudbane}')" style="cursor:pointer;">THM{more_like_sir_cloudbane}</span>
    <i onclick="navigator.clipboard.writeText('THM{more_like_sir_cloudbane}')" style="float:right; cursor:pointer; font-size:16px;">&#x1F4C4;</i>
  </div>
</details>