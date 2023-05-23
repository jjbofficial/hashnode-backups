---
title: "Hardware Debuggers Save Time"
datePublished: Tue May 23 2023 11:47:11 GMT+0000 (Coordinated Universal Time)
cuid: cli07okf5000e09mw4fjreb8u
slug: hardware-debuggers-save-time
cover: https://cdn.hashnode.com/res/hashnode/image/stock/unsplash/tTugjw8f4Ms/upload/8f1734307ae3dafd43be02e8866f357d.jpeg
tags: iot, electronics, ghana, embedded-systems

---

My entry into embedded systems like most other people started with an Arduino Uno. The UNO didn't have a hardware debugger on board so If I made a mistake, I had to place well-structured `Serial.println` statements throughout my code so that I could follow the logic and find out where things went wrong. Anyone who has done this enough times knows how frustrating it can be and I knew at the back of my mind that there definitely had to be a better way.

When I started programming STM32 microcontrollers, I got introduced to using a hardware debugger and the experience was surreal. The ability to step through my code line-by-line and see the values of registers and variables change in real-time made debugging a better experience. Most microcontroller platforms have some kind of hardware debugger that you can use. ESP32s have [ESP-PROG](https://espressif-docs.readthedocs-hosted.com/projects/espressif-esp-iot-solution/en/latest/hw-reference/ESP-Prog_guide.html), and STM32s have [ST-LINK](https://www.youtube.com/watch?v=wt8uwz7eJDE). If you have never tried using a hardware debugger before, I encourage you to treat yourself.

If you don't know where to get these programmers, you can read my previous article on [how I source components in Ghana](https://blog.jjbofficial.com/getting-electronic-components-for-your-next-project-in-ghana)