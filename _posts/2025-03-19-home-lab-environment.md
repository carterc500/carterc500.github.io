---
layout: post
title: Home Lab Environment
subtitle: What my environment looks like
tags: [homelab]
comments: true
mathjax: true
---

## Intro
The purpose of this is to simply layout what my homelab environment looks like as of today. The main purpose of my lab environment is for malware analysis, but I also have one machine at the moment that is used for running docker containers that will be ignored when going through everything.  

## Hardware & Hypervisor
First up is going over the basic infrastructure that everything is running on. The server itself is pretty basic as it is a Dell Optiplex 7050 that I bought from my local university's surplus store. The specs on the machine are pretty good though with me choosing one of the higher end models that they had at the store and then recently choosing to upgrade some specs like RAM and the storage capacity.
### Specs:
* CPU: 8 x Intel(R) Core(TM) i7-7700 CPU @ 3.60GHz
* RAM: 48 GB DDR4
* Storage: 2 separate 500 GB SSDs for a total of 1 TB of storage
* Hypervisor: [Proxmox Version 8.3](https://www.proxmox.com/en/)

## Network
Since this is a homelab environment all of this is running on my own LAN. However due to the nature of doing malware analysis I have also taken more precautions by having another subnetwork behind a pfSense firewall that I will be calling Malware-Net. Malware-Net, as the name describes, is the place where analysis will actually be taking place and has been setup so that when malware detonates it should not be able to touch other endpoints on the network.  
  
![](/assets/img/network diagram.drawio.png)  
  
## Hosts
For individual hosts I am running 2 in Malware-Net. 1 is a Windows VM that has been configured to run [Flare-VM](https://github.com/mandiant/flare-vm) which will be used for mainly detonating Windows malware within it. The other is a Linux box that is running [REMnux](https://remnux.org) and the plan for that one is to be used more for static analysis, but will also be used for detonating malware as well as needed. 
   
One additional resource that is going to come up from time to time is my Graylog server. [Graylog](https://graylog.org/) is another SIEM option similar to Splunk and that resource is running from outside Malware-Net on my regular LAN. Currently it mainly just injest logs from the pfSense firewall and there are some rules around extracting some of the data to custom fields. In the future I would like to setup some alerting functionality and have some custom dashboards created but currently there are no specific plans. 