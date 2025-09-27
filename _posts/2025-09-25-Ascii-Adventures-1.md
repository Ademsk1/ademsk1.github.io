---
title: ASCII Adventures Pt. 1
date: 2025-09-25 16:30:00 +0000
categories: [coding, ascii]
tags: [coding, software, ascii] # TAG names should always be lowercase
description: Soujourning into ASCII animations and art
mermaid: true
math: true
---

ASCII animations need to come back to website design. You can find the work I'm referencing [here](https://ademsk1.github.io/ascii-motion/)


Welcome to my first blog post! And hopefully not my last. 

Recently I've been playing a lot of Watch Dogs on the Steam Deck. As a PC gamer, it takes a lot of getting used to, and it's not always the most comfortable, particularly when spamming the `RB/LB` buttons. 

One of the effects that I always thought looked cool was the DedSec ASCII animations that would appear whenever you hacked into a server - and started wondering how easy ASCII-based animations are to make. 

I had a simple goal - to make ASCII characters point towards the users mouse - wherever it was. But I also knew that I wanted this project to evolve over time, so decided to make it a generic `Ascii` class, that would have setup and run states. I also simplified the number of ascii characters to `/, | \\, -` that would point in the direction of the mouse. 


```js
function getAngle({ mouseX, mouseY, charX, charY }) {
  let x = mouseX - charX
  const y = mouseY - charY
  if (x === 0) { x += 0.1 } // remove chance of NaN result 

  return Math.atan(y / x) 
}
```

This function would return a value between $-\frac{\pi}{2} - \frac{\pi}{2}$ (or $-90^\circ$ to $90^\circ$). 

It naturally followed to choose the character based on this angle. 

```js
function chooseChar(angle, t) {
  if (angle < 0) angle += Math.PI
  const chars = ["-", "\\", "|", "/", "-"] //because 0,0 is at the top left corner, this needs to be inverted to have the \ before the / . Coordinates!! 
  const index = Math.round((angle / (Math.PI / 4)) + t)//round to nearest 45 degree val. 

  return chars[index % chars.length]
}
```

This seems far less verbose than it should be, but this actually suited me quite well. We have an angle from -90 - 90 degrees - so 180 degree range - we shift up by $\pi$ if it's negative to get a positive range. I wanted to divvy this range into 4 segments (hence the `Math.PI / 4`) - then I divided the angle to get the char index I wanted. 

It's certainly not a generic solution to getting chars in the right place, but it will definitely work for this scenario! 

Finally I wanted a quick way to get a character coordinate. For this, I simply found the window width and height to calculate the total area of the screen, then (using monospace characters) got the width and height - and using one `<span>` element, I filled it up with characters. With a quick `mouseOver` event listener, I got my first result! 

```js
  // as part of the Ascii class I created
  async run(mouseX = window.innerWidth / 2, mouseY = window.innerHeight / 2) {

    this.element.textContent = ""
    console.log({ mouseX, mouseY })
    for (let i = 0; i < this.charCount; i++) {
      const { x, y } = this.getCharCoordinates(i)
      const angle = getAngle({ mouseX, mouseY, charX: x, charY: y })
      const char = chooseChar(angle, t)
      this.element.textContent += char
    }
  }
  ... //outside of the class
    document.addEventListener("mousemove", (event) => {
    mouseX = event.clientX
    mouseY = event.clientY
  })
  window.addEventListener("resize", () => ascii.setDimensions(window.innerWidth, window.innerHeight))

  function update(t) {
    ascii.run(mouseX, mouseY)
    requestAnimationFrame(update)
  }
  update()

```

The results were semi-good for a first run! Setting the mouse to the anywhere in the left hand side of the image, to the center returned exactly what I wanted.
![Image](/assets/img/ascii-adventures-1/center.png)

However, something strange happened when I move the mouse to the left. 

![Image](/assets/img/ascii-adventures-1/half-seen.png)

What on earth is going on here!? 

I still haven't properly found the root source. But in a single textContent, on a new line, dashes and em-dashes have a particular property of cutting away from the rest of the text.it wants to put the next non-dash char on the new line. I think this is a property inherent within how text-content is broken up, as automatically, words are broken up on the dash. Take a look on the two images below - this is me stepping through the code as I see each dash being added. The first one shows that the `\\`'s are in the right place. But as soon as they exceed the screen width, they break.

![Image](/assets/img/ascii-adventures-1/one-line.png)

![Image](/assets/img/ascii-adventures-1/two-lines.png)

Because the mouse on the left-hand side of the screen dashes are filled up, until around half way when it moves to back-slashes. And because the coordinate position is reliant on the index of the char, this breaks everything. 

This prompted me to move away from using `-`'s in general. I suspect that other characters are going to give me a headache as well in the future...

A simple solution would be to use a `<span>` element for every character there is. This would also have the benefit of being able to get a coloured ASCII video - so I'll probably migrate to that when all is said and done. 

Through messing around with this, I realised it's lacking panache! What about a spotlight effect? Easy! Using the time parameter injected into the `requestFrameAnimation` call, I could pass it diretly to the `chooseChar` function. 


```js
function chooseChar(angle, t) {
  if (angle < 0) angle += Math.PI

  const chars = ["z", "\\", "|", "/", "@"] //because 0,0 is at the top left corner, this needs to be inverted to have the \ before the / . Coordinates!! 
  const index = Math.round((angle / (Math.PI / 4)) + t)//round to nearest 45 degree val. 

  return chars[index % chars.length]
}
```
I omitted the dashes. And the result I get is really enjoyable to watch! Hover your mouse below! 

<iframe src="https://ademsk1.github.io/ascii-motion/" width="680" height="300"></iframe>


I've gotta say, I love how unintrusive it is - it feels like it naturally goes into the screen. Not sure about how it will look in dark mode though. And with the `getChars` function you can modify the equation a touch more to get a spiral effect! 

One last change I've made is the FPS. One of these alone was working up to 1.5 cores consumption. By setting it to a nice 20fps, that brought CPU down to 0.3. Who said ASCII was cheap. 

<iframe src="https://ademsk1.github.io/ascii-motion/spiral/?frequency=5" width="680" height="300"></iframe>


While I'm talking about spirals, if you increase the `frequency` search parameter in the `https://ademsk1.github.io/ascii-motion/spiral/?frequency=` request, you can get some effects that are awesome! Place your mouse in the top left of this next one!

<iframe src="https://ademsk1.github.io/ascii-motion/spiral/?frequency=600" width="680" height="300"></iframe>

See the weird tick like effect? I'm obsessed.


With iframes, and a bit of white-space masking, you can make text out of it!


<script>
document.addEventListener("DOMContentLoaded", () => {
  const iframes = document.querySelectorAll("iframe");

  iframes.forEach((iframe) => {
    iframe.onload = () => {
      const iframeDoc = iframe.contentDocument || iframe.contentWindow.document;
      const style = iframeDoc.createElement("style");
      style.textContent = `
        body {
          background-color: var(--main-bg) !important;
          color: #e0e0e0 !important;
        }
        a { color: #90caf9 !important; }
      `;
      iframeDoc.head.appendChild(style);
    };
  });
});
</script>




<div style="position: relative; width: 420px; height: 120px; overflow: hidden;">
  <!-- IFRAME BACKGROUND -->
  <iframe src="https://ademsk1.github.io/ascii-motion/spiral/?frequency=600" width="420" height="120" 
          style="position:absolute; top:0; left:0; border:none;"></iframe>

  <!-- WHITE MASKS (cover parts to carve HELLO) -->
  
  <!-- Space between H and E -->
  <div style="position:absolute; left:60px; top:0; width:20px; height:120px; background-color:var(--main-bg);"></div>

  <!-- Space between E and L -->
  <div style="position:absolute; left:140px; top:0; width:20px; height:120px; background-color:var(--main-bg);;"></div>

  <!-- Space between L and L -->
  <div style="position:absolute; left:220px; top:0; width:20px; height:120px; background-color:var(--main-bg);;"></div>

  <!-- Space between L and O -->
  <div style="position:absolute; left:300px; top:0; width:20px; height:120px; background-color:var(--main-bg);;"></div>

  <!-- H holes -->
  <div style="position:absolute; left:20px; top:0; width:20px; height:40px; background-color:var(--main-bg);"></div>
  <div style="position:absolute; left:20px; top:60px; width:20px; height:60px; background-color:var(--main-bg);"></div>

  <!-- E holes -->
  <div style="position:absolute; left:100px; top:20px; width:40px; height:20px; background-color:var(--main-bg);"></div>
  <div style="position:absolute; left:100px; top:60px; width:40px; height:40px; background-color:var(--main-bg);"></div>

  <!-- L holes -->
  <div style="position:absolute; left:180px; top:0; width:40px; height:100px; background-color:var(--main-bg);"></div>

  <!-- Second L holes -->
  <div style="position:absolute; left:260px; top:0; width:40px; height:100px; background-color:var(--main-bg);"></div>

  <!-- O hole -->
  <div style="position:absolute; left:350px; top:20px; width:40px; height:80px; background-color:var(--main-bg);"></div>
</div>


I'm planning to carry on with this work. I think colour will be important, but also, converting video files / video streaming would be cool to do. I'm also going to play with some more effects, so watch this space!

So what do you say? Shall we bring back ASCII? 





