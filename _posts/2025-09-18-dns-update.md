---
layout: post
title: Updating Blocked Services in AdGuard Home
subtitle: Using and reading the AdGuard Home API
tags: [homelab, AdGuard]
comments: true
mathjax: true
---

## Intro  
Recently on my home network I have decided to make a change. I have gotten very tired of seeing advertisment after advertisment after advertisment when I use the internet on anything except for my desktop (I use uBlock Origin on my desktop to get around these there). Additionally I want to also better protect my privacy on the internet and my data in general. The best way I have found in my research, short of putting in a full proxy and VPN solution in between the network and the internet, is using a network wide ad blocker, which I chose to use [AdGuard Home](https://adguard.com/en/adguard-home/overview.html) since it was one solution that allowed me to host on my own server.  
  
Now this solution isn't as effective as uBlock Origin on my desktop (mainly due to how the 2 work and limitations in doing blocking based on only DNS) but it still does allow me to filter out some of the more annoying ads that show up on my phone and does still protect my other devices to a certain degree. Additionally one of the nice features within AdGuard Home is that you can also block specific applications or websites if you choose to.  

![](/assets/img/dns-update/adguard_blocked_services.png)  

This is probably pretty useful if you especially have kids and want to limit access to certain sites, but one recent use case I have found it useful for is trying to limit social media usage for myself. For this post I want to walk through how I did this using the AdGuard Home API and why I did this the way I did. 

## How Blocking Services Works?  

The basic implementation of blocking a service is extremely easy and user friendly. All you do is login to AdGuard, and then under Filters go to Blocked Services and in here you see all the services you can block out of the box. If you see one you want to block you just toggle it and click save and any DNS requests going to those services will be blocked for any device attempting to on the network. Additionally if you want for there to be a pause in the blocking (again probably something pretty useful for if you have kids and want to let them use social media or certain services only during certain times) then you can choose a schedule to unblock the services for a limited time.  

![](/assets/img/dns-update/adguard_blocking_pause.png)  

For me and my use case what I specifically wanted to do was block social media only in the morning when I am getting ready for work. So I wanted a schedule of Monday-Friday 5:30AM-8:30AM for the block to be in place but all other times for the block to not be in place. The issue with this is the out of the box implementation for AdGuard Home only allows for pauses to be within a single day and cant extend into the next day. I found this out when I tried to put in a blocking pause to start at 8:30AM and go to 5:30AM but then I got an error saying that the ending time had to be after the start time so this wouldn't work.  

## AdGuard Home API  
In researching an alternative solution to this problem I found out about the AdGuard Home API that is available. Unfortunately for me I couldn't find any public documentation of the API so I had to read the OpenAPI [yaml](https://github.com/AdguardTeam/AdGuardHome/blob/master/openapi/openapi.yaml) instead to understand it.  

I have had to read these types of templates before but never so in depth where I was actually trying to understand and use the APIs defined in the document itself which turned into a nice learning oportunity.  

Within the API one nice thing from the end user perspective is that the API uses just Basic Auth, so all I need to do in order to use the API is pass my username and password as part of the headers. In terms of security, there are better ways of authenticating to APIs but for the purposes of a home implementation of a DNS/DHCP server that isn't publicly exposed I am okay with it. Another thing that I noticed too was that under the 'servers' declaration it states the url is '/control' meaning for each of the endpoints I need to prepend that before the endpoint. For example if I wanted to hit the dns_info endpoint I would use the URL "hxxp://SERVER_IP.com/control/dns_info" instead of just "hxxp://SERVER_IP.com/dns_info".

For actually finding the endpoint I need to use to change the services blocked I did a CTRL+F search on the yaml file for "services" which returned for me a block of API endpoints that all appeared to deal with this.  

![](/assets/img/dns-update/adguard_api_yaml_search.png)  

A lot of these endpoints ended up being deprecated as seen in their description, but the 2 that I needed had an implementation in [/blocked_services/get](https://github.com/AdguardTeam/AdGuardHome/blob/5e7e0092ac3b2f4bbdb30aebc5b67dc9e7995249/openapi/openapi.yaml#L1093) and in [/blocked_services/update](https://github.com/AdguardTeam/AdGuardHome/blob/5e7e0092ac3b2f4bbdb30aebc5b67dc9e7995249/openapi/openapi.yaml#L1106). Another one I found useful just for the beggining stages of verifying the format of what to pass to the "update" endpoint was the [/blocked_services/all](https://github.com/AdguardTeam/AdGuardHome/blob/5e7e0092ac3b2f4bbdb30aebc5b67dc9e7995249/openapi/openapi.yaml#L1047) which just returns a list of all the services that you can block.  

The next step was to figure out the schema that each API expects when a request gets sent to it. The /blocked_services/get and /blocked_services/all endpoints don't have anything since they are just APIs used to return data and don't need anything passed into them for that. The /blocked_services/update does have an expected schema though since it is a PUT request that is updating data so it needs information passed to it. Within this endpoint definition there is a schema declaration with a reference to a schema called "BlockedServicesSchedule". Doing another CTRL+F search found the declaration in the file and found the expected formatting there.  

![](/assets/img/dns-update/adguard_update_schema.png)  

Based on this, the expected data should be in JSON format and can contain 2 keys "schedule" and "ids". The "schedule" key I won't need to use since I am not using schedules so I never pass any data using it. The next one is the one I care about called "ids" which expects the value to be an array of strings.  

## Using the API with Python  
I now know what endpoints I need to use for updating AdGuard Home along with their schemas, but now I need to create a script that will run on a regular basis and use these endpoints. I am comfortable with python so that is what I chose to write the script in. The basic outline of the script I had generated with ChatGPT to speed up the process, but the final script after I made modifications can be seen below:  
```
import requests
import sys

ADGUARD_URL = ""                   # Change to your AdGuard Home URL
USERNAME = ""                      # Change to your username
PASSWORD = ""                      # Change to your password

IDS = ["tiktok", "instagram", "facebook"]

def login(session):
    """Authenticate and return True if successful."""
    resp = session.post(
        f"{ADGUARD_URL}/control/login",
        json={"name": USERNAME, "password": PASSWORD}
    )
    print(resp.text)
    return resp.status_code == 200

def add_rules(session):
    payload = {"ids": IDS}
    r = session.put(f"{ADGUARD_URL}/control/blocked_services/update", json=payload)

def remove_rules(session):
    payload = {"ids": []}
    r = session.put(f"{ADGUARD_URL}/control/blocked_services/update", json=payload)

if __name__ == "__main__":
    with requests.Session() as s:
        if not login(s):
            print("Login failed!")
            sys.exit(1)

        if sys.argv[1] == "block":
            add_rules(s)
            r = s.get(f"{ADGUARD_URL}/control/blocked_services/get")
            print(r.status_code, r.text)
        else:
            remove_rules(s)
            r = s.get(f"{ADGUARD_URL}/control/blocked_services/get")
            print(r.status_code, r.text)



```

The scipt logic is pretty basic overall of Login -> if "blocked" is passed as part of the arguments -> update the blocked services with those in IDS -> otherwise update the blocked services to be empty.  

The next step in this was hosting the script on my Elastic server. My Elastic server is just a server I use for hosting random scripts and docker containers so this one makes sense to me to add it there. I did debate just having the script on the AdGuard Home server itself, but for my personal organization I made the decision to put it on the Elastic server. I then created 2 Cron jobs to run daily at 5:30AM and 8:30AM to execute the script.  

![](/assets/img/dns-update/adguard_cron.png)

## End Result  

All said and done now, I have monitored the AdGuard Home instance and verified that it is working as expected through manual testing but also through checking the instance while it should be blocked.  

![](/assets/img/dns-update/adguard_services_blocked.png)  

I think this is something that will be useful to me in the short and long term but also this opens up other options to me now in the future if I want to do other projects using the AdGuard Home API.