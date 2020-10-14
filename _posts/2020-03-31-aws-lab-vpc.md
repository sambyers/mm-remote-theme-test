---
title: "AWS Lab VPC Automation"
date: 2020-03-31
categories: networking aws automation terraform ansible
---
_Simple automation for my AWS lab to connect to my home lab_

## AWS Lab VPC

Recently, I’ve moved a lot of my research of systems, security, and automation into IaaS providers. Specifically, AWS. In the past I’ve used ESXi to host VMs for research and labs. It works well and when you have easy access to hardware and rack space, it becomes the path of least resistance. My recent projects have had dependencies on resources across the internet, so I’ve chaffed a bit against the perimeter security stack. Usually, I use a lab at my employer and I am subject to either no internet access or limited access through a proxy.

While attempting to alleviate my access constraints, I started to make use of EC2, Lambda, S3, VPC, and other AWS services. I’ve also used Heroku, Azure functions, Umbrella, Duo, and others. Using IaaS as the core of my lab, I'm able to roll my own security stack and make policies and flows to my needs.

There are tradeoffs, however. Now labs cost me money to run. I’m testing in a lab and not running a production application, so there is a need to keep costs to a minimum. I chose the smallest instance or lowest tier that will support the lab. Before I had dedicated hardware like a C220 M4 with 18 core Xeon CPUs and 256GB of RAM. Now I have t2.micro instances on shared hardware with 1 vCPU and 1GiB of memory. This can mitigated by being smart about how I deployment whatever system I'm testing or solution I'm developing.

So, to enable all of this labbing in AWS and make it easier to deal with some of the tech (IoT devices, other gear at my house, etc.) that doesn't like NAT, CGNAT, etc. I needed to extend the lab VPC OTT to my home lab. I wanted a method to spin up a lab VPC and IPSec tunnels that was automated so I could tear it down when I'm done. This saves me time and money.

My home edge device is a Cisco ASA 5506-X, so that is what I used as the tunnel endpoint on premise. I used AWS customer gateway and AWS VPN gateway as the cloud lab tunnel endpoint. For tooling, I used Terraform for AWS and Ansible for the ASA.

There were some novelties I created for this to work. I wrote a quick and dirty script to parse output from Ansible for Terraform to consume and I use Python for that. I also created a command parser for the ASA in Ansible to be able to gather interface facts to be able to tell AWS what my VPN endpoint IP address is.

The scripts are in this [repo](https://github.com/sambyers/aws-lab-vpc).


## Other Ways to do this
I looked into a pure Ansible option using roles from the [Ansible network module](https://github.com/ansible-network). There is a role called cloud_vpn that does exactly what these scripts do. There doesn't seem to be support for ASA as an endpoint and the documentation wasn't clear.

## Ansible details
To gather interface facts from the ASA firewall, I used the Ansible [network-engine](https://galaxy.ansible.com/ansible-network/network-engine) command_parser. The ASA _show interface_ command parser is in the file show-interface.yaml. It borrows mostly from examples from RH. The regex is quickly done but works. The parser simply parses the output from the command _show interface_ and produces structured json. This data is used to gather interface information and the playbook copy-asa-interface-facts.yaml copies the json to custgw_address.json.

The playbook that configures the ASA is asa-s2s-vpn.yaml. It reads from the vars file ansible_vars.yaml created by the Terraform configuration from the template ansible_vars.yaml.tpl.

## Address parsing script details
A quick and dirty script to reduce custgw_address.json down to simply the outside address for easy inclusion into TF configuration. E.g. "address": "1.1.1.1"

## Terraform details
The Terraform configuration, labvpc.tf, reads the address variable from custgw_address.json and provisions a VPC, Customer gateway, VPN gateway, VPN connection, and sets the propagation option in the VPC routing table for the new VPN gateway.

## Why Ansible and not Terraform to configure ASA?
Terraform's [ASA support](https://www.terraform.io/docs/providers/ciscoasa/index.html) is limited to ACLs and object groups. Terraform requires ASA API. This isn't a constraint in my environment but I wanted this to work almost everywhere. Ansible still seems better suited to configuring individual systems than Terraform. Although, I looked through Terraform's [provider docs](https://www.terraform.io/docs/providers/) and am impressed with the system coverage.

## Why Terraform and not Ansible to configure AWS VPN?
Terraform is stateful and after using a few Ansible playbooks for AWS, state can be very helpful. Terraform's documentation is simple and to the point. I am interested in learning the tool. Terraform has potentially broad application across modern IT infrastructure (IaaS, PaaS, SaaS) -essentially, my effort in learning goes further. The last thing I'll mention about Terraform is how much I liked the declarative style of HCL. It was really magical to write a simple description of what I wanted and have Terraform make it so.

## All done!
Thank you for reading! I'm looking forward to automating more of AWS with Terraform and also taking a look at Cloudformation. Terraform is a dream to work with in AWS. In general, the IaC paradigm is really nice to work in when compared to a more traditional model of configuration in the infrastructure holding the state. More to come!
