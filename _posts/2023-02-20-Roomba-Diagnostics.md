---
layout: post
title: Parsing Windows Events with KQL
tags: 
- roomba 
- robot
---

## Connect a terminal program
1. Connect Micro-USB to Roomba and USB-A to computer
2. Open Putty with the correct COM port on baud 115200

## Put the Roomba 980 in Diagnostics Mode
1. Remove the Roomba from the charging base, make sure no LEDs are on, like the Clean button.
2. Press and hold down the Home button and Clean button while pressing the Spot Clean button 6 times.
3. Release all buttons and wait for the Roomba to play the BiTs tune.
4. You are now in test 0. 

Advance through each test by pushing the Spot Clean button. You must move the robot around per the test to get a pass/fail on most of the steps. For example, testing the cliff sensors requires you to pick the robot up and put it down to get a pass/fail response.