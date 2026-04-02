---
title: "Water Bottle - TryHackMe"
date: 2026-04-02 02:00:00 +0000
categories: [TryHackMe, Writeups]
tags: [osint, google-maps, google-earth]
image: /assets/img/writeups/covers/water-bottle.png
---
# Water Bottle — TryHackMe
![1.png](/assets/img/writeups/water-bottle/1.png)
**Room**: [Water Bottle](https://tryhackme.com/room/waterbottle)<br>**Difficulty**: Easy<br>**Category**: OSINT

## Overview
In this room, we need to find a water refilling station that someone visited in the past. We only have two clues to work with — the area where the station is located, and the first few digits of its phone number. Our job is to figure out the full station name and complete phone number.<br>🛠️**Tools Used**: Google Maps, Google Earth, browser<br>⚠️**Warning**: Full solution ahead. Try the room yourself first!


## Step 1 — Searching for Water Refill Stations Nearby
Start by opening Google Maps and searching for water refill stations near Boni Avenue. You will get several results. Don't worry — we will narrow them down in the next step.
![1.png](/assets/img/writeups/water-bottle/2.png)

## Step 2 — Checking Historical Imagery on Google Earth
Since we need to find a station that existed around 2014, we can't just rely on current maps. Open Google Earth, go to the Boni Avenue area, and use the historical imagery feature to go back in time.<br>Looking through the older images, a station called Aqua Best shows up at the target location during that time period. This is our match.
![1.png](/assets/img/writeups/water-bottle/3.png)
![1.png](/assets/img/writeups/water-bottle/4.png)
![1.png](/assets/img/writeups/water-bottle/5.png)
## Step 3 — Finding the Full Phone Number
Now that we have the name, search for "Aqua Best water refill station Boni Avenue" in your browser. This should lead you to their website.
Go to the contact section of the site. The challenge already gave us the starting digits of the number (63922). Complete the CAPTCHA on the page and the full phone number will be revealed.
![1.png](/assets/img/writeups/water-bottle/6.png)
![1.png](/assets/img/writeups/water-bottle/7.png)
## Step 4 — Building the Flag
We now have everything we need:
- Station name: **Aqua Best**
- Full contact number from the website

Combine them in the flag format required by the challenge and submit.
