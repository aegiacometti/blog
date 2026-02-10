---
title: "How to set up 2 network isolated Docker containers (front and back-end)"
date: 2021-08-06
categories: 
  - "uncategorized-en"
tags: 
  - "docker"
  - "iproute2"
  - "linux"
---

Making this work might sound crazy, but if Docker has to be comparable in functionalities already provided by other types of virtualization, then this has to work right?

Ok, this is the scenario, is a classic one:

- I need to deploy in one Docker host, 2 isolated containers, let's say front-end and back-end.
- The Linux host will have 2 NICs, each one with a different subnet.
- Each container will use a different NIC and external subnet.
- For Docker this means 2 docker networks (fe-bridge and be-bridge), which translates to 2 Linux bridges.
- Traffic between containers is not allowed internally, it has to go to the external network where there is a Firewall.
- External management traffic will go directly to the containers but each one through its respective physical NIC.
- And of course, everything is connected to Internet.

![](/images/docker.jpg)

**Conceptual components**

Keep in mind that Docker is the virtualization engine and is using the Linux Kernel and tools.

To work with this puzzle, I will just go back to the basics and follow the same path as in a traditional physical environment:

1. **Layer 2 - at MAC address level with a switch:** here I will be using "Linux Bridge" (Docker automates the use of Linux Bridge). Nothing to do here.
2. **Layer 3 - IP level with a router/firewall:** here "[iproute2](https://en.wikipedia.org/wiki/Iproute2)" allows me to solve the asymmetric problem and use both NICs. HERE I will do some work!
3. **Layer 4 - At TCP/UDP port level again a Firewall/Router:** Docker uses Linux "[iptables](https://en.wikipedia.org/wiki/Iptables)" to NAT the Docker networks/hosts to be reachable by external networks. Nothing to do here.

**Initial setup:**

- **Network:** if you don't have a firewall like in the diagram, then be sure to enable "unicast reverse path forwarding" at the router. This will block any asymmetric reply (in firewalls is the default behavior).
- **Linux host:** I'm using Ubuntu. Setup the NICs by editing the file **/etc/network/interfaces**, in this case:

```textile
iface enp0s3 inet static
	address 10.10.0.10
	netmask 255.255.255.0
	gateway 10.10.0.1
	up echo nameserver 8.8.8.8 > /etc/resolv.conf
iface enp0s8 inet static
	address 10.20.0.10
	netmask 255.255.255.0
auto eth0
auto eth1
```

- Docker: I'm using the convenience script to install Docker, which is great for development environments since it has all in one. (source: [https://docs.docker.com/engine/install/ubuntu/](https://docs.docker.com/engine/install/ubuntu/))

```textile
 curl -fsSL https://get.docker.com -o get-docker.sh
 sudo sh get-docker.sh
```

**Docker setup**

Now I need to configure 2 "Docker networks", **br-fe** and **br-be**. This will be automatically translated to Linux Bridges.

```textile
sudo docker network create --driver bridge --subnet 10.10.10.0/24 br-fe
sudo docker network create --driver bridge --subnet 10.20.10.0/24 br-be
```

Check the Docker networks created with "**docker network ls**" and their configuration with "**docker network inspect \[network\_id\]**".

You should be able to see those configurations reflected in the pure Linux host with "**bridge link show**", "**ip a**", "**bridge fdb show br \[bridge\_name\]**".

Remember, Docker use the Linux host kernel features, so it will create the needed Linux Bridges and the IPTables rules.

**New containers setup**

Now we will create 2 containers, and attach one to each bridge.

I will use a basic container with only SSH server installed just for the tests from [https://github.com/linuxserver/docker-openssh-server](https://github.com/linuxserver/docker-openssh-server).

If you already have a Docker image and/or containers skip this part. But pay attention to docker run your container with the same parameters for the bridge assignment (--nerwork) and ports (-p)

- Container front-end: binded to **enp0s3** with IP 10.10.0.10

```textile
docker run -d 
  --name=front-end 
  --network=br-fe 
  -p 10.10.0.10:2222:2222 
  ghcr.io/linuxserver/openssh-server
```

- Container back-end: binded to **unp0s8** with IP 10.20.20.10

```textile
docker run -d 
  --name=back-end 
  --network=br-be 
  -p 10.20.0.10:2222:2222 
  ghcr.io/linuxserver/openssh-server
```

**Setup iproute2 in the Linux host**

Under this scenario, we need to do several changes to the route tables and their selection rules. (_pay attention to the bridge-id in the examples_).

Just for reference, I will show my start-up configurations.

This is default and I didn't change anything, so it should be similar to the ones in your PC:

```textile
/ # ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:d7:b3:20 brd ff:ff:ff:ff:ff:ff
    inet 10.10.0.10/24 brd 10.10.0.255 scope global noprefixroute enp0s3
       valid_lft forever preferred_lft forever
    inet6 fe80::c188:cce3:83d:7f30/64 scope link noprefixroute 
       valid_lft forever preferred_lft forever
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:75:86:72 brd ff:ff:ff:ff:ff:ff
    inet 10.20.0.10/24 brd 10.20.0.255 scope global noprefixroute enp0s8
       valid_lft forever preferred_lft forever
    inet6 fe80::e8f6:8986:3f2b:5a5f/64 scope link noprefixroute 
       valid_lft forever preferred_lft forever
7: br-08e82a583c94: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether 02:42:fc:67:5f:bb brd ff:ff:ff:ff:ff:ff
    inet 10.20.20.1/24 brd 10.20.20.255 scope global br-08e82a583c94
       valid_lft forever preferred_lft forever
    inet6 fe80::42:fcff:fe67:5fbb/64 scope link 
       valid_lft forever preferred_lft forever
8: br-d26a98a1f90b: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether 02:42:e8:c4:bf:bb brd ff:ff:ff:ff:ff:ff
    inet 10.10.10.1/24 brd 10.10.10.255 scope global br-d26a98a1f90b
       valid_lft forever preferred_lft forever
    inet6 fe80::42:e8ff:fec4:bfbb/64 scope link 
       valid_lft forever preferred_lft forever
9: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default 
    link/ether 02:42:dd:c3:33:68 brd ff:ff:ff:ff:ff:ff
    inet 10.250.0.1/16 brd 10.250.255.255 scope global docker0
       valid_lft forever preferred_lft forever
17: veth1653b00@if16: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master br-d26a98a1f90b state UP group default 
    link/ether ea:6b:95:4e:38:8b brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet6 fe80::e86b:95ff:fe4e:388b/64 scope link 
       valid_lft forever preferred_lft forever
19: vethb5ce62d@if18: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master br-08e82a583c94 state UP group default 
    link/ether 6a:c5:03:7f:43:f5 brd ff:ff:ff:ff:ff:ff link-netnsid 1
    inet6 fe80::68c5:3ff:fe7f:43f5/64 scope link 
       valid_lft forever preferred_lft forever
/ # ip rule show
0:	from all lookup local 
32766:	from all lookup main 
32767:	from all lookup default 
/ # ip route show table local
broadcast 10.10.0.0 dev enp0s3 proto kernel scope link src 10.10.0.10 
local 10.10.0.10 dev enp0s3 proto kernel scope host src 10.10.0.10 
broadcast 10.10.0.255 dev enp0s3 proto kernel scope link src 10.10.0.10 
broadcast 10.10.10.0 dev br-d26a98a1f90b proto kernel scope link src 10.10.10.1 
local 10.10.10.1 dev br-d26a98a1f90b proto kernel scope host src 10.10.10.1 
broadcast 10.10.10.255 dev br-d26a98a1f90b proto kernel scope link src 10.10.10.1 
broadcast 10.20.0.0 dev enp0s8 proto kernel scope link src 10.20.0.10 
local 10.20.0.10 dev enp0s8 proto kernel scope host src 10.20.0.10 
broadcast 10.20.0.255 dev enp0s8 proto kernel scope link src 10.20.0.10 
broadcast 10.20.20.0 dev br-08e82a583c94 proto kernel scope link src 10.20.20.1 
local 10.20.20.1 dev br-08e82a583c94 proto kernel scope host src 10.20.20.1 
broadcast 10.20.20.255 dev br-08e82a583c94 proto kernel scope link src 10.20.20.1 
broadcast 10.250.0.0 dev docker0 proto kernel scope link src 10.250.0.1 linkdown 
local 10.250.0.1 dev docker0 proto kernel scope host src 10.250.0.1 
broadcast 10.250.255.255 dev docker0 proto kernel scope link src 10.250.0.1 linkdown 
broadcast 127.0.0.0 dev lo proto kernel scope link src 127.0.0.1 
local 127.0.0.0/8 dev lo proto kernel scope host src 127.0.0.1 
local 127.0.0.1 dev lo proto kernel scope host src 127.0.0.1 
broadcast 127.255.255.255 dev lo proto kernel scope link src 127.0.0.1 
broadcast 192.168.122.0 dev virbr0 proto kernel scope link src 192.168.122.1 linkdown 
local 192.168.122.1 dev virbr0 proto kernel scope host src 192.168.122.1 
broadcast 192.168.122.255 dev virbr0 proto kernel scope link src 192.168.122.1 linkdown 
```

The **local** routing table is maybe the most important since it is used for broadcast and local IPs. But now since when need to change the routing decisions, we need to split its content into the new routing tables.

So, we will create 2 new route tables one per NIC (NIC1=1001, NIC2=1002). You can use the number or you can refer to the tables "by names" editing the file /etc/iproute2/rt\_tables.

```textile
echo "1001 rt_nic1" >> /etc/iproute2/rt_tables
echo "1002 rt_nic2" >> /etc/iproute2/rt_tables
```

Now we add the new route tables:

```textile
# Route table NIC1=1001
ip route add default via 10.10.0.1 dev enp0s3 proto kernel table rt_nic1
ip route add broadcast 10.10.0.0 dev enp0s3 scope link src 10.10.0.10 proto kernel table rt_nic1
ip route add 10.10.0.0/24 dev enp0s3 scope link src 10.10.0.10 proto kernel table rt_nic1
ip route add local 10.10.0.10 dev enp0s3 scope host src 10.10.0.10 proto kernel table rt_nic1
ip route add broadcast 10.10.0.255 dev enp0s3 scope link src 10.10.0.10 proto kernel table rt_nic1
ip route add broadcast 10.10.10.0 dev br-d26a98a1f90b scope link src 10.10.10.1 proto kernel table rt_nic1
ip route add 10.10.10.0/24 dev br-d26a98a1f90b scope link src 10.10.10.1 proto kernel table rt_nic1
ip route add local 10.10.10.1 dev br-d26a98a1f90b scope host src 10.10.10.1 proto kernel table rt_nic1
ip route add broadcast 10.10.10.255 dev br-d26a98a1f90b scope link src 10.10.10.1 proto kernel table rt_nic1
# Route table NIC2=1002
ip route add default via 10.20.0.1 dev enp0s8 proto kernel table rt_nic2
ip route add broadcast 10.20.0.0 dev enp0s8 scope link src 10.20.0.10 proto kernel table rt_nic2
ip route add 10.20.0.0/24 dev enp0s8 scope link src 10.20.0.10 proto kernel table rt_nic2
ip route add local 10.20.0.10 dev enp0s8 scope host src 10.20.0.10 proto kernel table rt_nic2
ip route add broadcast 10.20.0.255 dev enp0s8 scope link src 10.20.0.10 proto kernel table rt_nic2
ip route add broadcast 10.20.20.0 dev br-08e82a583c94 scope link src 10.20.20.1 proto kernel table rt_nic2
ip route add 10.20.20.0/24 dev br-08e82a583c94 scope link src 10.20.20.1 proto kernel table rt_nic2
ip route add local 10.20.20.1 dev br-08e82a583c94 scope host src 10.20.20.1 proto kernel table rt_nic2
ip route add broadcast 10.20.20.255 dev br-08e82a583c94 scope link src 10.20.20.1 proto kernel table rt_nic2
```

And finally, we will change the routing decisions. It would be ideal if you do this by a console or direct connection (just in case something goes wrong).

So first take a text backup and then change the rules.

Pay special attention to rules with priority 10 and 20, the IP subnet and the interface are mixed with other interfaces, this is how we will force the traffic to follow the path we want.

```textile
ip rule show >> ip_rule_bkp
ip rule add priority 10 iif enp0s3 table rt_nic1
ip rule add priority 11 iif enp0s8 table rt_nic2
ip rule add priority 20 iif br-d26a98a1f90b table rt_nic1
ip rule add priority 21 iif br-08e82a583c94 table rt_nic2
ip rule add priority 30 from 10.10.0.0/24 table rt_nic1
ip rule add priority 31 from 10.20.0.0/24 table rt_nic2
ip rule del from all table local
ip rule add from all table local priority 100
```

In the last 2 rules, we are changing the default behavior of using the local table, we need that to AVOID local routing inside the host, so, what we do is adding it with a higher number (lower priority).

Up to here both containers:

- have access to the external network through their respective physical NICs
- can be accessed remotely on their respective physical NICs
- can for communicate to each other using the EXTERNAL network (for security reasons the traffic has to go to an external firewall) 

Check by doing some pings from a remote network and verify with **tcpdump -i enp0s3** and/or **enp0s8** and even in the bridge interfaces or the container interfaces.

Should see that each packet flows by the NIC it should.

Finally, to make the changes add the lines from the last 2 snippets (route tables and rules) to an executable file like a script. make ip executable And add that file to **/etc/network/if-up.d/** directory.

```textile
# Example
echo linex >> set-routing.sh
chmod +x set-routing.sh
mv set-routing.sh /etc/network/if-up.d/ 
```

Now, think about when you are talking in "Cloud terminologies", they say "in the cloud you don't have routers anymore, you have route tables"... well .. voila... these are the route tables they talk about :)

I tried Linux VRF (which automates using **ip rules** in the back = fewer config lines), looked good but didn't work, at least in my Lab.

I wish this is useful for you since I couldn't find any working example on the Internet.

If you need help please let me know.

Cheers.
