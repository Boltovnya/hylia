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

Knowing this, we're more likely to be able to programmatically solve the CAPTCHAs, as we'll need the session ID to tie our `POST` requests to the CAPTCHA image we've received.

If we move our attention to `mturk.php`, the file which is rendered as an image, we'll see that it is actually just a PNG file which is generated on each call to the page.

![](/images/carbon-6-.png)

A couple calls to `mturk.php` proves this and yields these results.

![](/images/combined.png)

With no obvious headers or request body items to influence what CAPTCHA is actually generated on each call, we can easily assume that we can't make the CAPTCHA generate what we want and that it's done randomly each time the endpoint is hit. 

This is going to be tricky.

## CAPTCHA Background

If you cast your mind back to the start of this blog, you'll see that CAPTCHA is actually an acronym (or maybe a backronym) for "**C**ompletely **A**utomated **P**ublic **T**uring test to tell **C**omputers and **H**umans **A**part". To truly understand what that means, let's quickly describe what a Turing Test is. 

![](/images/tfg2etqk9yjnqurrwyhcdg.jpg "\\\\\"Good job! As part of a required test protocol, we will stop enhancing the truth in three... Two... One.\\\\\"")

A Turing Test is a method of testing whether or not an Artificial Intelligence (AI) is capable of thinking like a Human. During the test, a Human test subject and an AI interact with a Human observer who is asking questions. The AI and the human are both asked the same questions, within the same specific subject and the same format. The observer, after having asked the questions must then determine which answers were the Human's, and which were the AI's. This process is repeated a number of times, and the AI is deemed to have passed the Turing Test if the observer is incorrect 50% of the time or more. To date, no computer or AI has been able to pass the Turing test.

CAPTCHA's purpose is to tell Humans and Computers apart. There are a number of great reasons why you should do this, but mainly it's there to prevent Botting - the running of tasks in a repeated manner by a computer, sometimes maliciously.

You, the ~~test subject~~ reader, have undoubtedly come across CAPTCHAs in one way or another. If you're signing up to a new website, using Google over a company VPN, or have simply visited a web page one too many times.

CAPTCHAs are specifically designed to be unreadable by OCR (Optical Character Recognition) tools, meaning that Computers should not be able to read them. They make use of warped characters on low-contrast backgrounds, sometimes with arbitrary shapes around to make this task even more difficult. Upon completion of a CAPTCHA, the web application knows with some degree of confidence that the user attempting to access it is in fact a human.

Except that's not so true nowadays.

# Part 2 - Neural Networks

Yes, it's a bit of a buzzword nowadays, but Artificial Intelligence, Machine Learning and Neural Networks are terms that are at the forefront of modern computing, and they'll be around for a while. The only issue is, [Corpo rats](https://www.urbandictionary.com/define.php?term=Corpo) have gotten their hands on it and are applying it to everything, without really understanding what it is or how it works. [AI-powered Washing Machines](https://news.samsung.com/global/samsung-introduces-ai-powered-laundry-lineup-with-top-tier-energy-efficiency-new-refrigerators-customizable-for-every-lifestyle), [ML for trading](https://github.com/stefan-jansen/machine-learning-for-trading), it's all a bit reminiscent of when IoT first became a thing. Who needs an internet connected fridge that can think?

<center><iframe width="560" height="315" src="https://www.youtube.com/embed/bZe5J8SVCYQ" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe></center>

Rant over. While AI as a subject is waved around as a sales pitch, it can actually be quite useful. Pattern recognition in images is such an incredibly broad field with millions of potential applications. Diagnosing illness with medical scans, image description for the sight impaired, collision detection in self-driving cars and even hand tracking for VR/AR (yay, more buzzwords) applications are just a small number of applications that are in place in the real world that are powered by Neural Networks. 

Previously, these would have required absolutely massive data sets to train from, and an inordinate amount of time to perform the training and subsequent inference or predictions, but with advances in parallel processing, both of these variables have been vastly reduced. Modern frameworks like Tensorflow and Keras can transparently make use of GPU cores to accelerate the parellel processing workloads required to build Neural Network models. But you didn't come here for a lesson or a sales pitch.

- - -

The only way I could realistically see myself solving this challenge was either paying a bunch of Comp Sci students in pizza to access a proxied version of the challenge site (i.e. sharing the PHP Session Token), and hoping they'd be fast enough to beat the 30 second limit, or building a prediction model which could accurately "read" the CAPTCHA text and solve the equation given.

I went for the latter option, as I'm broke and I don't think I could trust students without being in the room with them.

And so I went off and attempted to build a model. I ended up using [Keras' own example](https://keras.io/examples/vision/captcha_ocr/) for CAPTCHA OCR, which cut out most of the hard work for me. I won't bother adding the code for the model to this blog, as it's long, verbose and it's not my own work. You'd also get a better appreciation for it by following the Keras example. 

This is where I ran into my first major issue. The Keras example assumes you already have a training dataset ready to plug into the model, based on an already existing library of CAPTCHA images, with each being labeled with the CAPTCHA text. Great, I thought, and then I realised `mturk.php`'s text was too different to be able to infer from that dataset.

So I had to build that one myself...

# Part 3 - Training

And off we go, a seemingly endless cycle of "Right-click, Save Image, Name the file, Refresh, Repeat". After about 45 mins, I'd amassed a grand total of 60 images. That should be enough to train the model, right?

Firstly, let's make sure that our dataset looks right. That's a great starting place to make sure that what we're working with makes sense to not only the computer, but also to us once we're done with it.

![](/images/input-dataset-plot.png "Figure 1 - Plot of training input")

Giving the dataset 100 Epochs (100 rounds of passing **all** data through the model once, forwards and backwards, to train the model to fit the expected output), and 60 images to work from, we get this output which I like to call Gen 0.

![](/images/estimates.png "Figure 2 - Gen 0 output (AKA Zalgo)")

I don't know about you, but getting any more images than that using the "right-click, save-as" method sounds like a rough estimation of what punishments await me in hell. There had to be a better way to do it, but regardless there had to be some human input at some point, purely by virtue of us not yet having a system to read the images. That's what we're building, after all.

So I broke out the Python and wrote a little app to make my life easier.

## Python App

The easiest way I thought of setting this up was by using a Flask app with two endpoints, one to display the page, and another to send the filename and image data to. Let's code that as we go.

<center><iframe
  src="https://carbon.now.sh/embed?bg=rgba%28182%2C162%2C145%2C1%29&t=zenburn&wt=bw&l=jsx&width=800&ds=true&dsyoff=20px&dsblur=68px&wc=true&wa=false&pv=56px&ph=56px&ln=true&fl=1&fm=Hack&fs=14.5px&lh=144%25&si=false&es=2x&wm=false&code=from%2520flask%2520import%2520Flask%250A%250Aapp%2520%253D%2520Flask%28__name__%29%250A%250A%2540app.route%28%2522%252F%2522%29%253A%250Adef%2520home%28%29%253A%250A%2520%2520return%2520%2522Hello%2520World%2522%250A%250A%2540app.route%28%2522%252Fpost%2522%252C%2520methods%253D%255B%27POST%27%255D%29%250Adef%2520post%28%29%253A%250A%2520%2520return%2520%2522Post%2522%250A%250Aapp.run%28%29"
  style="width: 750px; height: 461px; border:0; transform: scale(1); overflow:hidden;"
  sandbox="allow-scripts allow-same-origin">
</iframe></center>

We have the bare-bones now, but we also need a webpage to display and interact with. Using Flask, we can actually do this in Jinja quite easily

<center><iframe
  src="https://carbon.now.sh/embed?bg=rgba%28182%2C162%2C145%2C1%29&t=zenburn&wt=bw&l=jsx&width=800&ds=true&dsyoff=20px&dsblur=68px&wc=true&wa=false&pv=56px&ph=56px&ln=true&fl=1&fm=Hack&fs=14.5px&lh=144%25&si=false&es=2x&wm=false&code=%253C%21DOCTYPE%2520html%253E%250A%253Chtml%2520lang%253D%2522en%2522%253E%250A%2520%2520%2520%2520%253Chead%253E%250A%2520%2520%2520%2520%2520%2520%2520%2520%253Cmeta%2520charset%253D%2522utf-8%2522%253E%250A%2520%2520%2520%2520%2520%2520%2520%2520%253Ctitle%253EAI%2520Indexer%253C%252Ftitle%253E%250A%2520%2520%2520%2520%253C%252Fhead%253E%250A%2520%2520%2520%2520%253Cbody%253E%250A%2520%2520%2520%2520%2520%2520%2520%2520%253Ch1%253EAI%2520Indexer%253C%252Fh1%253E%250A%2520%2520%2520%2520%2520%2520%2520%2520%253Cimg%2520src%253D%2522%257B%257B%2520image%2520%257D%257D%2522%252F%253E%250A%2520%2520%2520%2520%2520%2520%2520%2520%253Cform%2520action%253D%2522%252Fpost%2522%2520method%253D%2522post%2522%253E%250A%2520%2520%2520%2520%2520%2520%2520%2520%2520%2520%2520%2520%253Cinput%2520type%253D%2522text%2522%2520%250A%2520%2520%2520%2520%2520%2520%2520%2520%2520%2520%2520%2520%2520%2520name%253D%2522filename%2522%2520%250A%2520%2520%2520%2520%2520%2520%2520%2520%2520%2520%2520%2520%2520%2520autofocus%2520%252F%253E%250A%2520%2520%2520%2520%2520%2520%2520%2520%2520%2520%2520%2520%253Cinput%2520type%253D%2522submit%2522%2520%250A%2520%2520%2520%2520%2520%2520%2520%2520%2520%2520%2520%2520%2520%2520value%253D%2522Submit%2522%2520%252F%253E%250A%2520%2520%2520%2520%2520%2520%2520%2520%253C%252Fform%253E%250A%2520%2520%2520%2520%2520%2520%253Cp%253E%257B%257B%2520i%2520%257D%257D%2520files%253C%252Fp%253E%250A%2520%2520%2520%2520%253C%252Fbody%253E%250A%253C%252Fhtml%253E"
  style="width: 700px; height: 587px; border:0; transform: scale(1); overflow:hidden;"
  sandbox="allow-scripts allow-same-origin">
</iframe></center>

So now we have a super simple page which is dedicated to filling in CAPTCHAs, as well as showing how many images you've spent your time filling in. It's like a really shit cookie clicker.

![](/images/screen-shot-2022-03-23-at-01.12.26.png "I'd play this, would you?")

Getting the UI and endpoints set up was the easy part, the hard part was now actually performing the process of:

1. Displaying the image
2. Taking the filename
3. Writing the image to disk with the filename

all while making sure we're working on the same image. Remember, each time `mturk.php` is called, a new image is generated, so we can only call down the image once.

If we break down each step, the task seems a lot more manageable than at first glance.

Rendering the Jinja template seemed difficult, as you can't pass binary data to HTML and render it as an image. Then I remembered that you can, but only if you Base64 encode the image data into plaintext. In order to do this, we need to edit the j2 template we made previously so that the `img` tag knows how to read the data.

<center><iframe
  src="https://carbon.now.sh/embed?bg=rgba%28182%2C162%2C145%2C1%29&t=zenburn&wt=bw&l=jsx&width=700&ds=true&dsyoff=20px&dsblur=68px&wc=true&wa=false&pv=56px&ph=56px&ln=true&fl=1&fm=Hack&fs=14.5px&lh=144%25&si=false&es=2x&wm=false&code=...%250A%253Cimg%2520src%253D%2522data%253A%253Bbase64%252C%257B%257B%2520image%2520%257D%257D%2522%252F%253E%250A..."
  style="width: 700px; height: 253px; border:0; transform: scale(1); overflow:hidden;"
  sandbox="allow-scripts allow-same-origin">
</iframe></center>

Except now the issue is how do we pass the image data from the `GET` route, to the `POST` route that saves the file. The answer is, send it with the `POST` request. You can create a hidden input which sends the image data to the `POST` endpoint, which can then be decoded and turned back into an image. 

<center><iframe
  src="https://carbon.now.sh/embed?bg=rgba%28182%2C162%2C145%2C1%29&t=zenburn&wt=bw&l=jsx&width=700&ds=true&dsyoff=20px&dsblur=68px&wc=true&wa=false&pv=56px&ph=56px&ln=true&fl=1&fm=Hack&fs=14.5px&lh=144%25&si=false&es=2x&wm=false&code=...%250A%253Cinput%2520type%253D%2522hidden%2522%2520%250A%2520%2520name%253D%2522image%2522%2520value%253D%2522%257B%257B%2520image%2520%257D%257D%2522%2520%252F%253E%250A..."
  style="width: 700px; height: 274px; border:0; transform: scale(1); overflow:hidden;"
  sandbox="allow-scripts allow-same-origin">
</iframe>
</center>

So we've worked out how to display and pass data from the HTML perspective, let's actually get these routes doing something.

A few more imports gets us all the tools we need to build this app

<iframe
  src="https://carbon.now.sh/embed?bg=rgba%28182%2C162%2C145%2C1%29&t=zenburn&wt=bw&l=python&width=700&ds=true&dsyoff=20px&dsblur=68px&wc=true&wa=true&pv=56px&ph=56px&ln=true&fl=1&fm=Hack&fs=14.5px&lh=144%25&si=false&es=2x&wm=false&code=from%2520flask%2520import%2520Flask%252C%2520render_template%252C%2520request%252C%2520redirect%252C%2520url_for%250Afrom%2520base64%2520import%2520b64encode%252C%2520b64decode%250Aimport%2520requests%250Aimport%2520os"
  style="width: 1024px; height: 473px; border:0; transform: scale(1); "
  sandbox="allow-scripts allow-same-origin">
</iframe>



```python
# Additional imports for needed functionality
from flask import Flask, render_template, request, redirect, url_for
import requests
from base64 import b64encode, b64decode
import os
...
@app.route("/")
def home()
  r = requests.get("https://coolctf.example/mturk.php")
  img = r.content
  b64 = b64encode(img).decode("utf-8")
  files = len(os.listdir('./images'))
  return render_template('index.j2', image=image, i=num_files)

@app.route("/post", methods=['POST'])
def post():
  filename = request.form["filename"]
  image = request.form["image"]
  
  with open(f"./images/{filename}.png", "wb") as wb:
    wb.write(b64decode(image))
  return redirect(url_for('home'))
```