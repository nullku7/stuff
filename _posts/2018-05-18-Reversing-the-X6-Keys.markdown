---
layout: post
title:  "Reversing the X6 Key Cutting Machine"
date:   2018-05-18 08:00:00 +11005
categories: reversing
---

### Introduction

This is a "Work in Progress"

### Notes
I am NOT a locksmith
I infact know NONE of the correct terminologies.
Therefor...
I will call the number of cuts..... the number of cuts....
I will call X axis as the long direction of the key, X1, X2 are the cut positions.
I will call Y axis as the cut depth/position, Y1, Y2, so on


### X6 / 15-Golf
This seems to match, very closely to the metric profile for the Golf in InstaCode

Format

|Possible Meaning|????|?|?|650|300|?|?|???|Number of Cuts|X1|X2|X3|X4|X5|X6|X7|X8|
|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|
|From File|!SB5|1|1|650|300|0|0|100|8|2430|2140|1850|1560|1270|980|690|400|
|Actual from Key||||||||8|2400|2100|
