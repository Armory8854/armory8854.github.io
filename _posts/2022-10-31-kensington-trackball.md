---
layout: post
title: "Kensington Expert Trackball"
date: 2022-10-31
categories: Linux Gentoo Kensington Trackball xinput
---
# Introduction
I use my Kensington Expert Pro Trackball, [Seen Here](https://www.kensington.com/p/products/electronic-control-solutions/trackball-products/expert-mouse-wired-trackball/), on my Gentoo installation on my iMac. This has 4 buttons on it which are programmable for various functions, like copying / pasting / back and forward button navigation in browsers. The main problem that I'm having now is that only 1 of the extra buttons besides left, and right, click works. And that only goes back in a browser. If I want to change the functions of these buttons on linux, I am faced with 2 problems. These 2 problems are:

1. Kensington doesn't provide their [Kensington Works Software](https://www.kensington.com/software/kensingtonworks/) for Linux devices.
2. Even if they did, I'm trying as hard as possible to use open source only options on my Gentoo installation.

# How I Troubleshooted The Issue
First, I know I'm using `xorg`, on top of the `libinput` drivers. Due to previous experience of setting up mouse speed on linux, I'm aware that the more than likely answer I need to figure out how my system reads the unique mouse buttons is `xinput`. Some googling led me to the specific answer I need from Stackexchange [^1], which is how to read input from a mouse. The actual program I needed was `xev`.

[^1]: https://unix.stackexchange.com/questions/106736/detect-if-mouse-button-is-pressed-then-invoke-a-script-or-command
