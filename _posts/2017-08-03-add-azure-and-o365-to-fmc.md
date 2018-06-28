---
title: "Add Azure and Office 365 prefixes to your FirePOWER Management Center"
date: 2017-08-03 14:49:05 -0400
categories: technology firepower script
---
Recently, I had to add Azure and Office 365 IP prefixes to FMC. There are hundreds of prefixes in the XML files for both of these services! When I saw the size of the files, I reached for my handy scripting tool: Python!

The project is [here](https://github.com/sambyers/o365_fmc).

Usage of the script is really easy: `usage: o365_fmc.py [-h] server username password service`

Service can be azure or o365.

I hope this helps someone faced with the same issue. If you'd like to improve the script, please submit a pull request! If something doesn't work, submit an issue!

Thanks for reading!