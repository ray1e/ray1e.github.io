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


![Phishing-email-2.png](/assets/phishing-email.png)

In order to start analyzing the email there are a few headers that will be of importance to us.  
*Sending address:* The email address of the sender. Can be spoofed.  
*subject line*: The subject of the email (suspicious emails usually have an urgent sounding subject)  
*Recipients(unless they are in BCC)*: In an organization it is important to know who else received the email unless its a blind carbon copy where we can't see the other recipients then we have to look at logs in the mail server.  
*Date + time*: Date and time when the email was sent  
*sending server IP(x-sender IP):* IP address of the sender. It is also important to note that this might be absent depending on the email client you use, due to privacy reasons.  
*reply to address*: where to send replies to the email. This header can be spoofed and may be different from the sending address.  
*Full URL*: Any link that might be included in the email - usually phishing links or download links.  