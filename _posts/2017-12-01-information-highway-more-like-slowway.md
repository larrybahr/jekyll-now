---
layout: post
title: Information Highway, More Like Slowway
---
# Intro
It has happened to all of us. You are driving down the Highway going the speed limit (or maybe a lot faster) when you come up on a slow down. You try to look ahead to see what is going on, but you cannot see anything. After 15 minutes of going 5 mph, traffic finally picks up. But wait! There was nothing causing the slow down? This is the problem I was running into after a new production web server was only serving content at dial up speeds. This article will walk through an overview of the process I took to find the slow down. 

After weeks of develupment on a new proxy for our webserver, it was time for it to move to production. I exported the virtual machine and send the file to IT to be imported on one of the production servers. The next day I was informed by my suppervisor that the new proxy was unusably slow! As most programmers would do, I assumed that it could not be my code and explained I had a working example in the develupment lab. After a quick demo my supervisor was covinced it was not our issue and email IT asking them to fix the networking issue. With in a hour he got a reply stating that a diagnostic was ran and found no issues on their end so it must be our fault. So now our team was in the middle of the blame game with the clock ticking down to the new proxy launch date. We did not have time to argue. We needed to pin point the issue as much as possible and provide indisputable evidence!

Here is a general map of the setup:

**client <=Internet=> public proxy <=VPN=> private proxy <=LAN=> web server**

Some things to note about the enviornment:
* Development is done on a Windows 10 PC
* All servers run CentOs 5, 6, or 7
* I have no access to physical hardware, but I can remote into all servers
* While I cannot see all the network cabling I can assume it is capable of Gigabit speeds

# TL;DR
You can use [sar](https://linux.die.net/man/1/sar), [iperf](https://iperf.fr/), [sockperf](https://github.com/Mellanox/sockperf) to help identify slow downs in a network.

# How Many Miles is this Slow Down! (Find the BottleNeck)
use sar to find the computer that is slowing things down

First thing to do when you are stuck in a traffic jam is pull out Google Maps to see how long you will be stuck. Unfortunatly, Google Maps does not have a map of our network yet (it is only a matter of time until they index the whole world), so the next best thing is to use [sar](https://linux.die.net/man/1/sar). By running ```sar -n DEV 1``` (or ```sar -n ALL 1 100``` on CentOs 5) on all the servers in the chain and trying to load the webpage on the webserver.

![_config.yml]({{ site.baseurl }}/images/posts/information-highway-more-like-slowway/sar-test.png)

# Pimp My Ride (Computer Hardware)
find if the network card is gigabit

# Do We Need Another Lane? (Throughput)
Throughput

```shell
iperf -s -p 6284
iperf -c r7601246.rva.reyrey.net -p 6284 -t 60
```

# Are We There Yet? (Latency)
Latency and Jitter

```shell
sockperf server -p 6284 --tcp
sockperf under-load -i 10.2.10.14 -p 6284 -t 60 --tcp
sockperf under-load -i 198.19.31.15 -p 6284 -t 20 --tcp --full-log /home/uccop/bahrlarr/jitter.csv
```

# Are We Lost? (Dropped Packet)
packet loss, QoS
