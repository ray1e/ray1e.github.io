---
layout: post
title: Billing Writeup
author: 0xray1e
date: 2026-03-04T14:30:00.000+03:00
description: A walkthrough of the Billing Room in TryHackMe.
categories:
  - Boot-to-Root
---
[Billing room THM](https://tryhackme.com/room/billing)

## Room Description

Gain a shell, find the way and escalate your privileges!

**Note:** Bruteforcing is out of scope for this room.

## Recon

After starting the machine and getting the IP address, I started by doing an nmap scan.

```shell
nmap -sC -sV -Pn -p- 10.65.129.26 -T4 -oN nmap.txt
```

`-sC Runs default nmap scripts on discovered services`

`-sV Probe service and version info of open ports`

`-Pn skip host discovery (assume host is online)`

![billing-nmap scan](/assets/img/uploads/billing-nmap scan.png "billing-nmap scan")

The target had an http port open so I opened it in the browser and as you can see it is a login page.

![]()

The room had stated that the bruteforcing is not allowed so I just decided to find whether this was an actual system by looking it up on google. I found out that there is a system called [MagnusBilling](https://www.magnusbilling.org/) which is an open-source billing system. Next, was to look for public exploits.
