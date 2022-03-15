---
layout: layouts/post.njk
title: Repeating block XOR Encryption - CTF Writeup
socialImage: ""
date: 2022-03-15T21:14:20.412Z
tags:
  - ctf
  - 247ctf
  - crypto
---
# Foreword

I've decided that I need to stretch my security brain again, and after watching a number of [LiveOverflow's](https://www.youtube.com/liveoverflow) post-CTF write-up videos, I thought I'd start with some practice CTFs.

For the uninitiated, CTF stands for Capture the Flag. They're essentially competitions which invite people to think creatively and solve security challenges in order to "steal the flag". Typically these can be "Attack/Defence" challenges, where you have to protect your own flag while trying to steal the other team's, or they can be Jeopardy-style where you steal from the organiser with pre-set problems to solve. CTFs tend to be attached to big events with a limited lifespan, such as [DEF CON](https://defcon.org/) or [BSides](https://www.securitybsides.org.uk/).

I've found myself most comfortable with Jeopardy-style CTFs, mostly because I'm not in it to win it, I'd much rather pick and choose what I'm going to work on and be happy when I solve it. 

I recently came across [247CTF](https://247ctf.com), a moderately new site which hosts a number of different flag challenges across different categories with their difference being, their challenges are always live. This is where I'll start out.

## Challenge Info

```
+----------------+--------------------------------------------+
| CTF Name       | 247CTF                                     |
| Challenge Name | An Exclusive Key                           |
| Description    | Decrypt the provided file to find the flag |
| Category       | Cryptography                               |
| Points         | 200                                        |
+----------------+--------------------------------------------+
```

In this challenge, we're given an encrypted file and we're asked to find the encryption password and decrypt the file.

![Text reads: "We put together a substitution-flag-permutation network. We encrypted the flag with the network, but forgot to write down the key. Can you reverse the network and recover the flag plaintext?"](/images/screenshot-2022-03-15-at-21.09.34.png "Challenge description")


## Analysis

First things first, 

