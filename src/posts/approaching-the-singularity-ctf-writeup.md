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

At a quick glance, there's nothing actually being loaded remotely in terms of scripts, i.e. no JavaScript. The only thing that's loaded is the CSS, which isn't interesting at all.

![HTML source code of page](/images/carbon-1-.png)

Given that there's nothing of interest in the head, let's focus our attention on the body. The body of the page is, again, very simple - only loading an image from `mturk.php` and a form which on submission, sends a `POST` request to the page with the data entered into the text field. 

We can now rule out any form of JavaScript nonsense and focus our attention on what data is actually sent to and received from the server. Setting up a HTTP Proxy which intercepts requests is the best way to do that, and we'll use Burp Suite for that.

### Burp Suite

1. Loading the page - `GET /`

   ![](/images/carbon-2-.png)
2. Response

   ![](/images/carbon-3-.png)
3. Sending data - `POST /`

   ![](/images/carbon-4-.png)
4. Response

   ![](/images/carbon-5-.png)

From the two endpoints we've been able to hit, we've been able to see two pieces of evidence that may help us in solving this. 

1. The server is running on NGINX (not super helpful)
2. The site stores a PHP Session cookie (probably useful)

Knowing this, we're more likely to be able to programmatically solve the captchas. 

If we move our attention to `mturk.php`, the file which is rendered as an image, we'll see that it is actually just a PNG file which is generated on each call to the page.

![](/images/carbon-6-.png)