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

Looks easy enough. So it's time to code that up. We'll be doing this in Python, and the full code is available on [Gitlab](https://gitlab.com/boltovnya/247ctf).

The first difficult we encounter is Python's handling of binary data, and managing representations of it. It's doable, but requires long chains of parsing of different datatypes, so the code can sometimes look a little... heavy. This is only a problem as far as the Hamming distance and Xor code goes.

```python
def xor(a, b):
  x = [i ^ j for i, j in zip(a, b)]
  return bytes(x)

def hamming_distance(a, b):
  """
  The easiest way to work out the number of 1s in a byte in 
  Python is to convert it to a hex string, parse that as a number,
  convert that to Binary and then count the number of "1s".
  """
  return bin(int(xor(a, b).hex(), 16).count('1')
```

The next step is to read out file, and break it up into n-length blocks.  We can achieve this by taking slices of the file at n-stop increments, and creating a new list of these blocks.

While we're at it, we can also write our scoring function, which just uses what we've written already to build a dict of possible block sizes and their associated normalised Hamming distance. 

```python
from statistics import mean

fo = open("exclusive_key", 'rb')
f = fo.read()

def xor...
def hamming_distance...

def bin_split(b, size):
  return [b[i: i + size] for i in range(0, len(b), size)]

def scoring(f):
  scores = {}
  for i in range(1, 256):
    split = bin_split(f, i)
    h = [] # Empty list for storing the score for each round of comparison.
    for j in range(1, 6):
      h.append(hamming_distance(b[0], b[j]))
    havg = mean(h) # Averaging the results
    score = havg/i # Normalising the result
    scores.update({i: score})
    
  top1 = min(scores.items(), key=lambda x: x[1]) # Gets the lowest value of the dict
  del scores[top1[0]] # Deletes the lowest value so we can work out the second lowest
  top2 = min(scores.items(), key=lambda x: x[1])
  del scores[top2[0]]
  top3 = min(scores.items(), key=lambda x: x[1])
  del scores[top3[0]]
  top4 = min(scores.items(), key=lambda x: x[1])
  del scores[top4[0]]
  top5 = min(scores.items(), key=lambda x: x[0])
  
  print(f"#1 - {top1}\n#2 - {top2}\n#3 - {top3}\n#4 - {top4}\n#5 - {top5}")
```