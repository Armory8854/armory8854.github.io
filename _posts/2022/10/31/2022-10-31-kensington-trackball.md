---
layout: post
title: "Custom Mouse Button Bindings With xinput"
date: 2022-10-31
tags: linux-desktop gentoo
---
# Introduction
I use my Kensington Expert Pro Trackball, [Seen Here](https://www.kensington.com/p/products/electronic-control-solutions/trackball-products/expert-mouse-wired-trackball/), on my Gentoo installation on my iMac. This has 4 buttons on it which are programmable for various functions, like copying / pasting / back and forward button navigation in browsers. The main problem that I'm having now is that only 1 of the extra buttons besides left, and right, click works. And that only goes back in a browser. If I want to change the functions of these buttons on linux, I am faced with 2 problems. These 2 problems are:

1. Kensington doesn't provide their [Kensington Works Software](https://www.kensington.com/software/kensingtonworks/) for Linux devices.
2. Even if they did, I'm trying as hard as possible to use open source only options on my Gentoo installation.

So, instead of trying to get the Kensington Works software working on Gentoo, I'm going to go the other route and try to see if I can remedy this situation with the tools easily installed on any linux distribution. Some, if not all, of these tools may be installed already, depending on your setup.

# How I Solved The Issue
First, I know I'm using `xorg`, on top of the `libinput` drivers. Due to previous experience of setting up mouse speed on linux, I'm aware that the more than likely answer I need to figure out how my system reads the unique mouse buttons is `xinput`. Some googling led me to the specific answer I need from Stackexchange [^1], which is how to read input from a mouse. The actual program I needed was `xev`. Using xev, I was able to watch the input of each button and determine how it was read by system.

![Output Of The xev program](/assets/images/xev_output.png)

*Example output of the xev program*

Using this output, I was able to determine that 
1. the top left button was button 2
2. the top right button was button 8

OK, cool. I have the buttons I need... but how do I bind them to the actions I want? For context, I'm using the i3 window manager, so I have no traditional settings menu to change any keypresses to anything. So, back to google I go. Further research[^2] led me to `xdotool`, and the option to bind key via a configuration file called `.xbindkeysrc`. So if this link is right, I should simply be able to bind the buttons like so in my .xbindkeysrc

```
"xdotool key Alt+Left"
b:2

"xdotool key Alt+Right"
b:8
```

This did NOT work... But fret not, after a small amount of research, I came across the answer I was looking for: The arch wiki's entry for `xbindkeys`[^3]! after installing this package and running it, I was succesfully able to remap my mouse buttons!

# Footnotes
[^1]: https://unix.stackexchange.com/questions/106736/detect-if-mouse-button-is-pressed-then-invoke-a-script-or-command

[^2]: https://faq.i3wm.org/question/528/binding-mouse-buttons.1.html

[^3]: https://wiki.archlinux.org/title/Xbindkeys
