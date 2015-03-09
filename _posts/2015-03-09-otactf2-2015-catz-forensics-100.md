---
layout: post
title: "OTACTF2 2015 - Catz - Forensics 100"
description: ""
category: ctf
tags: ['otactf', 'forensics']
author: roguecoder
---
{% include JB/setup %}


Opening this challenge we were given the following text

*I have some pictures of catz on my drive! I also had a sensitive file on there, but even after it was deleted, it was still there! I took some extra precautions to prevent prying eyes...*

<!---->

This was the first time I've done a challenge like this, so I wasn't really sure what to expect, so I downloaded and extracted the file.

```
> tar xvf catz.tar.bz2
catz.img
```

There was only one file in the archive, and judging from the `.img` extension I understood that this had to be some sort of image file. Also, based on the description I thought it had to be an image of a hard drive. To see what I was dealing with I ran the file command

```
> file catz.img
catz.img: Linux rev 1.0 ext4 filesystem data, UUID=2d362e1b-69ae-4137-bdbb-4fde2775ac91 (extents) (huge files)
```

Yup, this is a file system image... Ì mounted it to see what was there.

```
> mount catz.img /mnt/tmp/

> ls -la /mnt/tmp/
total 7420
drwxr-xr-x 3 root root  1024 Mar  5 02:10 .
drwxr-xr-x 4 root root  4096 Mar  5 23:51 ..
-rw-r--r-- 1 root root   176471 Mar  5 02:02 cat2.jpg
-rw-r--r-- 1 root root  87939 Mar  5 02:02 cat3.jpg
-rw-r--r-- 1 root root  54082 Mar  5 02:02 cat4.jpg
-rw-r--r-- 1 root root   116500 Mar  5 02:02 cat5.jpg
-rw-r--r-- 1 root root  73366 Mar  5 02:03 cat6.jpg
-rw-r--r-- 1 root root  68140 Mar  5 02:03 cat7.jpg
-rw-r--r-- 1 root root  49577 Mar  5 02:03 cat8.jpg
-rw-r--r-- 1 1000 users 2014332 Mar  5 02:08 catcuddle.gif
-rw-r--r-- 1 root root   511441 Mar  5 02:03 catdog.gif
-rw-r--r-- 1 1000 users 1124170 Mar  5 02:05 catfunnyface.jpg
-rw-r--r-- 1 root root   823535 Mar  5 02:04 catgif.gif
-rw-r--r-- 1 root root  64796 Mar  5 02:02 cat.jpg
-rw-r--r-- 1 root root  71428 Mar  5 02:04 catreindeer.jpg
-rw-r--r-- 1 root root  2093456 Mar  5 02:04 catsipsip.gif
-rw-r--r-- 1 1000 users  207633 Mar  5 02:10 catwindow.jpg
-rw-r--r-- 1 root root  36115 Mar  5 02:04 catyum.gif
drwx------ 2 root root  12288 Mar  5 02:01 lost+found

> ls -la /mnt/tmp/lost+found/
total 13
drwx------ 2 root root 12288 Mar  5 02:01 .
drwxr-xr-x 3 root root  1024 Mar  5 02:10 ..
```

Soooooo, what the hell to do next? I read the description again. Some files had been deleted. Ok, so these files are probably what we want to recover. This brought me to my next question. *How the hell do we recover deleted files!?*

I went out for a smoke and thought about the challenge, when I suddenly realized I had seen a challenge much like this not too long ago actually. I remembered some chatter about a tool called *extundelete*. I checked if it was installed in Kali, which it was. I executed the `extundelete` command with no options guessing it would give me a help screen, which it did.

So, the usage syntax for this tool is...

```
Usage: extundelete [options] [--] device-file
```

... and after reading through the rest of the text, this is what looked the most interesting

```
  --restore-all          Attempts to restore everything.
```

With this information I tried executing the following command

```
> extundelete --restore-all catz.img
```

What the fuck!? Segfault!? extundelete crashed.. Then I remembered I had heard that the installation in Kali wasn't working correctly. So I compiled it on my host machine, and moved the image to the shared folder and tried it again from the host.


```
> extundelete --restore-all catz.img
NOTICE: Extended attributes are not restored.
Loading filesystem metadata ... 2 groups loaded.
Loading journal descriptors ... 146 descriptors loaded.
Searching for recoverable inodes in directory / ...
2 recoverable inodes found.
Looking through the directory structure for deleted files ...
1 recoverable inodes still lost.
```

That's more like it! `2 recoverable inodes found.` ok? So does this mean I recovered anything? Let's see if it has spit out anything.

```
> ls -la
total 17624
drwxr-xr-x 3 rogue rogue    4096 Mar  8 01:47 .
drwxr-xr-x 5 rogue rogue    4096 Mar  8 01:41 ..
-rw-r--r-- 1 rogue rogue 10485760 Mar  8 01:47 catz.img
-rw-r--r-- 1 rogue rogue  7546151 Mar  8 01:41 catz.tar.bz2
drwxr-xr-x 2 rogue rogue    4096 Mar  8 01:56 RECOVERED_FILES
```

Well well well. A new directory named RECOVERED_FILES has been made. Let's see if it has anything in it.

```
> ls -la RECOVERED_FILES/
total 24
drwxr-xr-x 2 rogue rogue  4096 Mar  8 01:56 .
drwxr-xr-x 3 rogue rogue  4096 Mar  8 01:47 ..
-rw-r--r-- 1 rogue rogue    54 Mar  8 01:56 .cat.jpg
-rw-r--r-- 1 rogue rogue 12288 Mar  8 01:56 file.17
```

So a hidden picture of a cat, and some random file. That random file looks the most interesting right now, so let's cat it to see if it is as interesting as I hope.

```
> cat RECOVERED_FILES/file.17
b0VIM 7.4�
U3210#"! Utpad�f l a g { f u g l y _ c a t s _ n e e d _ l u v _ 2 }
```
Indeed! There was the flag!

**Flag:** flag{fugly_cats_need_luv_2}