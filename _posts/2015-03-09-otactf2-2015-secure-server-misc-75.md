---
layout: post
title: "OTACTF2 2015 - Secure Server - Misc 75"
description: ""
category: 'ctf'
tags: ['otactf']
author: roguecoder
---
{% include JB/setup %}


In this challenge we were given the ip *104.131.124.226*. Visiting it in the browser returned nothing so I decided to give it a quick scan with nmap

<!---->

```
> nmap -sV -Pn -n 104.131.124.226

Starting Nmap 6.47 ( http://nmap.org ) at 2015-03-06 10:37 CET
Nmap scan report for 104.131.124.226
Host is up (0.12s latency).
Not shown: 998 closed ports
PORT    STATE SERVICE VERSION
22/tcp   open  ssh  OpenSSH 6.0p1 Debian 4+deb7u2 (protocol 2.0)
8080/tcp open  http nginx 1.2.1
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at http://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 8.50 seconds
```

Here we quickly see that the server has two open ports, SSH is running on the default port 22, and the webserver (nginx) is running on port 8080.

I opened my browser and navigated to http://104.131.124.226:8080/ and was greeted with this text

```
Welcome my secret website!

This is my secret site where I keep all of my secret.

Don't go snooping around for my secret, you might not like what you see...
```

The word **secret** repeats 4 times, so I was sure this was some sort of clue. I decided to see if there was a folder on the server named secret, so I added it to the URL, and in http://104.131.124.226:8080/secret/ I saw this

```
Index of /secret/

../
secret.zip                                      05-Mar-2015 18:42           2874766
```

I downloaded and extracted the zip file

```
> unzip secret.zip
Archive:  secret.zip
   creating: secret/
   creating: secret/.ssh/
  inflating: secret/.ssh/key
  inflating: secret/.ssh/key.pub
  inflating: secret/observation 19november32013.jpg
....
```

The file **secret/.ssh/key** caught my attention right away and I was sure this had something to do with gaining access to the SSH server. I changed the permissions of the file ...

```
> chmod 400 secret/.ssh/key
```

... and tried to connect to the server using the username **secret**

```
> ssh 104.131.124.226 -i key -l secret
Linux OTAMISC 3.2.0-4-amd64 #1 SMP Debian 3.2.65-1+deb7u1 x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
You have mail.
Last login: Fri Mar  6 09:43:47 2015 from vote.howl.volia.net
hello, welcome to my secret server
I thought somone may try to break into my server so i setup some extra protection
i dont want to give you a shell, but here is some ascii art for you:
Connection to 104.131.124.226 closed.
```

I successfully connected to the server, but it kicked me out immediately. From previous experience I know that some challenges has a exit command in the .bashrc file which will kick us out like we just saw here. To bypass this we can send a command along with the SSH connection string to rename the .bashrc so that it won't execute on the next attempt to connect.

To do this I add `"mv .bashrc"` to the end of the command

```
> ssh 104.131.124.226 -i key -l secret "mv .bashrc"
hello, welcome to my secret server
I thought somone may try to break into my server so i setup some extra protection
i dont want to give you a shell, but here is some ascii art for you:
#!/bin/bash

echo "hello, welcome to my secret server"
echo "I thought somone may try to break into my server so i setup some extra protection"
echo "i dont want to give you a shell, but here is some ascii art for you:"
shuf -n1 -e * | xargs less -E
```

Uhm, ok so, huh? What the hell is this!? Looks like it might be using a non-traditional shell. This bash script looks like another hint. I started reading up on the man pages for shuf, xargs and less. I was popping shells left and locally but every time I was trying it against the server it delivered another ASCII art, some of whihc were actually really funny. In the beginning.

Then suddenly one ASCII art caught my eye. I had seen this one many times earlier, but I was too caught up in trying to gain a shell so I didn't pay attention to it.

```
> ssh 104.131.124.226 -i key -l secret ".bashrc"
:!/bin/dash
/bin/pwd
hello, welcome to my secret server
I thought somone may try to break into my server so i setup some extra protection
i dont want to give you a shell, but here is some ascii art for you:
http://cl.ly/image/2c2a3O1T3q3j
/\
||_____-----_____-----_____
||   O                  O  \
||  O\\ ___ //O /
||      \\ /   \//      \
||      |_O O_|         /
||          ^ | ^       \
||      // UUU \\       /
||  O//         \\O   \
||   O                  O  /
||_____-----_____-----_____\
||
||.
```

The ASCII art looks trashed because I didn't keep the formatting when taking the notes, but it was a pirate flag. Yup, that's right. The ASCII art was a **flag** and it didn't strike me once that it could be a hint. But now, when it had caught my attention I looked at the entire message and noticed the link right above it.

This displayed an image, and the flag was written right there on the image.

**Flag:** flag{pUttHeBintheC}