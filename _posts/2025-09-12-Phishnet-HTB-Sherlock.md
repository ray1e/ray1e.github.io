---
title: PhishNet HTB Sherlock
date: 2025-09-12 20:36:58 +0300
categories: [HTB, sherlock]
tags: [sherlock, phishing]     # TAG names should always be lowercase
---

# PhishNet - HTB Sherlock Walktrhough
In this walkthrough I will take you through the process of how I solved the Phishnet sherlock in HackTheBox.  

*If you have not attempted the sherlock you can find it [here](https://app.hackthebox.com/sherlocks/PhishNet).*

*If you have not attempted it you can find it [here](https://app.hackthebox.com/sherlocks/PhishNet).*

## Scenario:
An accounting team receives an urgent payment request from a known vendor. The email appears legitimate but contains a suspicious link and a .zip attachment hiding malware. Your task is to analyze the email headers, and uncover the attacker's scheme.

## Download files
The sherlock has a zip file that contains the email. Download it and extract it.

To read the email open it using outlook. The email is about an overdue payment for an invoice, and it contains an attachment tahat contains the invoice(likely malicious files). 

[![phishing-email.png](https://i.postimg.cc/5Np0K2n1/phishing-email.png)](https://postimg.cc/qzh4z0PZ)

