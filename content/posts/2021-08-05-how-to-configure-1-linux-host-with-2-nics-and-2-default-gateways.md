---
title: "How to configure 1 Linux host with 2 NICs and 2 Default Gateways"
date: 2021-08-05
categories: 
  - "uncategorized-en"
tags: 
  - "iproute2"
  - "linux"
---

Historically, it was not possible to have 2 default gateways in a Linux host. 

You can SEND traffic to both NICs, but the reply will always come from the NIC which has the default gateway.

This will generate asymmetric traffic and it will be denied in the network or by the originating host. Asymmetric traffic means that the packets will go back using a different path. In this case by ping to NIC1 the return will be sent by NIC2.

But with [iproute2](https://en.wikipedia.org/wiki/Iproute2), it can be done with 2 commands.

Let's do this!

This is the basic scenario:

![](/images/basic.jpg)

Initial setup:

- Network: if you don't have a firewall like in the diagram, then be sure to enable "unicast reverse path forwarding" at the router. This will block any asymmetric reply (in firewalls is the default behavior).
- Linux host: I'm using Ubuntu. Setup the NICs by editing the file /etc/network/interfaces, in this case:

```textile
iface eth0 inet static
	address 10.10.0.10
	netmask 255.255.255.0
	gateway 10.10.0.1
	up echo nameserver 8.8.8.8 > /etc/resolv.conf
auto eth0
iface eth1 inet static
	address 10.20.0.10
	netmask 255.255.255.0
auto eth1
```

**Setup iproute2 in the Linux host**

In order to enable the use of 2 default gateways, we need to set up a new **route table** and a **route rules**. This will make the traffic entering NIC2 go back using the same NIC2.

```textile
sudo ip route add default via 10.20.0.1 dev eth1 table 1001
sudo ip rule add from 10.20.0.0/24 table 1001
```

Done, now I can externally access both NICs independently. 

Check by doing some pings from a remote network and verify with **tcpdump -i eth0** and/or **eth1**.

Should see that each packet flows by the NIC it should.

Finally, to make the changes permanent to add these lines to **/etc/network/interfaces**.

```textile
iface eth1 inet static
    address 10.20.0.10
    netmask 255.255.255.0
    post-up ip route add default via 10.20.0.1 dev eth1 table 1001
    post-up ip rule add from 10.20.0.0/24 table 1001
```

You can check the resulting config with these commands:

```textile
/# sudo ip route show table main
default via 10.10.0.1 dev eth0 
10.10.0.0/24 dev eth0 proto kernel scope link src 10.10.0.10 
10.20.0.0/24 dev eth1 proto kernel scope link src 10.20.0.10 
/# sudo ip route show table 1001
default via 10.20.0.1 dev eth1 
/# sudo ip rule show
0:	from all lookup local
32765:	from 10.20.0.0/24 lookup 1001
32766:	from all lookup main
32767:	from all lookup default
```

It was way more easy than expected right?

I've seen other configurations using **iptables** to select the routing tables by using traffic marking. But for something this simple it doesn't make any sense to get into **iptables**.

In the next post, I will extend the scenario, to [support 2 network isolated Docker containers (front-end and back-end) in 1 Linux host, with 2 NICs and 2 default gateway](https://adriangiacometti.net/index.php/2021/08/06/how-to-set-up-2-network-insolated-docker-containers/).

Cheers!
