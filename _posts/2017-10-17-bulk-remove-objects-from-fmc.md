---
title: "Bulk remove objects from the FMC"
date: 2017-10-17
categories: technology firepower script
---
I recently had a case where configuration was being imported into an FMC and duplicate objects were being erroneasouly created. This is an issue that doesn't really affect functionality, but it definintely pollutes your objects.

So, I used the [fmcapi](https://github.com/daxm/fmcapi) library to easily work with the FMC API and remove these duplicate objects.

The project is [here](https://github.com/sambyers/fmcapi-examples).

Usage of the script is really simple: `usage: rmv_objects.py [-h] server username password object_type regex`

I made the repo general to just examples of using this great library. I also made the rmv_objects script general by allowing the user to provide a regular expression for matching on the object names that should be deleted.

If you'd like to improve the script, please submit a pull request! If something doesn't work, submit an issue!

Thanks for reading!