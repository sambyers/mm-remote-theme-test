---
title: "Archiving Configurations from Cisco DNA Center"
date: 2019-10-18
categories: api dnac python
---
_Simple utility to archive network device configurations from Cisco DNA Center_

## Why?

Today I am sharing a small utility to help Cisco DNA Center users pull network device configurations out of the platform and save them locally. I'm sharing this as an example of what can be done with the DNA Center API and also as a practical POC for a customer that required this functionality.

## What?

The utility is a python script that uses the REST API of Cisco DNA Center. The Cisco DNA Center API has a lot of functionality. It gives users a unique opportunity to add useful utilities and functions that don’t natively exist on the platform.

In this case, I’m sure configuration archival will be added in the future by the Cisco DNA Center team, but if you have a use case and need to move at speed, the open API enables you to do so.

The API of Cisco DNA Center is well documented and provides a north-bound interface to manage and audit networks and devices that are managed by the system. This API provides capabilities to network operators who were building custom scripts or using configuration management tools before. Hopefully, by aggregating network devices into a central controller, network operators can move faster and go further by standing on the shoulders of the API.

You can find more information about the Cisco DNA Center API at the [developer website](https://developer.cisco.com/docs/dna-center/). Another great resource is Adam Radford’s [blog series](https://blogs.cisco.com/author/adamradford) on using the API. He documents the Cisco DNA Center SDK available from DevNet, device configuration templating, and plug and play. If you want to get started right away, head over to the [API docs](https://developer.cisco.com/docs/dna-center/api/1-3-3-x/) for version 1.3.3.x.

## How?

You can get the utility on [github.com](https://github.com/CiscoSE/dnac_network_device_config_archiver_poc).

Using the the utility is straightforward:

```
python dnac_network_device_config_archiver_poc.py
```

The utility makes directories for the configurations it downloads from DNA Center: 

```
./DNAC_Config_Archive/2019-10-16 15:36:48.905188/c9200.site1.company.com
```

The only configuration required is the DNA Center FQDN/IP and credentials in the config file dnac_config.py:

```
DNAC = 'sandboxdnac.cisco.com'
DNAC_PORT = '443'
DNAC_USER = 'devnetuser'
DNAC_PASSWORD = 'Cisco123!'
```

## Now what?

This utility can easily be extended to suit your needs, like copying the configurations to a remote location.

Also, now that you've seen the DNA Center API in action, do you have any ideas or use cases it could be used for?

Thanks for reading and I hope this was helpful!
