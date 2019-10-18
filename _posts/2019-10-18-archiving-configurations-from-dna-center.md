---
title: "Archiving Configurations from DNA Center"
date: 2019-10-18
categories: api dnac python
---
_Simple utility to archive network device configurations from DNA Center_

## Why?

Today I am sharing a small utility to help DNA Center users pull network device configurations out of the platform and save them locally. I'm sharing this as an example of what can be done with the DNA Center API and also as a practical POC for a customer that required this functionality.

## What?

The utility is a python script that uses the REST API of DNA Center. The DNA Center API has a really good API with a lot of functionality. It gives users a unique opportunity to add useful utiliies and functions that don't natively exist on the platform.

In this case, I'm sure configuration archival will be added in the future by the DNA Center team, but if you have a use case and need to move at speed, the open API enables you to do so. 

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

This utility can easily be extneded to suit your needs like copying the configurations to a remote location.

Also, now that you've seen the DNA Center API in action, do you have any ideas or use cases it could be use for?

Thanks for reading and I hope this was helpful!