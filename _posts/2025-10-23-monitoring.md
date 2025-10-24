---
layout: post
title: Monitoring Server Health
subtitle: How and why I implemented a monitoring stack
tags: [homelab, monitoring]
comments: true
mathjax: true
---

### Introduction

I have recently encountered a new issue in supporting my home lab environment, specifically for my Elastic server. The server has run out of drive space and I had no idea until I realized I wasn't receiving emails from some of the automated jobs running on that server. The issue itself was relatively easy to resolve with it being a virtual server in that I had to clear out logs and files no longer needed, add some more space onto the virtual hard drive, and then modify the virtual drive within Ubuntu to use that extra space that it was provided. Afterwards though, this forced me to realize I needed to do something I hadn't really done until now, and that was implement a monitoring solution.

### How should I monitor these servers?

That was the first question I had to ask myself. I knew from personal experience that going into each server and checking the file size and the CPU load and anything else I was worried about just wasn't practical even if I have a generally low amount of servers that I maintain. I could install and host monitoring software on each host, each server with their own dashboard, which would get me closer to what I want in that I could easily look at the data and understand what each server looks like, but going to a new URL and dashboard for each server would be tedious and also would go down if the server went down too so getting alerts of issues would not be possible. So the solution I needed was an independent server that could monitor each server in the environment and then make a dashboard that will let me look at each server to understand how it is performing.

Now that I knew what I wanted in the solution, I could begin researching for how to implement. Using ChatGPT, this was something I was able to get done pretty quickly, and I was able to find a solution that looked good in using [Prometheus](https://prometheus.io/) for collecting the data and then using [Grafana](https://grafana.com/) for visualizing this data. I liked this solution mainly due to the simplicity of it in that I can implement it with supported Docker containers, but also I have used Grafana before too so I am familiar with it.

### Implementing the solution

At a high level what I needed to do was:

1. Create a new server to host the monitoring software
2. Install Prometheus and Grafana on the server and configure the services to run correctly
3. On the server I want to monitor, install Node Exporter (part of Prometheus family to connect to the central collector) and configure it to point towards the monitoring server

Now the server creation step was the easiest with it being something I've had to do often enough in ProxMox. I gave it mostly the default values for everything except the biggest difference being I gave it 75Gs of storage with the idea of future proofing it a bit so that it will have enough space to hold all the information sent to it.

Installing Prometheus and Grafana though gave me an opportunity to try something I hadn't until this point in using Docker Compose for installing the applications. Up until this point I had installed containers on a few servers using the basic docker commands, mostly just custom ones to do random tasks, but I hadn't ever used Docker Compose yet. The way I understand Docker Compose, and the reason I liked trying to use it for the first time, is a way to speed up building and running Docker containers where you can create a YAML file to hold the configurations of the container and then Docker will use that outline to create the container through a single simple command. So instead of having for each container I want to create having to `docker run -d IMAGE -v volumes:volumes -p 8080:80 -e ENV1:Value -e ENV2:Value` I can just define it in a single file and then run `docker compose -f compose.yml up`

Fortunately with this implementation, it is decently popular and there are a lot of other documents out there on how to set up this configuration so I was able to make the compose file pretty quickly.

```yaml
version: "3"
services:
  prometheus:
    image: prom/prometheus:latest
    container_name: prometheus
    restart: unless-stopped
    volumes:
      - ./prometheus/prometheus.yml:/etc/prometheus/prometheus.yml:ro
      - prometheus_data:/prometheus
    ports:
      - "9090:9090"

  grafana:
    image: grafana/grafana:latest
    container_name: grafana
    restart: unless-stopped
    environment:
      - GF_SECURITY_ADMIN_USER=admin
      - GF_SECURITY_ADMIN_PASSWORD=admin
    ports:
      - "3000:3000"
    volumes:
      - grafana_data:/var/lib/grafana
      
  node_exporter:
    image: prom/node-exporter:latest
    container_name: node_exporter
    restart: unless-stopped
    network_mode: "host"   # exposes port 9100 directly
    pid: "host"
    volumes:
      - /proc:/host/proc:ro
      - /sys:/host/sys:ro
      - /:/rootfs:ro
    command:
      - '--path.procfs=/host/proc'
      - '--path.rootfs=/rootfs'
      - '--path.sysfs=/host/sys'
      - '--collector.filesystem.ignored-mount-points=^/(sys|proc|dev|host|etc)($$|/)'


volumes:
  prometheus_data:
  grafana_data:


```

Walking through this YAML file, it is split between 3 main blocks. The first creates the Prometheus instance that will collect the data and defines 2 volumes that should be mapped onto the host server. The first one pointing at prometheus/prometheus.yml is the Prometheus configuration file that lets Prometheus know what endpoints it should be looking for to get information from (defined in the targets section using the host IP) and what cadence it should be scrapping the data from Node Exporter (defined in scrape\_interval).

```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'node_exporters'
    static_configs:
      - targets:
          - '192.168.0.XX:9100'
          - '192.168.0.XX:9100'

```

The second volume defined just acts as a way of backing up the Prometheus data if the container every goes down. Since containers are ephemeral if I were to stop the container for maintenance or if the server was shut down by mistake, any of the data that would be stored would be lost. So by mounting the `/prometheus` directory within the container as `prometheus_data` on the server, I don't have to worry about this. This is also the reason why for this volume I allow Docker to have write capabilities, compared to the `prometheus.yml` file which I only give read rights through the `:ro` tag at the end of volume declaration.

The second block in the Docker Compose file, creates a Grafana instance on the server. In the Environment section I define 2 environmental variables that set the administrative username and password (for this file I just set them to the Grafana defaults, which would cause a prompt to have them changed on first login). Then once again I mount a drive on the server to one in the container for saving the data.

The third block, installs the Node Exporter agent on the server. As I mentioned earlier this is what is installed on every server I want to monitor, so for every other server their Docker Compose file would just be this block. The volumes here are mounted as read only and each of the directories mounted contain different information about the server performance such as memory usage, CPU usage, or disk usage. Then we also pass a few commands in the container to make sure that Node Exporter is pointing at the right folders in the container and pass a Regex expression to ignore virtual filesystems.

At the end of the `compose.yml` file I also have a block called volumes that has the 2 directories defined earlier for storing the Prometheus and Grafana data in. This block is just explicitly declaring the directories that we want Docker to use, but isn't necessary as Docker will figure it out.

### Visualizing the data

Normally with Grafana any visualization you would want has to be created manually, but Grafana does have [community dashboards](https://grafana.com/grafana/dashboards/) pre-made and available for everyone. One such dashboard is the [Node Exporter Full](https://grafana.com/grafana/dashboards/1860-node-exporter-full/) dashboard that automatically takes in Prometheus data and plots it accordingly. After importing the dashboard the data was pulled and displayed like the following (let the containers run for over a day to make sure there was a healthy population of data):

![](/assets/img/monitoring/Grafana.png)  

There is much more data available if you really want to dive deep into different aspects of the system, but for my purposes this top high level view that it provides right away is what I need. I can see that there is a good bit of space available on the monitoring server as expected, and that in general it is maybe provisioned too many resources based on the CPU load and Memory load so I can take away some resources from this server.

### Moving forward

Now that I have all of this working for the monitoring server, I decided to put the Node Exporter agent onto my elastic server to make sure I could get information from another server. I already had Docker installed on the server so I just had to put the following Docker Compose file on the server to install Node Exporter:

```yaml
version: "3"
services:
  node_exporter:
    image: prom/node-exporter:latest
    container_name: node_exporter
    restart: unless-stopped
    network_mode: "host"   # exposes port 9100 directly
    pid: "host"
    volumes:
      - /proc:/host/proc:ro
      - /sys:/host/sys:ro
      - /:/rootfs:ro
    command:
      - '--path.procfs=/host/proc'
      - '--path.rootfs=/rootfs'
      - '--path.sysfs=/host/sys'
      - '--collector.filesystem.ignored-mount-points=^/(sys|proc|dev|host|etc)($$|/)'
```

After starting the container with the `docker compose -f compose.yml up` command Node Exporter was started and then after a few minutes I re-freshed my Grafana dashboard and I had a new option under Nodename for elastic. Now for any other server I want monitored I just need to do the same and I'll have this monitoring capability.

This implementation also gets me to a more healthy spot, but isn't perfect yet. For one thing I still have to manually check the dashboard for information, so implementing alerting would be a big help in getting messages right away when issues occur. In the future that is what I will be looking into but for the moment I have taken a good step forward.