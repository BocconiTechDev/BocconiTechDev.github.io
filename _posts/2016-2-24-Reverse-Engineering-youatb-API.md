---
layout: post
title: Reverse Engineering the API of youatb Android client
published: true
---


## The problem
The Bocconi official Android app feels very unprofessional, constantly restarts, the timetable is never cached, has no design at all, etc. To write a replacement-enchacement there are three ways:
- Reverse Engineering the API
- Write a client emulating desktop browser behaviour around the youatb website
- Get the information in a different way, for example, the timetables are available on the main website by code
I'll try the API way. 
I have decompiled the application with apktool, but nothing is reusable there anyway. Luckily, the communication channel between the app and API server seems to be unencrypted at all. Let's just sniff the traffic!

## Instruments
We need:
- WireShark
- An Android in a virtual machine, quickest way to setup seems to be to use Genymotion
- A package of gapps(I use [OpenGApps](http://opengapps.org/)), Bocconi app fails to work on AOSP without proprietary libraries
- APK of the app, one can download from an actual device, or just login to Google. There are gapps and Google Play in this virtual machine, after all
- adb, to install apk to the virtual machine

##Workflow in Pictures
![Screen1-2016-2-24.png]({{site.baseurl}}/images/Screen1-2016-2-24.png)
![Screen2-2016-2-24.png]({{site.baseurl}}/images/Screen2-2016-2-24.png)
![Screen3-2016-2-24.png]({{site.baseurl}}/images/Screen3-2016-2-24.png)
![Screen4-2016-2-24.png]({{site.baseurl}}/images/Screen4-2016-2-24.png)


## Collected data
