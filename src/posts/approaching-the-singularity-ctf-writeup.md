---
layout: layouts/post.njk
title: Approaching the Singularity - CTF Writeup
socialImage: /images/owen-beard-k21dn4ovxnw-unsplash.jpg
date: 2022-03-22T15:53:29.732Z
tags:
  - ctf
  - ai
  - python
  - captcha
  - flask
  - tensorflow
---
## Challenge Info

This one was incredibly tough, and required me to experiment with new ways to solve problems and automate tasks. I'm quite proud of the result, and you'll hopefully see why by the end of the post.

```
    +----------------+--------------------------------------------+
    | CTF Name       | 247CTF                                     |
    | Challenge Name | Mechanical Turk                            |
    | Description    | Solve 100 captcha equations in 30 seconds  |
    | Category       | Web                                        |
    | Points         | 310                                        |
    +----------------+--------------------------------------------+
```

![If you can solve our custom CAPTCHA addition equation 100 times in 30 seconds you will be rewarded with a flag.](/images/screenshot-from-2022-03-22-15-59-10.png)

# Part 1 - Investigation

![Web page titled "Mechanical Turk" with a CAPTCHA, text input field and submit button](/images/screen-shot-2022-03-22-at-16.28.57.png)

On starting the challenge instance, we're met with a simple web page with a CAPTCHA and a form consisting of an input box and a submit button. Given that the page is so plain, let's check out the source and see if there's anything that could give us insight into how the CAPTCHA is generated, and if we can modify anything.



![HTML source code of page](/images/carbon-1-.png)