---
title: PhishNet HTB-Sherlock
date: 2025-09-12 20:36:58 +0300
categories: [HTB, sherlock]
tags: [sherlock, phishing]     # TAG names should always be lowercase
---

# PhishNet - HTB Sherlock Walkthrough

In this walkthrough I will take you through the process of how I solved the Phishnet sherlock in Hack The Box.  

*If you have not attempted the sherlock you can find it [here](https://app.hackthebox.com/sherlocks/PhishNet).*

## Scenario:
An accounting team receives an urgent payment request from a known vendor. The email appears legitimate but contains a suspicious link and a .zip attachment hiding malware. Your task is to analyze the email headers, and uncover the attacker's scheme.
## Downloading files
The sherlock has a zip file that contains the email. Download and extract it.
To read the email open it using outlook. The email is about an overdue payment for an invoice, and it contains an attachment that contains the invoice(likely malicious files).

![phishing email.png](/assets/phishing-email.png)

In order to start analyzing the email there are a few headers that will be of importance to us.

*Sending address:* The email address of the sender. Can be spoofed.

*subject line*: The subject of the email (suspicious emails usually have an urgent sounding subject)

*Recipients(unless they are in BCC)*: In an organization it is important to know who else received the email unless its a blind carbon copy where we can't see the other recipients then we have to look at logs in the mail server.

*Date + time*: Date and time when the email was sent

*sending server IP(x-sender IP):* IP address of the sender. It is also important to note that this might be absent depending on the email client you use, due to privacy reasons.

*reply to address*: where to send replies to the email. This header can be spoofed and may be different from the sending address.

*Full URL*: Any link that might be included in the email - usually phishing links or download links.

## How does email spoofing work?
Before we look at the raw email headers it is essential to understand how spoofing takes place. Basically what happens when a user sends an email using an email client,  the *From* header e.g. john.doe@domain.com is pre-filled for you by the email client. However it is possible to change the address to something like finance@amazon.com using basic scripts, if proper security measures are not implemented by the sender. This is because when you send an email, the email is routed to the outgoing mail server in your domain which uses SMTP (Simple Mail Transfer Protocol) to route the email to the specified domain's recipient mail server. Typically, IMAP or POP3 protocols are  used to retrieve the email into the email client. The outgoing mail server does not  authenticate the *From* header by default. To prevent spoofing, SPF, DMARC and DKIM are utilized.

## Inspect the raw email headers
Now that we have an idea of what we will be looking for, let's dive into it analyzing the headers. To do this, you can use several methods.
### Using outlook
Open the email using outlook and click on the *file* tab.  

![outlook-file-tab.png](/assets/outlook-file-tab.png)  

Go to info and select properties. You'll be able to see internet headers

![outlook-properties-option.png](/assets/outlook-properties-option.png)

![outlook-headers.png](/assets/outlook-headers.png)

This method is however not the best since you can't do searches and the view is not as good.
### Using notepad++
Another option(the one I prefer) would be to use [notepad++.](https://notepad-plus-plus.org/downloads/) You can download it for free and then right click on your email and select edit it with notepad++. This is the tool we will be using for this walkthrough.
### Using mxtoolbox
Alternatively, you could copy the email headers and analyze them using [mxtoolbox.](https://mxtoolbox.com/EmailHeaders.aspx)

## Tasks
### Task 1
**What is the originating IP address of the sender?**
Look for the x-Originating-IP header. This is the IP address of the sender.  

![x-originating-ip.png](/assets/x-originating-ip.png)  

**45.67.89.10**
### Task 2
**Which mail server relayed this email before reaching the victim?**
The received header records the hops made by the email from sender to recipient. The *bottom-most* received header shows where the message originated while the *top-most* received shows the last server that handled the message before it got to your inbox.

![email server that last handled the email.png](/assets/email-server-that-last-handled-the-email.png)

**203.0.113.25**
### Task 3
**What is the sender's email address?**
The *From* header contains the senders email address.

![senders email address.png](/assets/sender-email-address.png)


**finance@business-finance.com**
### Task 4
**What is the 'Reply-To' email address specified in the email?**
Look at the *Reply-To* header. This is  the address that receives replies. Attackers might set this to an email they control and therefore it is important to have a look at it.

![reply-to email.png](/assets/reply-to-email.png)

**support@business-finance.com**
### Task 5
**What is the SPF (Sender Policy Framework) result for this email?**
SPF is an authentication protocol that mitigates email spoofing. It does this by specifying which hosts or IP's are allowed to send emails on behalf on the domain, using SPF records in the DNS records of the sender domain.
Example; if a receiving mail server receives an email from amazon.com, it checks  whether the server's IP is listed as authorized in amazon's DNS records. It then returns a result like *pass* or *fail*. 

SPF results are in the *Authentication-Results* or *Received-SPF* header.

![spf results.png](/assets/spf-results.png)

**pass**

### Task 6
**What is the domain used in the phishing URL inside the email?**
The URL is in the HTML content of the email. 

![domain for phishing url.png](/assets/domain-for-phishing-url.png)

**secure.business-finance.com**
### Task 7
**What is the fake company name used in the email?**

![fake company name.png](/assets/fake-company-name.png)

**Business Finance Ltd.**
### Task 8
**What is the name of the attachment included in the email?**

![name of attachment.png](/assets/name-of-attachment.png)

**Invoice_2025_Payment.zip**
### Task 9
**What is the SHA-256 hash of the attachment?**
To do this without downloading the file, first we create a base64 file. This is where we will store the base64 content from our email attachment.
```bash
touch attachment.b64
```
Next add the content to your created file.
```bash
nano attachment.b64
```
Decode the base64 file and output the file into an archive. (we already know the attachment is a zip file).
```bash
base64 -d attachment.b64 > attachment.zip
```
Finally check the hash of the attachment.
```bash
sha256sum attachment.zip
```

![[sha256 attachment.png]](/assets/sha256-attachment.png)

**8379C41239E9AF845B2AB6C27A7509AE8804D7D73E455C800A551B22BA25BB4A**
### Task 10
**What is the filename of the malicious file contained within the ZIP attachment?**
lets list the files in our zip attachment.
```bash
7z l attachment.zip
```

![malicious file.png](/assets/malicious-file.png)

**invoice_document.pdf.bat**
### Task 12
**Which MITRE ATT&CK techniques are associated with this attack?**
This is a phishing attack with a malicious attachment.
link to MITRE attack technique: https://attack.mitre.org/techniques/T1566/001
**T1566.001**

## Last word
That is the end of this walkthrough. It was fairly easy but there is always something to learn. Adios!