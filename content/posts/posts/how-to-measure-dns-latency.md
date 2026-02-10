---
title: "How to measure DNS latency in Linux with tcpdump"
date: 2023-08-14
categories: 
  - "uncategorized-en"
tags: 
  - "cloudflare"
  - "cloudflare-family"
  - "dns"
  - "google"
  - "latency"
  - "linux"
  - "opendns"
---

In this case I was trying to identify which DNS resolver would be the best for my infrastructe in remote sites.

Usually configuring 8.8.8.8 would be ok, yes is simple, but there are a couple of things going on in the background that deserves some checking if you want to achieve best performance. Which was my case since we have high perfomance robots in the remote sites infrastructure.

So the first step is to measure right? If you don't measure you don't know if you are improving or even what to improve.

In the remote sites I only and a very small Linux distro running in routers and luckly tcpdump was included. So creating a shell script was the way to go.

You can find the script [here](https://github.com/aegiacometti/dns_latency).

The script will measure DNS response latency from multiple remote providers and the local router which has a DNS daemon included.

It uses TCPDUMP timestamps with microsecond fractional resolution, then capture the traffic while issuing 3 nslookup query per provider, discard first nslookup which is usually higher because DNS cache might not exist yet, and finally calculate the average.

Basically script will match 2 lines (these number of tests can be customized in the script)and calculate the time difference based on the network timestamps. These network timestamps is the more precise meassure you can get, since it eliminates any CPU/memory influence at the local host.

I'm going to compare the resolution latency for 3 providers and local.

1. Google 8.8.8.8
2. Cloudflare 1.1.1.1
3. Cloudflare Family 1.1.1.3 <--- this is super cool and free service which includes Malware and/or adult content filter by blocking the DNS request. Check it at this [link](https://blog.cloudflare.com/introducing-1-1-1-1-for-families/).
4. And my local Router 172.21.192.1.

Now caching is a key component in DNS resolution, because the resolver will keep a local cache of the IP address the first time it send the query, and then it will not ask again to the authoritative DNS resolver for that record as long as the TTL is not expired. Instead it will reply with the local cached value. We will see that in action in a second.

Below is the script output, and from there I will comment the important parts and a conclusion.

Pay attention to the Response time for Test 1 of each provider. They all take longer than the rest of the tests, because is the first query and they need to forward the query to the root resolver of the record. The following 2 tests will always be lower because they use their cache.  

```shell
adrian@wsl:~/projects/dns_latency$ ./dns_latency.sh 
-- Hostto check: adriangiacometti.net
   Protocol: udp
   Port: 53
-- Provider: Local_Default_Gateway
-- Provider IP: 172.21.192.1
- Test 1 - Discarding this first test from the computation
  Response time is: 49.640000 milliseconds
- Test 2
  Response time is: 1.296000 milliseconds
- Test 3
  Response time is: 1.008000 milliseconds
==> Average Local_Def_Gw response time: 1.152 milliseconds
-- Provider: Google
-- Provider IP: 8.8.8.8
- Test 1 - Discarding this first test from the computation
  Response time is: 42.809000 milliseconds
- Test 2
  Response time is: 29.476000 milliseconds
- Test 3
  Response time is: 31.689000 milliseconds
==> Average Google response time: 30.5825 milliseconds
-- Provider: Cloudflare
-- Provider IP: 1.1.1.1
- Test 1 - Discarding this first test from the computation
  Response time is: 52.256000 milliseconds
- Test 2
  Response time is: 24.839000 milliseconds
- Test 3
  Response time is: 21.443000 milliseconds
==> Average Cloudflare response time: 23.141 milliseconds
-- Provider: Cloudflare-Block
-- Provider IP: 1.1.1.3
- Test 1 - Discarding this first test from the computation
  Response time is: 56.212000 milliseconds
- Test 2
  Response time is: 23.433000 milliseconds
- Test 3
  Response time is: 23.669000 milliseconds
==> Average Cloudflare-Block response time: 23.551 milliseconds
-- Provider: OpenDNS
-- Provider IP: 208.67.222.222
- Test 1 - Discarding this first test from the computation
  Response time is: 65.719000 milliseconds
- Test 2
  Response time is: 32.022000 milliseconds
- Test 3
  Response time is: 28.349000 milliseconds
==> Average OpenDNS response time: 30.1855 milliseconds
```

Now compare the average response time of all the providers.

As you can see the local provider is way less. This is because the local provider is local to your network, meaning the reply answer is close to you, while any of the other replies from the public providers are still far away from you. There is big network distance difference.

As a conclusion, the local resolution is always the best, and then you can choose which public provider to use based on your results.

Usually, their times will be in this order.

1. Local (by far)
2. Cloudflare
3. Cloudflare family (with a small increase because it has to do its job of checking if the records have to be blocked)
4. Google
5. OpenDNS

And finally, I will set up my local clients by DHCP to use **local resolution**, and I will configure my Router to resolve using **CloudFlare family**, even if it is a bit slower I get DNS Malware block for free.

This is an example of the 2 lines the script captures from tcpdump and then check the timestamp.

```shell
Query packet: 21:00:31.896093 IP 172.21.206.185.58484 > 172.21.192.1.53: 48526+ A? adriangiacometti.net. (38)
Reply packet: 21:00:31.945733 IP 172.21.192.1.53 > 172.21.206.185.58484: 48526- 2/0/0 A 45.84.207.132 (54)
```

The script can be modfied to capture the 2 lines you prefeer, and thus it can be used to measure any latency with the smallest resolution possible and pure from the network. As an example you could capture SYN and FIN flags to check the total time of connection.

Thanks for reading ;)
