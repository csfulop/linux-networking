# Intro
So, my life in a short, stucked on the Airport in Zurich, and find my way out, so I finally found a shop open. Nice, now I have something to drink, but security won't open before 5:00 AM, so I have a chance to work on this finally... 6h to use for something, sleeping is overrated anyway.

In a nutshell for a long time I planned to make a summary about Linux Network tools and how to use them. It shall give you some basic overview of the possibilities, and teach you how to use the CLI to configure a Linux box.

## Basics

In a moder linux distro `ip` command is your way to go. For sentimental and historical reasons I will sometimes show how to do things in the old way or with the old tools.

Lets get through some basics, shall we?

To list your network interfaces you can do this:

```
ip link
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: wlp59s0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN mode DORMANT group default qlen 1000
    link/ether ae:9c:4e:b6:e0:72 brd ff:ff:ff:ff:ff:ff
3: virbr0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN mode DEFAULT group default qlen 1000
    link/ether 52:54:00:8f:55:01 brd ff:ff:ff:ff:ff:ff
4: virbr0-nic: <BROADCAST,MULTICAST> mtu 1500 qdisc fq_codel master virbr0 state DOWN mode DEFAULT group default qlen 1000
    link/ether 52:54:00:8f:55:01 brd ff:ff:ff:ff:ff:ff
```

now as you can see I have some, a loopback device, a wireless, some virtual bridge and a virtual nic. Those are created by libvirtd in my case, but never mind them later you will understand better what are those.

one important feature of the `ip` command is that you can shorten things, instead of link you can say "l" or "li" or even "lin". Try it!

Lets see the ip addresses assigned to these interfaces:

```
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: wlp59s0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default qlen 1000
    link/ether ae:9c:4e:b6:e0:72 brd ff:ff:ff:ff:ff:ff
3: virbr0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default qlen 1000
    link/ether 52:54:00:8f:55:01 brd ff:ff:ff:ff:ff:ff
    inet 192.168.122.1/24 brd 192.168.122.255 scope global virbr0
       valid_lft forever preferred_lft forever
4: virbr0-nic: <BROADCAST,MULTICAST> mtu 1500 qdisc fq_codel master virbr0 state DOWN group default qlen 1000
    link/ether 52:54:00:8f:55:01 brd ff:ff:ff:ff:ff:ff
```

Perfect, two more things to mention here, one by default you get IPv4 (at the time being), if you want to see IPv6 information you can use the `-6` switch:

```
ip -6 address
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 state UNKNOWN qlen 1000
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
```

`ip` comes with a very nice colorful output if your terminal supports it, some distros this is the default behavior others requires you to use the `-c` option

![alt](.static/ip_color.png)

Nice!

Back to the basics, lets say you want to shut down one of the interfaces, on L2, the always easy target loopback device is there for us.

To confirm your loopback device works:

```
ping 127.0.0.1
PING 127.0.0.1 (127.0.0.1) 56(84) bytes of data.
64 bytes from 127.0.0.1: icmp_seq=1 ttl=64 time=0.084 ms
64 bytes from 127.0.0.1: icmp_seq=2 ttl=64 time=0.064 ms
64 bytes from 127.0.0.1: icmp_seq=3 ttl=64 time=0.036 ms
64 bytes from 127.0.0.1: icmp_seq=4 ttl=64 time=0.072 ms
64 bytes from 127.0.0.1: icmp_seq=5 ttl=64 time=0.069 ms
64 bytes from 127.0.0.1: icmp_seq=6 ttl=64 time=0.065 ms
64 bytes from 127.0.0.1: icmp_seq=7 ttl=64 time=0.080 ms
64 bytes from 127.0.0.1: icmp_seq=8 ttl=64 time=0.083 ms
64 bytes from 127.0.0.1: icmp_seq=9 ttl=64 time=0.073 ms
64 bytes from 127.0.0.1: icmp_seq=10 ttl=64 time=0.070 ms
^C
--- 127.0.0.1 ping statistics ---
10 packets transmitted, 10 received, 0% packet loss, time 224ms
rtt min/avg/max/mdev = 0.036/0.069/0.084/0.015 ms
```

As expected right? Now bring it down:

```
sudo ip link set lo down
```

(You need to be someone to mess with network interfaces :) ), so lets see the status now:

```
ping 127.0.0.1 -w 1
PING 127.0.0.1 (127.0.0.1) 56(84) bytes of data.

--- 127.0.0.1 ping statistics ---
1 packets transmitted, 0 received, 100% packet loss, time 0ms
```

(Little side note, hopefully you have some idea what PING is, if not it is really time to do a search, in a nutshell it uses ICMP protocol to check connection between two network points. (ICMP is on L3) -> and if you don't know the network layers, really good time to search OSI Layers too.)

As expected, we can't don't get answer anymore, as our interface is down, just to confirm:

```
ip l l lo
1: lo: <LOOPBACK> mtu 65536 qdisc noqueue state DOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
```

Note: see how you can filter for a known interface ?

Okay, so now bring it back to life, some services may need it on your system :)

```
sudo ip l s lo up
```

Next things next on the basics, lets say you want to assign a static IP to an interface, not that rare as you would think. 

Let's add the magical `10.0.0.1/24` address to the lo interface:

```
sudo ip address add 10.0.0.1/24 dev lo
```

Again some homework, if you never heard about private ip ranges, do a favor to yourself and search it now. Plus if you have no idea what is a network mask, do search that too.

List the addresses associated to the loopback interface:

```
ip a l lo
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet 10.0.0.1/24 scope global lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
```

Now this is interesting, in the old times we used a tool called ifconfig, which did not allow you to assign multiple addresses to the same interface, lets see what does it think now:

```
ifconfig lo
lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 28  bytes 2120 (2.0 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 28  bytes 2120 (2.0 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

Uho, no sign of our new address, lets try to ping it:

```
ping 10.0.0.1
PING 10.0.0.1 (10.0.0.1) 56(84) bytes of data.
64 bytes from 10.0.0.1: icmp_seq=1 ttl=64 time=0.095 ms
64 bytes from 10.0.0.1: icmp_seq=2 ttl=64 time=0.071 ms
^C
--- 10.0.0.1 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 45ms
rtt min/avg/max/mdev = 0.071/0.083/0.095/0.012 ms
```

So it is there, guess now we have a good reason to not use ifconfig anymore.
(note: it is possible with ifconfig to assign multiple IP addresses to one physical device, but it is creating a new "link" device - so say. Never mind, use `ip` please!)

So we added a new IP address to one of our interfaces, what happened to our routing table? Lets print it out:

```
ip route list
192.168.122.0/24 dev virbr0 proto kernel scope link src 192.168.122.1 linkdown 
```

Interesting, this interface is not even related, and looks like we don't have a route for the lo device... what happens here? For now it is enough to understand that linux has more than one routing table, to list the entries related to lo device use table id 0:

```
ip route list table 0
192.168.122.0/24 dev virbr0 proto kernel scope link src 192.168.122.1 linkdown 
broadcast 10.0.0.0 dev lo table local proto kernel scope link src 10.0.0.1 
local 10.0.0.0/24 dev lo table local proto kernel scope host src 10.0.0.1 
local 10.0.0.1 dev lo table local proto kernel scope host src 10.0.0.1 
broadcast 10.0.0.255 dev lo table local proto kernel scope link src 10.0.0.1 
broadcast 127.0.0.0 dev lo table local proto kernel scope link src 127.0.0.1 
local 127.0.0.0/8 dev lo table local proto kernel scope host src 127.0.0.1 
local 127.0.0.1 dev lo table local proto kernel scope host src 127.0.0.1 
broadcast 127.255.255.255 dev lo table local proto kernel scope link src 127.0.0.1 
broadcast 192.168.122.0 dev virbr0 table local proto kernel scope link src 192.168.122.1 linkdown 
local 192.168.122.1 dev virbr0 table local proto kernel scope host src 192.168.122.1 
broadcast 192.168.122.255 dev virbr0 table local proto kernel scope link src 192.168.122.1 linkdown 
local ::1 dev lo table local proto kernel metric 0 pref medium
```

WOW! lots of entries, and my machine is not even connected now to any network!

We will talk about this and rouring tables more in the future of this document :)

Okay, so we can set links up and down, we can add static ips, how to remove them?

```
sudo ip a delete 10.0.0.1/24 dev lo
```

Simple. I like simple.

How about dynamic IPs? Well I will add it here once I found a network to connect to:

**BIG FAT TODO**

## IP namespaces - Not so basics anymore
In linux kernel, you can have some things called ip namespaces, the magic is that a namespace separates your whole network stack from an other "copy" of your network stack. Sounds crazy if you hear about it first, and I don't even pretend to understand the complexity below, but how I imagine it, you have your routing tables, ip addresses interfaces, all of these are refered by some structures in the kernel, and as the programers did a great job, now you can have more than one of this structure. 

More about the why? - Because, of containers, because of you want to test a complex network on a single machine without any hardware ... etc. Many reasons in fact. 

But lets jump into it, you will understand it better once you see it.

Lets create a new namespace:

```
sudo ip netns add test
```

Okay, so nothing happened? Lets list the namespaces:

```
ip netns
test
```

As you can see it won't list the "default" namespace. Now running commands with the namespace as a context is a bit tricky:

```
sudo ip netns exec test ip l
1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
```

Uho, looks like a recursion? No worries it gets worse. As you can see in our new namespace there is only one interface, the loopback and even it is down. Not really an issue for now.

So how this command looks like:
ip netns exec test <COMMAND to RUN "within" the namespace>

To make life easier I like to "enter" into namespaces by running a new shell with the namespace / or in the namespace:

```
sudo ip netns exec test bash
```

This will "Drop" you into a new shell, try now any IP command:

```
ip l
1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
```

**BIG FAT WARNING**: as most of the systems you have to be root to run anything in a net namespace - your shell will be with root privilages, and despite the network is spearated, the filesystems, processes are not! Be careful in this shell.

Now so you are in a namespace lovely, can't do much, as you have no network connection from this namespace yet. And the bummer one device can be only in one namespace at the time. 

Maybe before we proceed, lets create a dummy interface - so we can play with that instead of the loopback interface:

(get out from the netns shell, or open a new window, point: be in the default namespace)

```
sudo ip l add type dummy 
```

Easy huh, check the list of the interfaces, now you have a dummy0 interface:

```
ip l
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: wlp59s0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN mode DORMANT group default qlen 1000
    link/ether f2:2b:71:24:e5:ca brd ff:ff:ff:ff:ff:ff
3: virbr0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN mode DEFAULT group default qlen 1000
    link/ether 52:54:00:8f:55:01 brd ff:ff:ff:ff:ff:ff
4: virbr0-nic: <BROADCAST,MULTICAST> mtu 1500 qdisc fq_codel master virbr0 state DOWN mode DEFAULT group default qlen 1000
    link/ether 52:54:00:8f:55:01 brd ff:ff:ff:ff:ff:ff
5: dummy0: <BROADCAST,NOARP> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether ea:d1:76:17:3d:5e brd ff:ff:ff:ff:ff:ff
```

As the name kind of predicts it, dummy is not really up for anything, does not connceted to any hardware and well does not do much, but we can abuse it with all the kind of commands we just learned, you can assign IP address to it and so on. Most importantly for our main topic in this section, you can move it into an other namespace:

```
sudo ip l s dummy0 netns test
```

if you list now the interfaces in the default namespace:

```
ip l
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: wlp59s0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN mode DORMANT group default qlen 1000
    link/ether b2:67:19:5c:c2:4e brd ff:ff:ff:ff:ff:ff
3: virbr0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN mode DEFAULT group default qlen 1000
    link/ether 52:54:00:8f:55:01 brd ff:ff:ff:ff:ff:ff
4: virbr0-nic: <BROADCAST,MULTICAST> mtu 1500 qdisc fq_codel master virbr0 state DOWN mode DEFAULT group default qlen 1000
    link/ether 52:54:00:8f:55:01 brd ff:ff:ff:ff:ff:ff
```

Dummy went missing, lets try to list the interfaces in our namespace:

```
sudo ip netns exec test ip l
1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
5: dummy0: <BROADCAST,NOARP> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether ea:d1:76:17:3d:5e brd ff:ff:ff:ff:ff:ff
```

Hey there we have dummy! A M A Z I N G!
Now some notes again, in order to be able to toss a device into a namespace, its driver has to support this. Happened before that some wifi chipset kernel drivers / modules were not supporting this. so if you constantly fail to move something into a namespace check the kernelogs. 

As you may figured out you can delete namespaces too:

```
sudo ip netns delete test
```

if you check now the default namespace:

```
ip l
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: wlp59s0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN mode DORMANT group default qlen 1000
    link/ether b2:67:19:5c:c2:4e brd ff:ff:ff:ff:ff:ff
3: virbr0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN mode DEFAULT group default qlen 1000
    link/ether 52:54:00:8f:55:01 brd ff:ff:ff:ff:ff:ff
4: virbr0-nic: <BROADCAST,MULTICAST> mtu 1500 qdisc fq_codel master virbr0 state DOWN mode DEFAULT group default qlen 1000
    link/ether 52:54:00:8f:55:01 brd ff:ff:ff:ff:ff:ff
```

dummy is not here, and we don't have the namespace anymore. Now so this is interesting right, we get rid of dummy, by deleting the namespace he lived in. I'm not 100% convinced that dummy wend down as he supposed to be, but at least we don't have a way to reach him anymore, I advise to delete the interfaces before you delete the namespaces, but it is just because I don't know for sure that those are freed properly just because we deleted their home. (Anyone?)

Okay, so far we had to separated namespaces, not much of a fun, we want to build some real life like stuff right? 

Lets learn about VETH pairs.

A VETH pair, as the name kind of reveals already, means two interfaces, and they are magically connected in the subspace. How it works? You drop anything into one of them, and it will fall out on the other end. Virtual Ethernet? (to confirm) It is like a network cable with two ends.

Lets create one:

```
sudo ip l a type veth
```

and list the interfaces:

```
ip l
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: wlp59s0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN mode DORMANT group default qlen 1000
    link/ether 3e:0f:bd:e3:fc:07 brd ff:ff:ff:ff:ff:ff
3: virbr0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN mode DEFAULT group default qlen 1000
    link/ether 52:54:00:8f:55:01 brd ff:ff:ff:ff:ff:ff
4: virbr0-nic: <BROADCAST,MULTICAST> mtu 1500 qdisc fq_codel master virbr0 state DOWN mode DEFAULT group default qlen 1000
    link/ether 52:54:00:8f:55:01 brd ff:ff:ff:ff:ff:ff
6: veth0@veth1: <BROADCAST,MULTICAST,M-DOWN> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether 1e:0f:1e:a2:c5:f8 brd ff:ff:ff:ff:ff:ff
7: veth1@veth0: <BROADCAST,MULTICAST,M-DOWN> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether b6:f3:81:52:a1:2d brd ff:ff:ff:ff:ff:ff
```

Very good, as you can see the name kinda tries to help, shows which is the "other" iterface connected to the given one. Don't get confused those called `veth1 and veth0` after the `@` you just have a hint which belongs to which.

Any idea how this can be useful? 

## Exercise: Connect namespaces with VETH pairs
Before I show, try to do this now:
 - Create a VETH pair
 - Create a netns
 - Move one side of the VETH pair into the netns
 - Set a static IP on VETH0 - lets say 10.0.0.1/24
 - Set a static IP on VETH1 - lets say 10.0.0.2/24
 - Set the link up on VETHx
 - Do ping the other end

 In other words, lets connect two namespaces with a VETH pair, which you can think about as two Routers being connected with a cable, do you start to see the possibilities here?

 <TODO>