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

![Text reads: "We XOR encrypted this file, but forgot to save the password. Can you recover the password for us and find the flag?"](/images/screenshot-2022-03-15-at-22.00.25.png "Challenge description")

## Method

First things first, let's see what info we can glean or assumptions (we'll be seeing a lot of those) we can make before actually getting into solving the problem.

![Buzz Lightyear from Toy Story telling Woody, "Assumptions. Assumptions everywhere"](/images/assumptions-assumptions-everywhere-c2ci7z.jpeg "Yes. they will be everywhere. So sue me.")

We're told that the file was encrypted with XOR which narrows down the problem a lot and lets us consider some reusable techniques from other CTF/cryptoanalysis problems - such as block size detection using normalised Hamming distance. Let's try that out first.

### Step 1 - Work out the normalised Hamming distance of a range of block sizes

In order to work out the block size of the XOR key, we need to:

1. Split the cipherblock into n-length blocks, where n is in the range of 1 to some higher number. Let's pick 256.
2. Work out the Hamming distance of the first couple of n-length blocks, and take the average. 
3. Normalise the value by dividing it by the length of the block, that way values will be relative to each other, and it'll be easier to work through later.
4. Take the lowest 4-5 resulting distances. One of these is likely to be your key length.

Calculating the Hamming distance of values is surprisingly easy given we're working with binary. Just XOR the two values, and then count the number of 1s in the result.

An illustrated example would look like:
```
(15)       (109)       (98) 
00001111 ^ 01101101 =  01100010 -> Hamming Distance = 3
```

So it's time to code that up:
```python
def xor(a, b):
    return bytes([i ^ j for i, j in zip(a, b)])

def hamming_distance(a, b):
    return bin(int(xor(a, b).hex(), 16).count('1')
```