---
title: "Room Wifi Service POC"
date: 2020-03-10
categories: poc demo wifi wireless meraki flask
---
_Self service wifi for short and medium term guest users_

Recently, I was chatting with some colleagues about interesting applications of API driven infrastructure. They had a come up with an idea and demo to allow short and medium term guests to provision their own wifi networks on an organization's infrastructure. This capability could be used in rented rooms, office space, conference rooms, or other temporary arrangements where self service wifi would be helpful.

My own experience as a guest user lead me to think this was a good idea as a service. I have a lot of devices and usually I don't bother to get them all on wifi when I am a guest user. I will either use my phone as a hotspot or turn on an ad hoc wifi network and share internet through my laptop. I thought it’d be useful if I could do the same thing but with the facility’s AP in my immediate location.

After thinking about this concept for a while and seeing a demo of what my colleagues built, I decided I wanted to build a demo too. The result is the Room Wifi Service. It's a small POC built with flask, SQLite, and the Meraki dashboard API.

The basic requirements to run this service are a place to host the flask app, Meraki MR AP, Meraki dashboard API key, and internet access.

The main features of room wifi service are adding users, provisioning wifi networks for those users based on their provided information, and de-provisioning users and their networks. There are some non-functional features like CSRF protection and object based forms implemented with a nifty package called FlaskWTF. I also implemented simple magic links with python's standard library secrets module. Magic links allow users to provide the network name and password (SSID and PSK) without having to login with credentials.

### Demo of the magic link sent to the user via email/SMS
![](https://media.giphy.com/media/j2G0ASq7TgqTKlqpB8/giphy.gif)

For the demo app I added a convenience web UI to manage users in the system. It allows an operator to add users manually, verify user details, and delete/de-provision users.

### Adding a user

![](https://media.giphy.com/media/Pkjsl7dDRHaexqMNR1/giphy.gif)

### Deleting a user

![](https://media.giphy.com/media/VcvcsTlHBqfVi7Zz5m/giphy.gif)

Users could be added through an API call from a CRM or CIS to create a semi-automated provisioning process. A user would then get an email with the magic link and could continue the registration process.

I drew some high level sequences to show how the process should flow. I’m new to diagraming software interactions and this was good practice.

### CRM or CIS seeding the Room Wifi Service with user information
![](/assets/images/roomwifiservice-crm-cis-sequence.png)

### User tapping the magic link in their registration email
![](/assets/images/roomwifiservice-user-sequence.png)

You can find more details and the code at the [Github repo](https://github.com/CiscoSE/roomwifiservice).

I felt weird about essentially remaking a demo someone else had already made. I realized that there were things to learn from making this toy POC and that was enough reason to go ahead and build this demo. But I definitely want to give credit where it is due. Credit for the original idea goes 100% to Geoff Giulino and Matt Siegel.

Thanks for reading!