---
title: Adventures in Eurorack | Pt.1
data: '2021-01-03'
tags:
  - music
  - hobby
  - homelab
  - blog   

---

## Happy new year!

> It's been a long time since my last post, and I'm not convinced I'll ever actually finish writing the Smart Snek post (project no longer necessary, and it's all but been retired now), but a lot has changed since then. The world is almost unrecognisable now compared to a year ago and I'll be honest, it's scary. That being said, I'm not gonna tell you that you have to be brave and welcome the new world with open arms; it's okay to not be okay. Take some time to yourself, do something new, learn a language, learn to code, or just do nothing. Find what works for you in this 'new normal' and let it bring you joy.    
> For me, that's been learning Electrical Engineering and Sound Design, which is what this blog is all about. If you're really struggling with what's going on, reach out to someone. A loved one, a friend, or a charity setup to help people during crises. There's always help.   
Peace,  
Erin

---

## Kicking off
Back in June '19, I visited the [Swindon Makerspace](https://www.swindon-makerspace.org/) on their annual open day. There, I was shown around the awesome space they have, the tools at their disposal, as well as being introduced to the wonderful people who help make it happen. While there, I was introduced to the concept of modular synthesisers, and got to play around with one and  making some noise by plugging wires into different modules and twisting the knobs. After a while of jamming with some vague static and buzzing, the synth's owner introduced themselves to me and we got to talking about the process of building modules, designing a synth and what cases are available. 

Around a year or so later, the owner sent me a message asking if I was interested in having the synth. They were looking to get rid of it since, as with all great makers, they started the project and forgot about all it, leaving it to gather dust. I took it in a heartbeat and have been planning this blog series ever since.

Now that Christmas and New Years have been and gone, I decided to finally take the plunge and start designing the synth (and spending an unreasonable amount of money on circuits that go "buzz")

> Seriously, this shit ain't cheap

## Eurorack 101
If you're reading this post, you're one of two kinds of people. Those who know everything there is to know about Eurorack, or those who haven't even heard of it before.  

To cater for the latter, let's take a quick deep dive into the main concepts to know about eurorack.

### Modular Synthesisers
A modular synthesiser (or modular synth for short) is a electronic synthesiser made up of separate modules performing different functions. These modules can be connected together using patch cables to create a patch which can produce a sound specified by parameters on each module. 

Each module tends to do one or multiple functions within a single domain, and below covers most of types:
* Oscillators (VCO)
* Low-frequency Oscillators (LFO)
* Filters (VCF)
* Envelope Generators (EG)
  * Also known as ADSR (Attack-Decay-Sustain-Release)
* Clock Generator
* Logic
* Quantiser
* Sequencer

We'll explore what these actually mean later on in this post, but for now, that should cover most of what we need. The image below shows an example eurorack patch featuring these types of modules.

![Eurorack Patch](/images/eurorack_1/modular_patch.jpg "'Modular synthesizer - Jam Syntotek, Stockholm, 2014-09-09' by Henning Klokkeråsen is licensed under CC BY 3.0" )


### Controls
Synth modules are controlled by what are known as Control Voltage (CV), Gate and Trigger. 

#### CV  
CV controls parametric information, such as which note to play (known as Volts per Octave, or 1V/oct). CV can also automate the control of parameters usually controlled by a button or knob, such as volume, or the level of modulation or filtering to apply to a sound.

#### Gate
Gate is a binary signal which usually determines when an envelope should "open". When it does, it goes through its attack and decay phases before remaining at the sustain phase for the duration of the gate being high. When the gate goes low again, the envelope then moved onto its release stage. 

#### Trigger  
Trigger is a short pulse signal, rising to its high level for a few milliseconds. Triggers are used to start or "trigger" playback, clocks or any control that may be controlled with a momentary switch. They an also be used as an envelope gate for a very short sustain.

Audio is another signal used in Eurorack. It is what the ADSR envelope generator is applied to, but it is also, of course, the sound that we hear.

---

### Eurorack 
Now that we know the basics of Modular Synths, let's talk about eurorack.

Eurorack is a particular format of modular synth, first designed by Doepfer Musikelektronik back in the 90's with their A-100 synthesiser. Inspired by the Eurocard PCB format (popular with electronic lab equipment manufacturers), Eurorack as a standard is specified by its physical sizes whereby it uses rack units (U) as designated by the 19" rack standard. Modules are usually 3U (5.25in/133.4mm) tall, as with Eurocard, and the width is measured in Horizontal Pitch, where 1HP is equal to 0.2in (5.08mm). However, 1U and 6U modules do exist, with 1U modules becoming popular with Intellijel users.

![A-100](/images/eurorack_1/Doepfer_A-100.jpg "'Doepfer A-100' by Nina Richards is licensed under CC BY 3.0")

Depth is not standardised in eurorack, as cases can be of varying depths, allowing the user to build their own modules as deep as their case will allow. A typical DIY eurorack user may use a "skiff", which is a shallow and compact case which are often only 3U or 6U in height.

Eurorack also defines a standard for power supply and delivery, by using a 10 or 16 pin ribbon cable to carry dual rail 12VDC and single rail 5VDC. The power connector on modules is also standardised as a 10 or 16 pin IDC connector.

![Eurorack Power](/images/eurorack_1/Eurorack-Power-Pinouts.png "16 and 10 pin connectors are compatible if the connector isn't keyed.")

Audio and control signals as mentioned above are exchanged by 3.5mm mono jack cables (TRS), and carry voltages of between ±10V. 

Some Eurorack enthusiasts opt to use Banana plug cables, as these can be a cheaper way to stack cables across multiple modules when compared to stackable 3.5mm TRS cables. 

Eurorack has also inspired other formats for modular synths, namely the Kosmo format first defined by [Look Mum No Computer](https://www.lookmumnocomputer.com/modular). This format adheres closely to Eurorack, using the same power standards and control voltages. However, it makes use of larger modules and uses ¼" (6.3mm) jack cables. Modules are in the so-called Metric 5U format, where they are 20cm tall (as opposed to 22.25cm in regular 5U).

![LMNC Kosmo](/images/eurorack_1/lmnc_1114.jpg "Sam from LMNC's Kosmo #1114 Filter/Clipper/VCA")

>I really recommend checking out Look Mum No Computer's [Kosmo build series](https://www.youtube.com/watch?v=3q5JJWKzNno), he has some great designs on the go and his community on [Discorse](https://lookmumnocomputer.discourse.group/) is full of great minds.

---

### Affinity to hackers

Eurorack, and modular synthesisers as a whole, are a bit of an oddity in the music creation world in that there is a strong focus on the DIY community. Because of its standardised format, anybody can easily create (with some basic background knowledge of electronics) their own modules to fit their uses. In fact, a lot of DIY modules have developed into well renowned open source projects that anyone in the community is free to copy, contribute and modify as they see fit. 

You'll find a lot of popular module manufacturers publish the schematics, panel graphics, and firmware on public repositories like GitHub, including but not limited to; [Erica Synths](https://github.com/erica-synths/diy-eurorack), [Mutable Instruments](https://github.com/pichenettes/eurorack), [System80](https://github.com/minisystem/JOVE), and [Music Thing Modular](https://github.com/TomWhitwell/TuringMachine).

An interesting, but exciting side effect of open sourcing modules is that it introduces the possibility of simulating them in software. Enter VCVRack.

![VCVRack Modules Page](/images/eurorack_1/module-browser.png)

[VCVRack](https://vcvrack.com/Rack) is an amazing open source project which allows you to simulate your physical rack in software. You can try out patches, use MIDI more easily, and even use it to expand your physical Eurorack synth through CV-CC controlers. A lot of the open source eurorack manufacturers release their own modules as software modules, but others have been reverse engineered from the original hardware and firmware.

>If you're interested in trying out Eurorack, but don't want to wager a lot of money on it, definitely try out VCVRack. It's free, there are masses of free modules out there, and the integrated catalog allows you to install new modules with the click of a button.

