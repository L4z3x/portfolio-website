---
title: "Weaknesss: Threat Intel challenge"
published: 2025-10-26
description: 'a detailed write-up of my first threat intel challenge from nextrace event.'
image: ''
tags: ["ThreatIntel","Easy","MyChallenges"]
category: 'CTF'
draft: false 
lang: ''
---

## Description
Welcome to the threat intelligence challenge.  
I have a hash of malware that attacked my friends overseas, I need to gather more infos about it, so i can help them.  
can you help me ? i heard virus total is a good place to start.  
**malware Hash**: `9b162f43bcbfaef4e7e7bdffcf82b7512fac0fe81b7f2c172e1972e5fe4c9327`
download the solve script to answer the questions.<br/>
**questions**:
  > 1. what file format was used in the delivery of the malware ?  
  > 2. what language was used to write the malware ?  
  > 3. what is the name of the executable that was used in the attack ?  
  > 4. what is the name of the apt group that carried out the attack ?  
  > 5. what was the name of the TV character referenced in the c2 communincation of the malware that was used to download files and write them to a folder ?
**Author** : L4Z3X
this is a basic threat intel challenge.

### Q1: what file format was used in the delivery of the malware ?
first paste the hash in virustotal, in the top section you will find the file type is `rar`

### Q2: what language was used to write the malware ?
in the Community section you will find a comment with a blog url, when you visit the url, you will find that the title is "Delphi Used To Score Against Palestine" so the answer is `delphi`

### Q3: what is the name of the executable that was used in the attack ?
in the previous blog post, it was mentioned that the archive contained a single executable named:`InternetPolicy_65573247239876023_3247648974234_32487234235667_pdf.exe`

### Q4: what is the name of the apt group that carried out the attack ?
for this question, you need to take the name of the malware from the blog post: `Micropsia` and search it on [mittre&attack](https://attack.mitre.org/), once you're in micropsia page, scroll down to "Groups that used this Software" and you will find that the group is `APT-C-23`.

### Q5: what was the name of the TV character referenced in the c2 communincation of the malware that was used to download files and write them to a folder ?
for this question you can find the answer on the previous blog post, but it's not in detail, however if you search for micropsia on google, you will find this [blog post](https://www.radware.com/blog/security/micropsia-malware/), scroll down to bot registration and you will find all json keys and description, the key that was used to download files and execute them is `mikasa` among other three.


## Flag
paste all the answers in the solve script and run it, you will get this flag:
`nexus{thr34t_1nt3l_0n_p4l3st1n3_3n3my}`
