---
title: Smart Snek Pt. 1
subtitle: aka Applied Ophiomancy
date: 2020-08-12T21:44:00.000Z
tags:
  - homelab
  - IoT
  - Edge
  - Messaging
  - SmartHome
---

### Preamble

So, it's been a while since I last posted. To give a quick update as to what I've done in the meantime - I've set up OpenShift 5 times at home, realised I didn't have enough memory to run a stable cluster, given up, and left things as they were post-DNS install. I've also looked at buying more memory, but that's gonna need to wait a while because 192GiB of ECC DDR3 is still £££. Anyways...


### What's this all about?

Smart Snek is a long-term project of mine to develop and build an IOT Home Automation system to look after snake husbandry. Now, I hear you asking, "but why?", and that's a really good question which requires a bit of a long explaination...

---
## Eva? Aioli? Eee-fee?

> Note, this section is a bit long, but it sets the context of the "Why"

In March of 2019, I was gifted a new pet snake. Now, normally I would have gone to the local pet store, picked out the usual pieces such as heat pads, water dish, substrate and a vivarium. But this snake is different... Let me introduce you to Aoife

![img](/images/aoife.jpg "/ˈiːfə/ EE-fə")

Aoife is very special, not only because she's my baby, but because she's also one-of-a-kind. You see, Aoife is a Puerto Rican Boa [(_C. Inornatus_)](https://en.wikipedia.org/wiki/Puerto_Rican_boa) and she was and remains to be, to my knowledge, the only one of her species to have ever entered the UK. 

Along with this, C. Inornatus is CITES I protected, meaning that no First Generation or Wild Caught snakes may be sold, therefore we had to abide by all in-place export controls despite being classified as Least concern by the IUCN Red List. There are some politics surrounding this, however that's mostly due to outdated US federal law put in place because of declining populations in the 1970s.

Because of all this, I'm putting extra effort into making sure everything is comfortable for her. Except, that's where another issue comes in. Because Captive Bred specimens are so rare, there aren't any care sheets or known standard practices available to reference.

This is where Smart Snek comes in.

### The plan

As with any Bring-Your-Own Smart Home plan, Smart Snek is primarily based on a bunch of connected things communicating over a common message broker. In this instance, using MQTT with Red Hat AMQ Broker. 

The phases of the project were originally in order of snake priorities, as follows:

1. Heating
2. Humidity
3. Water
4. Lighting
5. Food

So let's start with heating and humidity!

### Snug as a snek in a... rug... sure

Those of you who keep reptiles know how important heat and humidity plays into taking care of snakes. Those who don't, I'll give a brief explaination.

When it comes to the lifestyle of a snake, heating and humidity dictate a number of behaviours; when to eat, when to shed, when to sleep, when to wander around. With this in mind, ensuring that your snake has comfortable temperatures and humidity is key to guaranteeing a happy, healthy and active snake.

An even more important element that heat plays into is brumation. For the uninitiated, brumation is a hibernation-like state that cold blooded animals enter when it starts to get very cold. As the temperature decreases, reptiles slow down considerably and their heart rate drops, becoming extremely lethargic - to the point where they could be mistaken as dead. Brumation is dangerous outside of nature as their bodies are not adapted to the controlled temperature changes and if the temperature is not right, they may freeze before reaching brumation.

So now that we have that in mind, let's start with the most basic architecture...

---

## The tech

```
       +------------------+
       | Google Assistant |
       +------------------+
                |
         +-------------+
         |  Nobucasa   |                             +--------------+
         +-------------+                             |    Sonoff    |
                |                                   >|   (Tasmota)  |
                |                                  / +--------------+
                |                                -/          ^
         +-------------+            +---------+ /            |
         |   hass.io   |<---------->|   AMQ   |<             |
         +-------------+            +---------+      +--------------+
                                                     |    Si87021    |
                                                     +--------------+
       |  User Services  |         |  Backend  |    |    Hardware    |
                                   |  Services |     
```

Working up from the left, we've got the only physical pieces of the system so far - the Si7021 temperature/humidity sensor, and the Sonoff TH16 smart power relay. 

### Hardware

The Si7021 is a low cost Temperature/Humidity probe which works well for detecting ambient readings, being able to provide an accurate temperature from -10 to +85℃ within a margin of ±0.4℃, and high precision relative humidity with a region of 0-80%, within ±3%. Normally, this sensor would work over I²C, however the Sonoff does not support communication over I²C. The Sonoff TH16 exposes a 2.5mm TRS jack to allow for plug'n'play with sensors over one-wire communication. However, to allow for this, an 8-bit MCU is added to the board which changes the communication.

The Sonoff TH16 is an ESP8266 based Smart Power Socket, capable of switching peak loads of up to 4KW. This makes it ideal for heating applications due to the regular switching requirements. You can use the Sonoff out of the box with the included firmware and the eWeLink Smart Home app, however this is extremely limited in what it can do. You're tied into using a cloud hosted service which, to be frank, has fairly low security around it (I had passwords sent to me via email).

Instead, I use [Tasmota](https://tasmota.github.io/docs/). Tasmota is written to run on any ESP32/8266, but has specific bindings for the Sonoff range, including native thermostat functions. However, thermostat functionality was not available at the time this project was originally deployed and isn't used in this solution. Instead, that's offloaded to a Generic Thermostat component in HomeAssistant.

![img](/images/sonoff_th16.jpg "Small, cheap, unimposing")

The Sonoff itself is not anything impressive on the outside. Bland white case, wires running out of it and a button for reset. The mounting screw at the top is useful for hiding it all behind 

![img](/images/sonoff_naked.jpg "Clean and simple")

On the inside is where it gets a bit more interesting. At the very front you have the mains power bus, with the relay input and output alongside the earth. The input plug and output socket were actually salvaged from a cheap 1-gang extension lead, separated in the middle to allow for some flex between the outlet and the heater. The installation of the power components was super simple, with a spring loaded compression fitting on each bus connection.

Just to the left of the reset button, you've got the 2.5mm jack which exposes GPIO over 1-wire. There's not much to talk about there.

The exciting part is the serial IO at the top right of the board. The ESP flash on the board is not write protected, so you can flash any ESP8266 compatible firmware onto it. So with the help of an FTDI232, I was able to flash Tasmota onto the chip.

![img](/images/ftdi.jpg "A couple quid on amazon and works super well!")

Onto the sensor. It looks like any sensor you'd put into a snake vivarium, so it doesn't look out of place. I suppose that's all that can be said about that.

![img](/images/si7021.jpg "Yup, that's a sensor")

Inside the SI7021, there are a few interesting features. First and foremost is the sensor itself, which packs some hidden surprises. Should you get access to the chip directly, you can enable and adjust an internal heater to compensate for errors in readings or enable dew-point readings. Once activated, you have 4 bits of freedom to control the heater, up to a maximum of 94.2mA at 3.3V. I can't imagine needing this for this project, but it's good to know the option is there should I want to get into more humid environments (like the keeping of frogs 🐸)

![img](/images/si7021_naked.jpg "More than meets the eye")

Also on the board is an 8bit MCU, an SI EFM8BB10F2G-A-QFN20 to be precise. This is the chip that turns I²C into 1-wire. The specs aren't conducive to any impressive side-functions, with only 250 bytes of RAM and 2kB of flash, but the 12-bit, 15 channel ADC and 25MHz clock speed means it can handle this task well and reliantly.

Last but not least is the sneaky extra GPIO on the board. I don't plan to use this, ever, since it would mean hacking away at the casing - but it's really nice to see that should I want to sidechain any other sensors, that they can be attached directly to the board and run from there, rather than running additional wiring from the already fairly cumbersome Sonoff.

### Middleware

To handle the MQTT messaging, I opted to use Red Hat AMQ broker. 

![img](/images/RHAMQ.png)

A few factors played into my choice here, the obvious one being the free entitlement I get as part of being a Red Hat associate. Making use of the products I endorse and sell as part of my day job not only enables me to understand the product better, but also gives me faith in them. Can't sell something if you don't believe in it!

Secondly, a few key features of AMQ allow me to have greater confidence that it won't break. The main one there being message queue persistence, allowing messages to be queued in case of failure to ensure minimal risk to the snake should either the thermostat or the Sonoff fail. 

![img](/images/amq_ss.png)

Lastly, the interrogation of queues allows me greater control of the content being passed over MQTT. The AMQ web-ui allows you to browse any published topics on the broker and view data about number of messages processed, queued, pending and rejected. It's a little overkill for this project, but it was good to learn how to deploy and use AMQ as a service.