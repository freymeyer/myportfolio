---
title: 'MacroHard WeakEdge'
date: '2025-10-31T11:30:21+08:00'
lastmod: '2025-10-31T11:30:21+08:00'
draft: true
description: ''
summary: ''
tags: ["Medium", "Forensics", "picoCTF 2021"]
categories: []
series: []
author: "Zeus Dela Cruz"

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
  image: 'https://w.wallhaven.cc/full/qd/wallhaven-qdj5dr.jpg' # /images/filename.jpg
  alt: ''
  caption: ''
  relative: false
  hidden: false
  hiddenInList: true
  hiddenInSingle: false

# Canonical link (for SEO)
canonicalURL: ''

# External edit link
editPost:
  URL: "https://github.com/freymeyer/myportfolio/tree/main/content"
  Text: "Suggest Changes"
  appendFilePath: true
---
Unzip the pptm file by either using

```
unzip or binwalk -e
```

It shows us multiple files, since the title is 'MacroHard WeakEdge' I assume it has something to do with the .vba file

But turns out, it was a decoy or a red herring. 

I used fuzzy finder to look more files

```
fzf --filter='!.xml.rel'
```

And saw a file named hidden, it outputs an encoded base64, so we only need to decode it.