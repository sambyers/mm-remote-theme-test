---
title: "Model-Driven Telemetry into the TIG stack"
date: 2020-10-01
categories: devnet mdt telemetry telegraf influxdb grafana csr yang netconf
---
_Using the TIG stack to consume and visualize MDT_

## Model-driven Telemetry 

Model-driven telemetry or MDT is streamed or polled operational data from systems structured by a certain data model. Streamed data is emitted from a system and transported to a receiver either when something changes or at an interval. Polled data is retrieved by a receiver from a system at an interval. A data model describes how data is represented and accessed.

In more concrete terms, the data models used for MDT are YANG data models. YANG is a data modeling language used to model configuration and state data on network systems. Refer to RFC 6020 for the details. The transport mechanisms for MDT are typically raw TCP or gRPC (HTTP/2). They can both be wrapped in TLS. The data is encoded in Google protocol buffers.

The system emitting the streaming data is called a publisher. The system consuming the data is called the receiver. The system establishing the subscription is called the subscriber. And finally, the system that creates subscriptions but doesn’t receive them is called a controller.

For streaming sessions there are a few options: dial-in and dial-out. With a dial-in, sometimes called dynamic, session the receiver initiates the session and the publisher pushes the data to the receiver. With a dial-out, sometimes called configured, session the publisher opens the session and pushes the data to the receiver. Dial-out is similar to the publisher and subscriber pattern. The receiver of the data (or another system) tells the publisher of the data to stream data to the receiver at an interval or on change.

NETCONF is used for dial-in sessions and gRPC is typically used for dial-out sessions.

## Consuming, Storing, and Showing Data

Recently, I was labbing MDT and I had to pick the software to consume the data coming from some network systems. I read through a number of docs on different stacks that do collection, storage, and visualization of streaming data. On my shortlist was ELK and TIG. Looking through some blogs and the Telegraf docs I found that there is an input plugin for MDT. This plugin sold me on building my lab with TIG.

The TIG stack is made up of Telegraf, Influxdb, and Grafana (TIG). Telegraf is a collection and processing engine for metrics. It’s very powerful and has wide support. Influxdb is a time series database. Grafana is an analytics and visualization front end. When these components are wired together they are very useful for operational data.

In the next sections I'll detail a lab I used to demo MDT and the TIG stack. The code is in a repo in the resources section and has scripts to setup the TIG host on Ubuntu, configure a network device for NETCONF and MDT subscriptions.

I hosted my lab on Cisco Modeling Labs or CML. It allows drag and drop or API-based lab creation. There's a management network, which all nodes in the lab connect to on their first NIC. This network is to access the lab nodes from my laptop. There's also a transport network, which all nodes connect to on their second NIC. This network is meant to carry the telemetry data from publishers to receivers.

## Lab Environment

In my demo lab we are running TIG and a CSR1000v on Cisco Modeling Labs (CML). There is an example topology file in the repo to get you started (topology.yaml). To generate an Ansible inventory from our CML lab, we can use cmlutils. It's a great utility when working with CML.

A prerequisite to generating an Ansible inventory is to have nodes tagged. The tag cmlutils is looking for is ansible_group=GROUP_NAME, where GROUP_NAME is the name of the host group you want to put the node in. In this lab, I chose _routers_ for network devices and _tig_ for tig hosts.

To generate an Ansible inventory, use this command:

``` shell
cml generate ansible
```

In the lab, we’re going to configure MDT on the CSR to stream CPU utilization and interface data to our TIG stack. We also have to configure Telegraf to use the Cisco MDT input plugin.

![]({{"/assets/images/mdt.png"}})

Telegraf is our receiver. The CSR1000v is our publisher. My laptop is the management controller.

## Model-driven Telemetry Configuration

Configuring MDT is straight forward and in this example I’m going to demonstrate the process on a IOS-XE (CSR1000v). I’m going to use two methods to configure MDT: CLI and NETCONF.

We don’t want to have to poll the device or even dial-in for data to be streamed back on-demand. We want the network device to stream data on an interval to our receiver. So, we’re going to use the dial-out method on IOS-XE.

For CLI, configuring MDT looks like this:
```
telemetry ietf subscription <ID>
    Stream yang-push
    Filter xpath <xpath>
    Update-policy on-change | periodic <centiseconds>
    Encoding encode-kvgpb
    Source-address <IP address>
    receiver ip address <ip-address> <receiver-port> protocol <protocol>
```
A few callouts from the configuration:

We are pushing telemetry from the publisher (our router) to the receiver (TIG stack). Subscribers do not have to poll or dial-in to receive the stream.

The XPath argument is the path notation to reach the particular leaf in the YANG model you want to stream telemetry about.

Encoding in our case is going to be key-value pairs encoded with Google protocol buffers (protobuffs, gpb).

The transport protocol we’re using is grpc-tcp. This is gRPC over TCP, which means it is not encrypted with TLS.

For NETCONF, the edit-config RPC looks like this:

``` xml
<config>
  <mdt-config-data xmlns="http://cisco.com/ns/yang/Cisco-IOS-XE-mdt-cfg">
    <mdt-subscription>
      <subscription-id>{{ sub_id }}</subscription-id>
      <base>
          <stream>yang-push</stream>
          <encoding>encode-kvgpb</encoding>
          <source-address>{{ source_ip }}</source-address>
          <period>{{ period }}</period>
          <xpath>{{ xpath }}</xpath>
      </base>
      <mdt-receivers>
          <address>{{ receiver_ip }}</address>
          <port>{{ receiver_port }}</port>
          <protocol>grpc-tcp</protocol>
      </mdt-receivers>
    </mdt-subscription>
  </mdt-config-data>
</config>
```

Now we can run our Ansible playbooks scripts to create MDT subscriptions. The scripts we’re going to run will create MDT subscriptions for CPU utilization and interface stats.

``` shell
foo@bar:~$ ansible-playbook -i YOUR_INVENTORY set_mdt_cpu_util.yml
```
``` shell
foo@bar:~$ ansible-playbook -i YOUR_INVENTORY seet_mdt_intf_stats.yml
```

There are playbooks in the repo to set the NTP server on the router, as well. This is a pretty important step as the router is streaming timestamped data to telegraf. This is a basic requirement of traditional logging systems, as well.

## Setup the TIG stack

Setting up a TIG stack is very easy. It’s so easy, in fact, you don’t even need to know how to actually do it. You can just use an Ansible playbook to perform all of the work for you. If you want to see the manual steps, there are many tutorials on the internet.

Check out my repo in the resources section to see an example of a playbook that provisions the stack (setup_mdt_lab_tig_host.yml). In that playbook, I am also adding dhcpd to serve IP addresses out to our lab network 198.18.1.0/24. Comment out that section if you do not want or need DHCP on your lab network.

In this lab, the tig host is configured as 198.18.1.12/24 on the lab network. The management interface is setup as DHCP.

## Grafana Setup and Dashboard

Now we go to the Grafana dashboard to see our lovely data. The first thing we need to do is add our Influxdb as a data source.

![]({{"/assets/images/grafana-datasrc.png"}})

After adding our data source, we can add panels onto our example dashboard.

![]({{"/assets/images/grafana-example-dash.png"}})

We have data! This is streaming telemetry from our router to our TIG stack. Our interval is set to every 5 seconds and we’re looking at a graph showing the CPU utilization every 5 seconds. That’s fast!

## All Done, For Now

Thank you for taking a look at this demo! I'd also like to thank everyone publishing resources on MDT, as I think this is an important capability to improve observability of infrastructure.

## Resources

[MDT Lab Repo](https://github.com/sambyers/mdt_tig_demo)

[IOS-XE/CSR1000v MDT Configuration Guide](https://www.cisco.com/c/en/us/td/docs/ios-xml/ios/prog/configuration/1610/b_1610_programmability_cg/model_driven_telemetry.html)

[IOS-XE MDT Whitepaper](https://www.cisco.com/c/en/us/products/collateral/wireless/catalyst-9800-series-wireless-controllers/white-paper-c11-743401.pdf)

[XPath in NETCONF and YANG](https://www.tail-f.com/xpath-netconf-yang/)

[Advanced NETCONF Explorer](https://github.com/cisco-ie/anx)

[Cisco MDT Telegraf plugin](https://github.com/influxdata/telegraf/tree/master/plugins/inputs/cisco_telemetry_mdt)

[Cisco Modeling Labs](https://developer.cisco.com/modeling-labs/)

[cmlutils](https://github.com/CiscoDevNet/virlutils)
