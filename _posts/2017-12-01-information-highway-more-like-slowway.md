---
layout: post
title: Information Highway, More Like Slowway
---
It has happened to all of us. You are driving down the Highway going the speed limit (or maybe a lot faster) when you come up on a slow down. You try to look ahead to see what is going on, but you cannot see anything. After 15 minutes, traffic finally picks up. But wait! I did not see anything causing the slow down! So why was traffic so slow? This is the problem I was running into after a new production web server was serving content at dial up speeds despite it working flawlessly in the lab. This article will walk through an overview of the process I took to find the slow down. 

# Brief Background
After weeks of develupment on a new proxy for our webserver, it was time for it to move to production. I exported the virtual machine and send the file to IT to be imported on one of the production servers. The next day I was informed by my suppervisor that the new proxy was unusably slow! As most programmers would do, I assumed that it could not be my code and explained I had a working example in the develupment lab. After a quick demo my supervisor was covinced it was not our issue and emailed IT asking them to fix the networking issue. With in a hour he got a reply stating that a diagnostic was ran and found no issues on their end so it must be our fault. So now our team was in the middle of the blame game with the clock ticking down to the new proxy launch date. We did not have time to argue. We needed to pin point the issue as much as possible and provide indisputable evidence!

Here is a general map of the setup:

**client <=Internet=> public proxy <=VPN=> private proxy <=LAN=> web server**

Some things to note about the enviornment:
* Development is done on a Windows 10 PC
* All servers run CentOs 5, 6, or 7
* I have no access to physical hardware, but I can remote into all servers
* While I cannot see all the network cabling I can assume it is capable of Gigabit speeds

# TL;DR
You can use [sar](https://linux.die.net/man/1/sar), [iperf](https://iperf.fr/), [sockperf](https://github.com/Mellanox/sockperf), and [tcpdump](https://linux.die.net/man/8/tcpdump)/[Wireshark](https://www.wireshark.org/) to help identify slow downs in a network.

# How Many Miles is this Slow Down! (Find the BottleNeck)
use sar to find the computer that is slowing things down

First thing to do when you are stuck in a traffic jam is pull out Google Maps to see how long you will be stuck. Unfortunatly, Google Maps does not have a map of our network yet (it is only a matter of time until they index the whole world), so the next best thing is to use [sar](https://linux.die.net/man/1/sar). I ran sar on all the servers in the chain and trying to load the webpage on the webserver.

{% highlight shell %}
sar -n DEV 1 # or sar -n ALL 1 100 on CentOs 5
{% endhighlight %}

![_config.yml]({{ site.baseurl }}/images/posts/information-highway-more-like-slowway/sar-test.png)

# Pimp My Ride (Computer Hardware)
After a quick test it appeared that the private proxy server was slowing things down. Knowing that the lab server was also sending traffic through the private proxy server, I knew it was unlikly an issue with the hardware. A quick command quickly confirmed my reservations on blame. NOTE: I also found many other useful commands at https://www.tecmint.com/commands-to-collect-system-and-hardware-information-in-linux/

{% highlight shell %}
lspci
{% endhighlight %}

![_config.yml]({{ site.baseurl }}/images/posts/information-highway-more-like-slowway/hardware-test.png)

At this point I had found the area the slow down was in and confirmed that the servers on either side were capable higher speed transfers. That means the only culprit left is the VPN. But what exaclty is going wrong?

# Do We Need Another Lane? (Throughput)
When I see a car accident, I usually wish there was an additional lane to make up for the bottle neck. With that in mind, the first thing I thought to check was the size of the pipe. Looks like it is time to fire up [iperf](https://iperf.fr/).

{% highlight shell %}
# Run on server
iperf -s -p 6284

# Run on client
iperf -c r7601246.rva.reyrey.net -p 6284 -t 60
{% endhighlight %}

![_config.yml]({{ site.baseurl }}/images/posts/information-highway-more-like-slowway/iperf-test.png)

While far from great, 1.3 Mbits/sec should be plenty for my sub 500kb total file size. I decided this was not my current issue and to move on to the next test after looking at these results and making a note to talk to managment about increaseing the pipe size.

# Are We There Yet? (Latency)
The cause of a slow down could also be from drivers quick to slow down, but slow to speed back up (aka network latency). Ping is usually the perfect tool for the job, but our VPN only had TCP port 6284 open. Sockperf to the rescue!

{% highlight shell %}
# Run on server
sockperf server -p 6284 --tcp

# Run on client
sockperf under-load -i 10.2.10.14 -p 6284 -t 60 --tcp
# --full-log to print everything to file
# sockperf under-load -i 198.19.31.15 -p 6284 -t 20 --tcp --full-log /home/uccop/bahrlarr/jitter.csv
{% endhighlight %}

![_config.yml]({{ site.baseurl }}/images/posts/information-highway-more-like-slowway/sockperf-test.png)

The average latency is ~71 milliseconds with 99.999% of traffic being less than ~109 milliseconds and the lowest latency being ~24 milliseconds. Ideally all traffic would have a maximum latency of 80 milliseconds, but with an average of 71 milliseconds this is hardly the major source of the issue.

# Are We Lost? (Dropped Packet)
We have all been in the situation were the GPS is telling you to take an exit on the highway, but you are not sure if it is talking about this exit or the one right after so you slow down to buy some time to figure it out. Some times packets can get "lost" and dropped altogether (Thank God I do not get dropped from existence when I get lost!).

To find if any packets are dropped, I needed a "road map" with the locations of each packet. Fortunately I have just the tool for that, Wireshark! For the sake of length, complexity, and to not duplicate the already abundant quality information on Google; I will not explain how I set the test up, but rather an overview of what the results ment.

My test was simply to request a test image from the server and watch the packet flow.

![_config.yml]({{ site.baseurl }}/images/posts/information-highway-more-like-slowway/wireshark-chrome-img-download.png)

The first thing I noticed in the packet capture was that for about one tenth of a second there we were nother but "TCP Dup ACK". This was a sign of packet loss.

![_config.yml]({{ site.baseurl }}/images/posts/information-highway-more-like-slowway/wireshark-dup-ack-start.png)
![_config.yml]({{ site.baseurl }}/images/posts/information-highway-more-like-slowway/wireshark-dup-ack.png)
![_config.yml]({{ site.baseurl }}/images/posts/information-highway-more-like-slowway/wireshark-dup-ack-end.png)

 After hours of research and testing a simple network traffic capture proved our inocence. After informing IT about the finding, they were able to quickly relize that they had an incorect QoS setting somewhere giving our data a low priority. And with that issue behind me, it was time to hit the road agian and proceed to my next project.
