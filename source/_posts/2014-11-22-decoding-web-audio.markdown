---
layout: post
title: "Build a Drum Machine in your Browser"
date: 2014-11-30 18:51:53 -0500
comments: true
categories: [Web Audio]
---

In my college career, I majored in electrical engineering, and because of this, I learned about signal processing and audio. Now that I'm learning web development, I've been eager to combine my skills and delve into Web Audio, an API that allows you to do audio processing magic in the browser. This API allows you to create websites like Soundcloud or even [remix songs algorithmically](http://echonest.github.io/remix/). I decided to create a basic drum machine, and then modify it to create a digital sequencer.

I first followed this [Virtual Synth](http://www.sitepoint.com/html5-web-audio-api-tutorial-building-virtual-synth-pad/) tutorial, which was a great place to start, but there were a few things I didn't like about it:

* No ability to create a sequence. What good is a drum machine without a sequencer ([unless you're Araabmuzik](https://www.youtube.com/watch?v=wXRnbS6o64U))?
* Either I implemented the filter wrong, which is very possible, or it doesn't work correctly. It doesn't use a logarithmic scale, however, which is definitely a big mistake.

As I was trying to figure out how to sequence audio events, I came upon a article that went slightly above my head when I first read it. In ["A Tale of Two Clocks"](http://www.html5rocks.com/en/tutorials/audio/scheduling/), the author makes it clear that you must synchronize to the Web Audio API clock, and not the Javascript clock, since the latter is imprecise. Even slight audio stuttering is very noticeable. The author links to a sequencer created by Google engineers, called [Shiny Drum Machine](http://chromium.googlecode.com/svn/trunk/samples/audio/shiny-drum-machine.html), which I used for reference. The code itself is good but not great, and could definitely be organized better. 

I threw a bunch of stuff together just to get a working app, but I think that my next step is definitely to pause for a moment and refactor. 

I will write a tutorial in the future for this, but for now, since my brains hurts a lot, you can check out the current version of my [Web Audio Sequencer](http://web-audio-sequencer.herokuapp.com/). And, of course, check out [my Github](https://github.com/catarak/web-audio-sequencer) if you want to look at the code ([latest commit as writing this post](https://github.com/catarak/web-audio-sequencer/commit/3f4a2f08f7f84acfc85297a626ce9286a43034d6)). 

{% img center http://media.giphy.com/media/vFtWp05vBYnMQ/giphy.gif %}