---
layout: post
title: Flare-On 11 Write-Up Part 2
date: 2024-12-01 17:07 +0100
categories: [Hacking, Software]
tags: [hacking, software, capture the flag, write-up, reverse engineering]
media_subpath: /assets/img/flareon11_2/
---

This is the second post in my **Flare-On 11** series. If you haven’t already, I recommend checking out the [previous writeup](/posts/flare-on-11-write-up-part-1/) to get up to speed. 

As with the first post, you can find supplementary resources, including all the scripts I used during the CTF, in [this repository](https://github.com/vik0t0r/flare-on-11).

## Challenge 4 - Meme Maker 3000

>You've made it very far, I'm proud of you even if noone else is. You've earned yourself a break with some nice HTML and JavaScript before we get into challenges that may require you to be very good at computers.

In this challenge we are given a single html document. When we open it on a web browser we are welcome with a meme generator:

![img-description](meme1.png){: width="300" }

Apparently we can choose between multiple templates from a desplegable and them will become populated with random strings from a pool of strings once we click on `Remake`.

#### Step 1: Initial approach

Lets dive into the code, clicking on `view source` reveals us that this webpage is composed of some css, some html and some obfuscated js:

```html
<body>
    <h1>FLARE Meme Maker 3000</h1>

    <div id="controls">
        <select id="meme-template">
            <option value="doge1.png">Doge</option>
            <option value="draw.jpg">Draw 25</option>
            <option value="drake.jpg">Drake</option>
            <option value="two_buttons.jpg">Two Buttons</option>
            <option value="boy_friend0.jpg">Distracted Boyfriend</option>
            <option value="success.jpg">Success</option>
            <option value="disaster.jpg">Disaster</option>
            <option value="aliens.jpg">Aliens</option>
        </select>
        <button id="remake">Remake</button>
    </div>

    <div id="meme-container">
        <img id="meme-image" src="" alt="">
        <div id="caption1" class="caption" contenteditable></div>
        <div id="caption2" class="caption" contenteditable></div>
        <div id="caption3" class="caption" contenteditable></div>
    </div>

    <script>
        const a0p=a0b;(function(a,b){const o=a0b,c=a();while(!![]){try{const d=parseInt(o(0xd7ed))/0x1*(parseInt(o(0x381d))/0x2)+-p
    </script>
</body>
```

This seems a simple webpage, but the javascript is obfuscated!

#### Step2: javascript deobfuscation

Lets see what this javascript does! My first step was to use an online deobfuscation like [this one](https://deobfuscate.io/). Then I replaced the obfuscated code wiht the deobfuscated version, for this, I moved the javascript file to an external one and I imported it from the html:

```html
<script src="memaker_deobf1.js"></script>
```

```javascript
const a0c = ["When you find a buffer overflow in legacy code", "Reverse Engineer", "When you decompile the obfuscated code and it makes perfect sense", "Me after a week of reverse engineering", "When your decompiler crashes", "It's not a bug, it'a a feature", "Security 'Expert'", "AI", "That's great, but can you hack it?", "When your code compiles for the first time", "If it ain't broke, break it", "Reading someone else's code", "EDR", "This is fine", "FLARE On", "It's always DNS", "strings.exe", "Don't click on that.", "When you find the perfect 0-day exploit", "Security through obscurity", "Instant Coffee", "H@x0r", "Malware", "$1,000,000", "IDA Pro", "Security Expert"];
const a0d = {
  doge1: [["75%", "25%"], ["75%", "82%"]],
  boy_friend0: [["75%", "25%"], ["40%", "60%"], ["70%", "70%"]],
  draw: [["30%", "30%"]],
  drake: [["10%", "75%"], ["55%", "75%"]],
  two_buttons: [["10%", "15%"], ["2%", "60%"]],
  success: [["75%", "50%"]],
  disaster: [["5%", "50%"]],
  aliens: [["5%", "50%"]]
};
const a0e = { ... }; // images encoded as base64

function a0f() {
  document.getElementById("caption1").hidden = true;
  document.getElementById("caption2").hidden = true;
  document.getElementById("caption3").hidden = true;
  const a = document.getElementById("meme-template");
  var b = a.value.split(".")[0];
  a0d[b].forEach(function (c, d) {
    var e = document.getElementById("caption" + (d + 1));
    e.hidden = false;
    e.style.top = a0d[b][d][0];
    e.style.left = a0d[b][d][1];
    e.textContent = a0c[Math.floor(Math.random() * (a0c.length - 1))];
  });
}
a0f();
const a0g = document.getElementById("meme-image");
const a0h = document.getElementById("meme-container");
const a0i = document.getElementById("remake");
const a0j = document.getElementById("meme-template");
a0g.src = a0e[a0j.value];
a0j.addEventListener("change", () => {
  a0g.src = a0e[a0j.value];
  a0g.alt = a0j.value;
  a0f();
});
a0i.addEventListener("click", () => {
  a0f();
});
function a0k() {
  const a = a0g.alt.split("/").pop();
  if (a !== Object.keys(a0e)[5]) {
    return;
  }
  const b = a0l.textContent;
  const c = a0m.textContent;
  const d = a0n.textContent;
  if (a0c.indexOf(b) == 14 && a0c.indexOf(c) == a0c.length - 1 && a0c.indexOf(d) == 22) {
    var e = new Date().getTime();
    while (new Date().getTime() < e + 3000) {}
    var f = d[3] + "h" + a[10] + b[2] + a[3] + c[5] + c[c.length - 1] + "5" + a[3] + "4" + a[3] + c[2] + c[4] + c[3] + "3" + d[2] + a[3] + "j4" + a0c[1][2] + d[4] + "5" + c[2] + d[5] + "1" + c[11] + "7" + a0c[21][1] + b.replace(" ", "-") + a[11] + a0c[4].substring(12, 15);
    f = f.toLowerCase();
    alert(atob("Q29uZ3JhdHVsYXRpb25zISBIZXJlIHlvdSBnbzog") + f);
  }
}
const a0l = document.getElementById("caption1");
const a0m = document.getElementById("caption2");
const a0n = document.getElementById("caption3");
a0l.addEventListener("keyup", () => {
  a0k();
});
a0m.addEventListener("keyup", () => {
  a0k();
});
a0n.addEventListener("keyup", () => {
  a0k();
});
```
Here we can see a couple of interesting things:

1. `a0c`: Is an array with possible strings for the memes.
2. The function `a0f` seems to be the one in charge of populating the text. This line is quite revealing: `e.textContent = a0c[Math.floor(Math.random() * (a0c.length - 1))];`, a text content being set by a random element of the array.
3. The function `a0k` seems to be the one in charge of printing the flag, as it ends with: `alert(atob("Q29uZ3JhdHVsYXRpb25zISBIZXJlIHlvdSBnbzog") + f);`, `atob` decodes base64 to: "Congratulations! Here you go: ", so `f` must hold our flag.
4. `if (a0c.indexOf(b) == 14 && a0c.indexOf(c) == a0c.length - 1 && a0c.indexOf(d) == 22) {` seems to check that we've got the correct conditions, so if we find `a`, `b`, `c`, `d` and execute the code, we would get the correct flag.
5. For this we will need to manually leverage the power of the web console, which allows us to execute JavaScript on the context of the webpage.

#### Step 3: Getting the flag

The conditions are easily reversed, so if we execute the following code we get the flag:
```javascript
a = Object.keys(a0e)[5];
b = a0c[14];
c = a0c[a0c.length-1];
d = a0c[22];
// and then we execute the generation of the flag
 f = d[3] + "h" + a[10] + b[2] + a[3] + c[5] + c[c.length - 1] + "5" + a[3] + "4" + a[3] + c[2] + c[4] + c[3] + "3" + d[2] + a[3] + "j4" + a0c[1][2] + d[4] + "5" + c[2] + d[5] + "1" + c[11] + "7" + a0c[21][1] + b.replace(" ", "-") + a[11] + a0c[4].substring(12, 15);
 f.toLowerCase()
```
Which returns: "wh0a_it5_4_cru3l_j4va5cr1p7@flare-on.com"

If we want to see which is the meme that generates the flag, we can print each of the variables:

1. `a` = "boy_friend0.jpg"
2. `b` = "FLARE On"
3. `c` = "Security Expert"
4. `d` = "Malware"

So we just need to show the "Distracted Boyfriend" template and execute the following JavaScript, and voilà:

```javascript
document.getElementById("caption1").textContent = b
document.getElementById("caption2").textContent = c
document.getElementById("caption3").textContent = d
```

Which gives us the following image:

![img-description](meme2.png){: width="600" }

What's interesting about this one is that the "Security Expert" string is the last one of the array, so by clicking `Remake` a lot of times we would never generate the correct combination by chance.

That wasn't that hard, was it?


## Continuation

Check the following post [here](/posts/flare-on-11-write-up-part-3/)