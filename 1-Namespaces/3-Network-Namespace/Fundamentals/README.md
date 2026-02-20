# Network Namespaces

A network namespace is an isolated copy of the Linux networking stack.

Each namespace has its own interfaces, routing tables, ARP tables, and firewall rules.

When created, a namespace only has the loopback interface (lo), which is initially down.

It cannot communicate outside until connected.



## Procedure to Create a Namespace Networking Stack


### Connecting to Host

Use a veth pair (virtual Ethernet cable) to link the namespace to the host or a bridge.

One end stays in the host; the other goes into the namespace.


### Assigning IP & Bringing Interfaces Up

Each side of the veth pair must have an IP address and be activated.

This allows basic layer 2 communication between host and namespace.

### Routing

A default route inside the namespace tells it where to send traffic that is not local.

Without a default route, it can only communicate with devices in the same subnet.

### Internet Access Requirements

The host must have IP forwarding enabled to act as a router.

NAT (MASQUERADE) is required to rewrite the source IP so replies from the Internet can return.

Forwarding rules must allow traffic between the veth interface and the host’s external interface.

### Bridges for Multiple Namespaces

A bridge acts like a virtual switch, connecting multiple namespaces together.

Each namespace communicates with others and can share a single gateway for Internet access.

### How docker implement this?

Docker creates a default bridge (docker0) and veth pair per container.

Containers get IPs on the bridge subnet.

Docker sets up default routes and NAT, giving containers Internet access automatically.

### Key Concepts

Namespace isolation = separate networking environment.

Veth pair = virtual cable connecting namespaces or bridge.

Bridge = virtual switch connecting multiple namespaces.

NAT + forwarding = allows isolated namespace to reach Internet.

Default route = tells the namespace where to send traffic outside its subnet.


Review the Demo session where we created a network namespace from scratch and enabled internet capabilities.


