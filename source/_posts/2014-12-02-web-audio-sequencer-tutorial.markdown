---
layout: post
title: "Web Audio Timing Tutorial"
date: 2014-12-02 22:56:24 -0500
comments: true
categories: [Web Audio, Tutorial]
---

My last post was a delirious declaration that I had finally made something with Web Audio. Now I want to explain how I did it. First, what is Web Audio, and why do we need it?

The Web Audio API is a versatile system for controlling audio in the web. It does this inside an **audio context**, which you declare as follows:

```javascript
var context = new AudioContext();
```

You then create sources, either by loading a song or sample via an AJAX request (though you can't use jQuery since it doesn't support responses of type 'arrayBuffer':

```javascript
var songBuffer = null;

//for multiple browser support
window.AudioContext = window.AudioContext || window.webkitAudioContext;
var context = new AudioContext();
function loadSong(url) { // the url can be full path or relative
  var request = new XMLHttpRequest();
  request.open('GET', url, true);
  request.responseType = 'arraybuffer';

  // Decode asynchronously
  request.onload = function() {
    context.decodeAudioData(request.response, function(buffer) {
      songBuffer = buffer;
    }, onError);
  }
  request.send();
}
```

Once you have sources, you route sounds through a series of **audio nodes** to add effects, such as reverb, filtering, and compression. It is also **modular**, meaning it is simple to change the routes to add or remove effects. It is analogous to physical synthesizers and modular synthesizers. The last node in the **audio routing graph** is the destination node, which is responsible for actually _playing_ the sounds. Here's a routing example:

```javascript
function playSound(buffer) {
  // creates a sound source from buffer just loaded
  var source = context.createBufferSource(); 
  source.buffer = buffer;                    // tell the source which sound to play
  source.connect(context.destination);       // connect source to context's destination (the speakers)
  source.start(0);                           // play the source now
}
```

Cool, so now we can play audio. As soon as I figured out how to do this, the first thing I wanted to learn how to do is how to sequence audio events, because that's how you making music: you organize sounds in time. This is a deep, dark rabbit hole. On the [introduction to Web Audio tutorial](http://www.html5rocks.com/en/tutorials/webaudio/intro/), they have an example which synchronizing to the Javascript clock. But this is bad. _Very_ bad. The Javascript clock is not precise, and therefore if you synchronize audio events to it, there will be noticeable latency and stuttering. You must synchronize to the Web Audio clock, which is a signal from an actual hardware crystal (and therefore very precise). 

Another problem is scheduling. Web Audio has no built in scheduler. Therefore, you have to build your own (or use a library that has one, for example, [WAAClock](https://www.npmjs.org/package/waaclock) or [SoundJS](https://github.com/CreateJS/SoundJS)). You also have to think how far ahead you want to schedule your audio events. For example, if you have the code (which is synchronized to the Javascript clock)

```javascript
for (var bar = 0; bar < 2; bar++) {
  var time = startTime + bar * 8 * eighthNoteTime;

  // Play the hi-hat every eighth note.
  for (var i = 0; i < 8; ++i) {
    playSound(hihat, time + i * eighthNoteTime);
  }
}
```

and you want to change tempo, what do you do? You can't unschedule the notes once you've told them to play. You would have to add a gain node to control the volume, turn off the volume on the now off-tempo loop, and then create a _new_ loop with the new correct tempo. And ad infinitum every time you want to change the tempo again.

That's awful, and there is a better way. After reading [A Tale of Two Clocks](http://www.html5rocks.com/en/tutorials/audio/scheduling/) a few times until it finally made sense. The way to make a good scheduler is to schedule one note ahead of time, and to synchronize it to the Web Audio clock. Unfortunately, it's easier said than done. Your scheduler function ends up looking something like this:

```javascript
while (nextNoteTime < audioContext.currentTime + scheduleAheadTime ) {
  scheduleNote( current16thNote, nextNoteTime );
  nextNote();
}
```

But scheduleAheadTime and nextNoteTime have to be tweaked according to your use case. In my case, I ended up using the same parameters as used in the [Google Shiny Drum Machine](http://chromium.googlecode.com/svn/trunk/samples/audio/shiny-drum-machine.html) and it seems to be okay. Let's look at my code for the core scheduling functionality:

```javascript
function handlePlay(event) {
    noteTime = 0.0;
    startTime = context.currentTime + 0.005;
    schedule();
}


function schedule() {
  var currentTime = context.currentTime;

  // The sequence starts at startTime, so normalize currentTime so that it's 0 at the start of the sequence.
  currentTime -= startTime;

  while (noteTime < currentTime + 0.200) {
    var contextPlayTime = noteTime + startTime;

    //Insert playing notes here

    //Insert draw stuff here

    advanceNote();

  }
  timeoutId = setTimeout("schedule()", 0);
}

function advanceNote() {
    // Setting tempo to 60 BPM just for now
    var tempo = 60.0;
    var secondsPerBeat = 60.0 / tempo;

    rhythmIndex++;
    if (rhythmIndex == LOOP_LENGTH) {
        rhythmIndex = 0;
    }
   
    //0.25 because each square is a 16th note
    noteTime += 0.25 * secondsPerBeat;
}
```

Okay, there's a lot going on here, so let's break this down. Note that I didn't include anything about playing notes or updating the visuals as I just wanted to focus on the timing/scheduling.  

First, in handlePlay(), we reset noteTime, which represents the length of a note. And then we set startTime, which includes an offset to take into account stuff happening in the main Javascript execution time (e.g. garbage collection). 

Then, in schedule, we first check if we need to sequence audio events, depending on noteTime and currentTime. Note that the 200 ms offset here represents the variable scheduleAheadTime in the previous example. The while loop inside of the function schedule() is run again and again until we've scheduled all notes that fall within our lookahead interval. Therefore, as we schedule more notes, and increment noteTime based on the tempo of the song (in advanceNote()):

```javascript
var secondsPerBeat = 60.0 / tempo;
//...
noteTime += 0.25 * secondsPerBeat;
```

Since we're assuming a 4/4 time signature, to obtain noteTime we multipy secondsPerBeat by 0.25 since each note is a 16th note. Once we've scheduled all of the notes that fall within our lookahead interval, the condition of the while loop in schedule() 

```javascript
while (noteTime < currentTime + 0.200) {
```

will fail. Lastly, what's with the last line of schedule()?

```javascript
timeoutId = setTimeout("schedule()", 0);
```

setTimeout is a Javascript library function that executes a function after a certain period of time. Therefore, this tells Javascript to call the schedule() function after 0 ms. So basically, call schedule() again right after it's scheduled all notes within out lookahead interval and begin the process over again. 

Lastly, there's the rhythmIndex variable in advanceNote():

```javascript
rhythmIndex++;
if (rhythmIndex == LOOP_LENGTH) {
    rhythmIndex = 0;
}
```

This variable simply represents where you are in the context of the loop, and resets when the sequence reaches then end so that it loops. 

Cool, so now we can sequence audio properly! Well, kind of. As is pointed out in [this StackOverflow](http://stackoverflow.com/questions/20598147/perfect-synchronization-with-web-audio-api), synchronizing to the Web Audio clock does not mean that your audio is synchronized to the animation frame refresh rate. However, this is only important if you want to synchronize animations to Web Audio. For that, you can read the last section in [A Tale of Two Clocks](http://www.html5rocks.com/en/tutorials/audio/scheduling/).

Look out for a part two of this blog post, which will explain more in depth about **audio routing graphs**, and different types of nodes, i.e. how to add different types of effects. 









