# part 2

## static

### starting docker images

from the part 1 we have the docker images , let's start them

``` bash
cd P1
docker build -t host_mchliyah -f ./alpine_host .
docker build -t router_mchliyah -f ./frr_router .
```

we add the imaegs to the GNS3 

`edit`>`Preferences`>`Docker containers`> `new` , then select the images we just created "the same for both"

for the routers we need to change the number of the adapters to 2 or more as needed 

### satic configuration 

after ading the connection between the swich, the two routers and the host we need to configure the routers and the hosts

based on the subject we have the following configuration


|     hostname          | interfaces |     IP      | gateway  |
|-----------------------|------------|-------------|----------|
|   host_mchliyah-1     |    eth1    | 30.1.1.1/24 | 30.1.1.3 |
|   host_mchliyah-2     |    eth1    | 30.1.1.2/24 | 30.1.1.3 |
|   router_mchliyah-1   |    eth0    | 10.1.1.1/24 |          |
|   router_mchliyah-1   |    eth1    | 30.1.1.3/24 |          |
|   router_mchliyah-2   |    eth0    | 10.1.1.2/24 |          |
|   router_mchliyah-2   |    eth1    | 30.1.1.3/24 |          |


#### routers

set he ip addreses

``` bash
ip addr add <ip/mask> dev <interface> 
```
ip/mask: 10.1.1.1/24 for the first interface <eth0> (connected to swich and 10.1.1.2/24 for the other router) and 30.1.1.3/24 for the second interface <eth1> (connected to hosts on both routers)

### bridge and vxlan
#### vxlan configuration

```bash 
ip link add vxlan10 type vxlan id 10 remote <tergeted router ip> dstport 4789 dev eth0
ip link set vxlan10 up
ip addr add <IP/mask eth1> dev vxlan10
```
#### Bridge Configuration

```bash
ip link add name br0 type bridge
ip link set br0 up
ip link set vxlan10 master br0
ip link set eth1 master br0
```

#### hosts

for the host we onlly add the ip address for the eth1 interface
```bash 
ip addr add <IP/mask> dev eth1 # 30.1.1.1/24 or 30.1.1.2/24
```

we may need to change the name of the eth0 to eth1 sens the subject requier the the interface name eth1
```bash
ip link set eth0 down
ip link set eth0 name eth1
ip link set eth1 up

```

## Dynamique / Multicast


### routers configuration

set he ip addreses the same as the static part

``` bash 
ip addr add <ip/mask> dev eth0
ip addr add <ip/mask> dev eth1
```

#### VxLAN configuration

```bash
ip link add vxlan10 type vxlan id 10 dstport 4789 group <group ip> dev eth0 ttl auto 
ip link set up dev vxlan10
ip addr add < ip/mask eth1 > dev vxlan10
```

the group ip is the ip of the router that will be the target of the vxlan tunnel in the subject we have 239.1.1.1 
the eth1 vxlan device will set to 30.1.1.3/24 the same as before

#### bridge configuration

```bash
ip link add name br0 type bridge
ip link set br0 up
ip link set vxlan10 master br0
ip link set eth1 master br0
```
or

```bash

brctl addbr br0
ip link set dev br0 up
brctl addif br0 vxlan10
brctl addif br0 eth1

```
#### hosts

the same as before 