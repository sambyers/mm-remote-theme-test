---
layout: post
title:  "Export Subnets in a Meraki Network"
date:   2017-09-06
categories: meraki
---
Someone on reddit recently had a question about exporting all of the configured subnets from a Meraki network. There doesn't seem to be a way to accomplish this through the dashboard. I'd never thought of it before, and I looked around the dashboard and couldn't find a way.

So, I used the [dashboard](https://documentation.meraki.com/zGeneral_Administration/Other_Topics/The_Cisco_Meraki_Dashboard_API) API to get it done!

The project is [here](https://github.com/sambyers/meraki).

Usage of the script is really simple: `usage: get-all-subnets.py [-h] [-k KEY] [-o ORG] [-n NET]`

I hope this helps someone faced with the same issue. If you'd like to improve the script, please submit a pull request! If something doesn't work, submit an issue!

Thanks for reading!