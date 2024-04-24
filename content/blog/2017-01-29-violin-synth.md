---
title: The Making of the Fiddle Synth I
date: 2017-01-29
tags:
    - music
    - electronics
image: violin-synth/cover.jpg
---

My idea for a violin synthesizer came about from a Lau concert I recently went to.

Before the concert there were a few different workshops, one of which was a synthesizer making workshop run by Martin Green, the accordion player from Lau. Unfortunately I didn't make it along, but I did manage to see the concert where he uses synthesizers in just the right places to create an amazing deep sound. The kind of synths he was using were not just keyboard synths though, he played some kind of mental wooden frame with a couple of strings and springs attached. The noise that came out of that when pumped though the keyboard to then add pitch to sound was incredible. During the entire gig I started to really feel like I wanted a way to produce these kinds of sounds but without taking all the time to actually learn a keyboard instrument. I figured, I'm already a decent fiddle player, why can't I construct a fiddle shaped synth?

In prototyping this idea it seems like there are a few distinct parts to the instrument; how it detects my finger placement on the instrument, how it processes that information and how it produces the sound.

### Detecting Finger Placement

I want to get the synth as close to playing a fiddle as possible. That means I have to have very good accuracy in where the synth thinks my fingers are placed, a discrete system of buttons might do but I also need microtones for musical features like vibrato and slides. So I need some way of continuously and accurately detecting where my fingers are.

My very first idea was to simply use the violin string itself as a variable resistor, passing a current from the endpoint of the string to an electrical contact on my finger and using the resistance of the string itself as a measure of the length. However, using a multimeter I found the resistance per length of the string to be under a hundred milliohms per centimeter. Doing a little bit of research on how the Arduino could possibly detect these small resistances it seems like it's a difficult task, especially if we're wanting to detect changes in length as small as perhaps a millimeter. This idea has certain advantages in that it wouldn't require building a whole new instrument! It could just be attached to a regular fiddle. Because of that I'm not giving up on this idea but it needs a little more testing of different types of string and ways to measure small resistances. 

After failing to really get anywhere with the string idea, I came across Spectra Symbol's [softpots](http://www.spectrasymbol.com/product/softpot/). These are simply long flat potentiometers which have a resistance that varies linearly with where you're touching it. Perfect, right? Well I've bought some and we'll see how it goes.

I had also heard of people using [electrical paint](https://www.bareconductive.com/shop/electric-paint-50ml/?gclid=Cj0KEQiAw_DEBRChnYiQ_562gsEBEiQA4LcssuHbBHbOonK1rWwtI1zLbZnkc8qW16UWxkNb6rlz9UoaAvzy8P8HAQ) for interesting projects so naturally I thought about perhaps using it for this. It's literally what it sounds like, conductive paint that you apply to a surface and the resistance varies over whatever you've painted it on. This would allow me to create my own (possibly linear) potentiometers in basically any shape I want. I'll see how the softpots work but this is probably an option I'll explore as well.

### Producing the Sound

Because I have very little electronics experience I figured using a nice simple arduino board would be the best way to at least prototype the whole thing. Having previously went through a short course on the arduino I knew roughly what I was doing but it was about 7 years ago so I decided to buy one of the [starter packs](https://www.amazon.co.uk/gp/product/B01D8KOZF4/ref=oh_aui_detailpage_o01_s00?ie=UTF8&psc=1). Ultimately I'd like to be able to take the notes that the arduino produces, convert it into MIDI, probably following [this](https://www.arduino.cc/en/Tutorial/Midi) tutorial, and play that through a regular keyboard synthesizer. 
