# part 3

after adding the routers and the connections between the for of them and the three alpine hosts basd on the recomanded topology or import the file from here, we can start configuration.
based on the subject we have the host_1 connected to the router_2 by eth1 
we can set host adapters to 2 and use the eth1 or by changing the name of the interface 

```bash
ip link set eth0 down
ip link set eth0 name eth1
ip link set eth1 up

```

### VTEP Configuration
In our project, there are three VTEPs (Virtual Termination End-Points), which are our "Leafs" in the topology.

## router 2
bridge configuration

``` bash
bash

brctl addbr br0 
ip link set up dev br0
ip link add vxlan10 type vxlan id 10 dstport 4789
ip link set up dev vxlan10
brctl addif br0 vxlan10
brctl addif br0 eth1
```

set the hostname , inter the shell and disable the ipv6 forwarding

```bash
hostname router_mchliyah_1 # <reflector name>

vtysh

no ipv6 forwarding
```
## Setup interface eth0
```bash

interface eth0
ip address 10.1.1.2/30
ip ospf area 0
```

## Setup interface lo
``` bash

interface lo
ip address 1.1.1.2/32
ip ospf area 0
```

## Setup bgp
```bash
router bgp 1
neighbor 1.1.1.1 remote-as 1
neighbor 1.1.1.1 update-source lo
```

## Setup evpn
```bash
address-family l2vpn evpn
neighbor 1.1.1.1 activate
advertise-all-vni
exit-address-family

router ospf
```

## router 3
we will mot configure the bridge on this router to highlight the importance of the bridges in our eVPN.

``` bash
vtysh
config t

hostname <reflector name> 
no ipv6 forwarding

interface eth1
ip address 10.1.1.6/30
ip ospf area 0

interface lo
ip address 1.1.1.3/32
ip ospf area 0

router bgp 1
neighbor 1.1.1.1 remote-as 1
neighbor 1.1.1.1 update-source lo

address-family l2vpn evpn
neighbor 1.1.1.1 activate
exit-address-family

router ospf

```

## router 4

``` bash

brctl addbr br0
ip link set up dev br0
ip link add vxlan10 type vxlan id 10 dstport 4789
ip link set up dev vxlan10
brctl addif br0 vxlan10
brctl addif br0 eth0

vtysh 

config t

hostname <vtep name> 
no ipv6 forwarding


interface eth2
ip address 10.1.1.10/30
ip ospf area 0

interface lo
ip address 1.1.1.4/32
ip ospf area 0

router bgp 1
neighbor 1.1.1.1 remote-as 1
neighbor 1.1.1.1 update-source lo

address-family l2vpn evpn
neighbor 1.1.1.1 activate
advertise-all-vni
exit-address-family

router ospf
```


## RR router 1 | reouter reflector

```bash 
vtysh

config t

hostname <reflector name>
no ipv6 forwarding

interface eth0
ip address 10.1.1.1/30

interface eth1
ip address 10.1.1.5/30

interface eth2
ip address 10.1.1.9/30

interface lo
ip address 1.1.1.1/32

router bgp 1
neighbor ibgp peer-group
neighbor ibgp remote-as 1
neighbor ibgp update-source lo
bgp listen range 1.1.1.0/29 peer-group ibgp

address-family l2vpn evpn
neighbor ibgp activate
neighbor ibgp route-reflector-client
exit-address-family

router ospf
network 0.0.0.0/0 area 0

line vty
```