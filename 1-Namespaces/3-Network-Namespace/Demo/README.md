# Network Namespace Demo


### STEP 1 — Create a Network Namespace 

```
[root@RHEL ~]# ip netns add ns1

[root@RHEL ~]# ip netns list
ns1                                 # Namespace ns1 created

[root@RHEL ~]# 

```
Does ns1 have internet? - NO

Does ns1 have an IP? - NO

Does ns1 have interfaces? - NO



** Let’s check ** 

```

[root@RHEL ~]# ip netns exec ns1 ip addr #  We are runnin the command inside the namespace. 


1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
[root@RHEL ~]# 

```

You can see that only loopback (lo) is present. Why Only Loopback?

When Linux creates a network namespace, it creates a brand new empty network stack. No ethernet, No bridge, No route, Nothing.

Just loopback.


As we know this namespace is isolated, we first need to create a connectivity from namespace to Host machine.

For this we use something called as veth pair (virtual ethernet cable)

What is a veth Pair?

[veth-A] <=======> [veth-B]

Whatever enters one side comes out the other side. It is literally a virtual ethernet cable.


STEP 2 — Create the Cable

To create a veth pair cable, run the following command:

```
# Command to create a veth pair

[root@RHEL ~]# ip link add veth-host type veth peer name veth-ns


# verify the veth pair names: 1. veth-host and 2. veth-ns


[root@RHEL ~]# ip link 
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: enp1s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP mode DEFAULT group default qlen 1000
    link/ether 52:54:00:3e:65:8e brd ff:ff:ff:ff:ff:ff
    altname enx5254003e658e
3: veth-ns@veth-host: <BROADCAST,MULTICAST,M-DOWN> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether da:49:6e:46:b9:13 brd ff:ff:ff:ff:ff:ff
4: veth-host@veth-ns: <BROADCAST,MULTICAST,M-DOWN> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether 8e:70:89:37:16:21 brd ff:ff:ff:ff:ff:ff
[root@RHEL ~]# 


```

But now both ends of the cable are still in hostnamespace.

We need to connect one end to host and other end to the namespace we created.

We want this topology:

```
Host Namespace        ns1 Namespace
--------------        --------------
veth-host  <=======>  veth-ns

```

So each namespace owns one end of the cable.


STEP 3 — Move One Side Into ns1

```
# we are moving the veth-ns end to the namespace ns1

[root@RHEL ~]# ip link set veth-ns netns ns1

# Cheking the interfaces, we can see veth-ns is not not visible on host, because its moved to namespace ns1.

[root@RHEL ~]# ip link
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: enp1s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP mode DEFAULT group default qlen 1000
    link/ether 52:54:00:3e:65:8e brd ff:ff:ff:ff:ff:ff
    altname enx5254003e658e
4: veth-host@if3: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether 8e:70:89:37:16:21 brd ff:ff:ff:ff:ff:ff link-netns ns1

```

Lets verify whether the veth-ns is inside the ns1 namespace:

```
[root@RHEL ~]# ip netns exec ns1 ip link

# You can see the "veth-ns@if4" is now inside the ns1 namespace.

1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
3: veth-ns@if4: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether da:49:6e:46:b9:13 brd ff:ff:ff:ff:ff:ff link-netnsid 0

```

Now we truly have:

```
Host  <=======>  ns1

```

Now whats next?

We need to assign the IP address to these interafaces and make it UP.


STEP 4 — Assign IP Addresses


Let’s create a small network: 10.0.0.0/24

We’ll use:

Host side → 10.0.0.1

ns1 side → 10.0.0.2


On Host: ip addr add 10.0.0.1/24 dev veth-host

```
# Assign IP

[root@RHEL ~]# ip addr add 10.0.0.1/24 dev veth-host

# Make the veth-host interface UP

[root@RHEL ~]# ip link set veth-host up

# Check the status

[root@RHEL ~]# ip addr show veth-host
4: veth-host@if3: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state LOWERLAYERDOWN group default qlen 1000
    link/ether 8e:70:89:37:16:21 brd ff:ff:ff:ff:ff:ff link-netns ns1
    inet 10.0.0.1/24 scope global veth-host
       valid_lft forever preferred_lft forever
[root@RHEL ~]# 

```

Inside ns1: ip netns exec ns1 ip addr add 10.0.0.2/24 dev veth-ns

```
# Assign IP

[root@RHEL ~]# ip netns exec ns1 ip addr add 10.0.0.2/24 dev veth-ns

# Make the veth-ns1 interface and lo UP

[root@RHEL ~]# ip netns exec ns1 ip link set veth-ns up
ip netns exec ns1 ip link set lo up

# # Check the status

[root@RHEL ~]# ip netns exec ns1 ip addr show veth-ns

3: veth-ns@if4: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether da:49:6e:46:b9:13 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 10.0.0.2/24 scope global veth-ns
       valid_lft forever preferred_lft forever
    inet6 fe80::d849:6eff:fe46:b913/64 scope link proto kernel_ll 
       valid_lft forever preferred_lft forever
```

We now have:

```
Host(10.0.0.1)  <=======>  ns1 (10.0.0.2)
```

Lets try pinging both interface to verify the connectivity:

Pinging from ns1 to host

```
[root@RHEL ~]# ip netns exec ns1 ping -c 2 10.0.0.1
PING 10.0.0.1 (10.0.0.1) 56(84) bytes of data.
64 bytes from 10.0.0.1: icmp_seq=1 ttl=64 time=0.079 ms
64 bytes from 10.0.0.1: icmp_seq=2 ttl=64 time=0.053 ms

--- 10.0.0.1 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1016ms
rtt min/avg/max/mdev = 0.053/0.066/0.079/0.013 ms
```

Pinging from host to ns1

```
[root@RHEL ~]# ping -c 2 10.0.0.2
PING 10.0.0.2 (10.0.0.2) 56(84) bytes of data.
64 bytes from 10.0.0.2: icmp_seq=1 ttl=64 time=0.151 ms
64 bytes from 10.0.0.2: icmp_seq=2 ttl=64 time=0.047 ms

--- 10.0.0.2 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1025ms
rtt min/avg/max/mdev = 0.047/0.099/0.151/0.052 ms


```
Bit how did the ping worked?

Because of Kernal route table!. Lets analyse .

```
# Chek the route table of the ns1 namespace

[root@RHEL ~]# ip netns exec ns1 ip route

# You can see there is a route present, This means if destination is in 10.0.0.x, send directly via veth-ns. No Gateway Needed here because both eth pair sides are in same network (10.0.0.2, and  10.0.0.1)

10.0.0.0/24 dev veth-ns proto kernel scope link src 10.0.0.2 


```


Great!. No out namespace can communicate with host machine. This is how Docker container talk to our host machine. But there are more to add... Lets continue.!


Now, Can the namespace ns1 connet to the internet???

NO, those IPs are not part of the same subnet (10.0.0.0/24)

So how do ns1 connect to the internet?. Yes through its default gateway to the host.

eg:  default via 10.0.0.1  # Means if  destination is not local, send to 10.0.0.1.


Lets add the default rouet to the ns1 namespace.

```
# Adding the route

[root@RHEL ~]# ip netns exec ns1 ip route add default via 10.0.0.1

# Checking the added default route

[root@RHEL ~]# ip netns exec ns1 ip route 
default via 10.0.0.1 dev veth-ns    # This is the default gateway if the destination IP falls out of the same subnet

10.0.0.0/24 dev veth-ns proto kernel scope link src 10.0.0.2 
```


OK. Now when namespace ns1 sent a traffic to an IP that is not part of its subnet, based on the route table, the packet is sent through the device with IP 10.0.0.1. The device is veth-ns, this is the one end of the veth pair.

We know that when packet reach one end of veth pair, it then travel to other end. ie, to the host machine.

Great!. Our packet now reached the host.!

But here the problem is : Host do not know what to do with this traffic. Sad right?

Now what to do?


To fix this, we need to introduce 2 Linux neworking concepts.

1. IP Forwarding : It forward the packet received from one interface to another interface

2. NATTING : Transilatig the  private IP address (IP of the namespace interface 10.0.0.2 ) to the public IP that the host uses to connect to internet.

Lets enabled them:

Enable IP forwarding

```

[root@RHEL ~]# sysctl -w net.ipv4.ip_forward=1
net.ipv4.ip_forward = 1

```

Enable NAT (MASQUERADE)

We want traffic from ns1 (10.0.0.0/24) to be rewritten so the outside sees it coming from host’s IP (enp1s0).

```
# Add NAT rule

[root@RHEL ~]# iptables -t nat -A POSTROUTING -s 10.0.0.0/24 -o enp1s0 -j MASQUERADE

# Vetify the NAT rule

[root@RHEL ~]# iptables -t nat -L -n -v | grep 10.0.0.0/24
    0     0 MASQUERADE  all  --  *      enp1s0  10.0.0.0/24          0.0.0.0/0   

```

Now we also need to create forwarding rule for the ns1 and host :

```
# Forward packets from bridge/veth to host NIC:

[root@RHEL ~]# iptables -A FORWARD -i veth-host -o enp1s0 -j ACCEPT


iptables -A FORWARD -i enp1s0 -o veth-host -m state --state RELATED,ESTABLISHED -j ACCEPT

# Check the added rules.

[root@RHEL ~]# iptables -L -n -v

Chain INPUT (policy ACCEPT 0 packets, 0 bytes)

 pkts bytes target     prot opt in     out     source               destination         

Chain FORWARD (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination         
    0     0 ACCEPT     all  --  veth-host enp1s0  0.0.0.0/0            0.0.0.0/0           
    0     0 ACCEPT     all  --  enp1s0 veth-host  0.0.0.0/0            0.0.0.0/0            state RELATED,ESTABLISHED
```

Now host will forward packets both directions.



### Step 5:  Test Connectivity


Finally lets verify whether our namespace can connect to the internet!



```
# Pinging google.com from namespace ns1

root@RHEL ~]# ip netns exec ns1 ping google.com  -c 4


# YES!. Connection Success...


PING google.com (142.250.137.100) 56(84) bytes of data.
64 bytes from pnyyzb-in-f100.1e100.net (142.250.137.100): icmp_seq=1 ttl=112 time=13.7 ms
64 bytes from pnyyzb-in-f100.1e100.net (142.250.137.100): icmp_seq=2 ttl=112 time=16.3 ms
64 bytes from pnyyzb-in-f100.1e100.net (142.250.137.100): icmp_seq=3 ttl=112 time=13.6 ms
64 bytes from pnyyzb-in-f100.1e100.net (142.250.137.100): icmp_seq=4 ttl=112 time=16.3 ms

--- google.com ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3005ms
rtt min/avg/max/mdev = 13.602/14.970/16.312/1.311 ms
[root@RHEL ~]# 

```


OVERVIEW:
```

We have completed creating a Full isolated network stacj for the namespace.

1. Created a Network Namespace (ns1)

2. Isolated network stack inside Linux.

Initially only had loopback interface (lo).

3. Created a veth pair (virtual cable)

4. veth-host → host namespace

5. veth-ns → moved into ns1 namespace

This connects host ↔ ns1 like a physical cable.

Assigned IP addresses and brought interfaces up

6. Host: veth-host → 10.0.0.1/24

7. ns1: veth-ns → 10.0.0.2/24

8. Brought veth interfaces and loopback lo up

ns1 could now ping host successfully

9. Added default route inside ns1

default via 10.0.0.1

10. Allows ns1 to send traffic to destinations outside its subnet

11. Enabled IP forwarding on host

sysctl -w net.ipv4.ip_forward=1

Host now acts as a router to forward packets between veth-host and enp1s0

11. Added NAT (MASQUERADE) on host

iptables -t nat -A POSTROUTING -s 10.0.0.0/24 -o enp1s0 -j MASQUERADE

12. Rewrites source IP of packets from ns1 to host IP, so replies from Internet can return

13. Configured iptables FORWARD rules

15. Allowed traffic from ns1 → host → Internet and back

16. Ensured connection tracking works for replies

17. ested connectivity from ns1

ping google.com→ worked


```