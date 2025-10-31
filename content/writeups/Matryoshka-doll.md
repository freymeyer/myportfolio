---
title: 'Matryoshka Doll'
date: '2025-10-31T11:05:29+08:00'
lastmod: '2025-10-31T11:05:29+08:00'
draft: false
description: 'Matryoshka dolls are a set of wooden dolls of decreasing size placed one inside another. Whats the final one? Image: this'
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
  image: 'https://w.wallhaven.cc/full/47/wallhaven-472jlo.jpg' # /images/filename.jpg
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

It gives us a file, "dolls.jpeg"

Looking at binwalk and strings, it has an image inside an image.

using

```
binwalk -e -M dolls.jpeg
```

It will recursively extract each extracted files and later on provides the flag txt file.

