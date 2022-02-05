# Configuring Google Cloud HA VPN



## Overview



HA VPN is a high-availability (HA) Cloud VPN solution that lets you securely connect your on-premises network to your VPC network through an IPsec VPN connection in a single region. HA VPN provides an SLA of 99.99% service availability.

HA VPN is a regional per VPC, VPN solution. HA VPN gateways have two interfaces, each with its own public IP address. When you create an HA VPN gateway, two public IP addresses are automatically chosen from different address pools. When HA VPN is configured with two tunnels, Cloud VPN offers a 99.99% service availability uptime.

In this lab you create a global VPC called **vpc-demo**, with two custom subnets in **us-east1** and **us-central1**. In this VPC, you add a Compute Engine instance in each region. You then create a second VPC called **on-prem** to simulate a customer's on-premises data center. In this second VPC, you add a subnet in region **us-central1** and a Compute Engine instance running in this region. Finally, you add an HA VPN and a cloud router in each VPC and run two tunnels from each HA VPN gateway before testing the configuration to verify the 99.99% SLA.

![img](https://cdn.qwiklabs.com/VLAXpKCCD3vR0NwjlabV3gVp9Zzf1PPu7D91KCm9VY4%3D)

### Objectives



In this lab, you learn how to perform the following tasks:

- Create two VPC networks and instances.
- Configure HA VPN gateways.
- Configure dynamic routing with VPN tunnels.
- Configure global dynamic routing mode.
- Verify and test HA VPN gateway configuration.

> **Note**: This is a Beta release of HA VPN. This feature is not covered by any SLA or deprecation policy and might be subject to backward-incompatible changes.



# Task 1. Set up a Global VPC environment



In this task you set up a Global VPC with two custom subnets and two VM instances running in each zone.

* In Cloud Shell, create a VPC network called **vpc-demo**:

```
gcloud compute networks create vpc-demo --subnet-mode custom
```

The output should look similar to this:

```
Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-03-cdb29e18d20d/global/networks/vpc-demo].
NAME: vpc-demo
SUBNET_MODE: CUSTOM
BGP_ROUTING_MODE: REGIONAL
IPV4_RANGE:
GATEWAY_IPV4:
```

* In Cloud Shell, create subnet **vpc-demo-subnet1** in the region **us-central1**:

```
gcloud beta compute networks subnets create vpc-demo-subnet1 \
--network vpc-demo --range 10.1.1.0/24 --region us-central1
```

* Create subnet **vpc-demo-subnet2** in the region **us-east1**:

```
gcloud beta compute networks subnets create vpc-demo-subnet2 \
--network vpc-demo --range 10.2.1.0/24 --region us-east1
```

* Create a firewall rule to allow all internal traffic within the network:

```
gcloud compute firewall-rules create vpc-demo-allow-internal \
  --network vpc-demo \
  --allow tcp:0-65535,udp:0-65535,icmp \
  --source-ranges 10.0.0.0/8
```



The output should look similar to this:

```
Creating firewall...working..Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-03-cdb29e18d20d/global/firewalls/vpc-demo-allow-internal].
Creating firewall...done.
NAME: vpc-demo-allow-internal
NETWORK: vpc-demo
DIRECTION: INGRESS
PRIORITY: 1000
ALLOW: tcp:0-65535,udp:0-65535,icmp
DENY:
DISABLED: False
```

* Create a firewall rule to allow SSH, ICMP traffic from anywhere:

```
gcloud compute firewall-rules create vpc-demo-allow-ssh-icmp \
    --network vpc-demo \
    --allow tcp:22,icmp
```

* Create a VM instance **vpc-demo-instance1** in zone **us-central1-b**:

```
gcloud compute instances create vpc-demo-instance1 --zone us-central1-b --subnet vpc-demo-subnet1
```

The output should look similar to this:

```
Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-03-cdb29e18d20d/zones/us-central1-b/instances/vpc-demo-instance1].
NAME: vpc-demo-instance1
ZONE: us-central1-b
MACHINE_TYPE: n1-standard-1
PREEMPTIBLE:
INTERNAL_IP: 10.1.1.2
EXTERNAL_IP: 34.71.135.218
STATUS: RUNNING
```

* Create a VM instance **vpc-demo-instance2** in zone **us-east1-b**:

```
gcloud compute instances create vpc-demo-instance2 --zone us-east1-b --subnet vpc-demo-subnet2
```



# Task 2. Set up a simulated on-premises environment



In this task you create a VPC called **on-prem** that simulates an on-premises environment from where a customer connects to the Google cloud environment.



* In Cloud Shell, create a VPC network called **on-prem**:

```
gcloud compute networks create on-prem --subnet-mode custom
```



The output should look similar to this:

```
Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-03-cdb29e18d20d/global/networks/on-prem].
NAME: on-prem
SUBNET_MODE: CUSTOM
BGP_ROUTING_MODE: REGIONAL
IPV4_RANGE:
GATEWAY_IPV4:
```

* Create a subnet called **on-prem-subnet1**:

```
gcloud beta compute networks subnets create on-prem-subnet1 \
--network on-prem --range 192.168.1.0/24 --region us-central1
```



* Create a firewall rule to allow all internal traffic within the network:

```
gcloud compute firewall-rules create on-prem-allow-internal \
  --network on-prem \
  --allow tcp:0-65535,udp:0-65535,icmp \
  --source-ranges 192.168.0.0/16
```



* Create a firewall rule to allow SSH, RDP, HTTP, and ICMP traffic to the instances:

```
gcloud compute firewall-rules create on-prem-allow-ssh-icmp \
    --network on-prem \
    --allow tcp:22,icmp
```



* Create an instance called **on-prem-instance1** in the region **us-central1**:

```
gcloud compute instances create on-prem-instance1 --zone us-central1-a --subnet on-prem-subnet1
```



# Task 3. Set up an HA VPN gateway



In this task you create an HA VPN gateway in each VPC network and then create HA VPN tunnels on each Cloud VPN gateway.



* In Cloud Shell, create an HA VPN in the **vpc-demo network**:

```
gcloud beta compute vpn-gateways create vpc-demo-vpn-gw1 --network vpc-demo --region us-central1
```



The output should look similar to this:

```
Creating VPN Gateway...done.   
NAME: vpc-demo-vpn-gw1
INTERFACE0: 35.242.117.95
INTERFACE1: 35.220.73.93
NETWORK: vpc-demo
REGION: us-central1
```

* Create an HA VPN in the **on-prem** network:

```
gcloud beta compute vpn-gateways create on-prem-vpn-gw1 --network on-prem --region us-central1
```



* View details of the **vpc-demo-vpn-gw1** gateway to verify its settings:

```
gcloud beta compute vpn-gateways describe vpc-demo-vpn-gw1 --region us-central1
```



The output should look similar to this:

```
creationTimestamp: '2022-01-25T03:02:20.983-08:00'
id: '7306781839576950355'
kind: compute#vpnGateway
labelFingerprint: 42WmSpB8rSM=
name: vpc-demo-vpn-gw1
network: https://www.googleapis.com/compute/beta/projects/qwiklabs-gcp-03-cdb29e18d20d/global/networks/vpc-demo
region: https://www.googleapis.com/compute/beta/projects/qwiklabs-gcp-03-cdb29e18d20d/regions/us-central1
selfLink: https://www.googleapis.com/compute/beta/projects/qwiklabs-gcp-03-cdb29e18d20d/regions/us-central1/vpnGateways/vpc-demo-vpn-gw1
vpnInterfaces:
- id: 0
  ipAddress: 35.242.117.95
- id: 1
  ipAddress: 35.220.73.93
```

* View details of the **on-prem-vpn-gw1** vpn-gateway to verify its settings:

```
gcloud beta compute vpn-gateways describe on-prem-vpn-gw1 --region us-central1
```



The output should look similar to this:

```
creationTimestamp: '2022-01-25T03:03:34.305-08:00'
id: '3697047034868688873'
kind: compute#vpnGateway
labelFingerprint: 42WmSpB8rSM=
name: on-prem-vpn-gw1
network: https://www.googleapis.com/compute/beta/projects/qwiklabs-gcp-03-cdb29e18d20d/global/networks/on-prem
region: https://www.googleapis.com/compute/beta/projects/qwiklabs-gcp-03-cdb29e18d20d/regions/us-central1
selfLink: https://www.googleapis.com/compute/beta/projects/qwiklabs-gcp-03-cdb29e18d20d/regions/us-central1/vpnGateways/on-prem-vpn-gw1
vpnInterfaces:
- id: 0
  ipAddress: 35.242.106.234
- id: 1
  ipAddress: 35.220.88.140
```



## Create cloud routers



* Create a cloud router in the **vpc-demo** network:

```
gcloud compute routers create vpc-demo-router1 \
    --region us-central1 \
    --network vpc-demo \
    --asn 65001
```



The output should look similar to this:

```
Creating router [vpc-demo-router1]...done.     
NAME: vpc-demo-router1
REGION: us-central1
NETWORK: vpc-demo
```

* Create a cloud router in the **on-prem** network:

```
gcloud compute routers create on-prem-router1 \
    --region us-central1 \
    --network on-prem \
    --asn 65002
```



# Task 4. Create two VPN tunnels



In this task you create VPN tunnels between the two gateways. For HA VPN setup, you add two tunnels from each gateway to the remote setup. You create a tunnel on **interface0** and connect to **interface0** on the remote gateway. Next, you create another tunnel on **interface1** and connect to **interface1** on the remote gateway.

When you run HA VPN tunnels between two Google Cloud VPCs, you need to make sure that the tunnel on **interface0** is connected to **interface0** on the remote VPN gateway. Similarly, the tunnel on **interface1** must be connected to **interface1** on the remote VPN gateway.

> In your own environment, if you run HA VPN to a remote VPN gateway on-premises for a customer, you can connect in one of the following ways:
>
> 
>
> - *Two on-premises VPN gateway devices:* Each of the tunnels from each interface on the Cloud VPN gateway must be connected to its own peer gateway.
> - 
>
> - *A single on-premises VPN gateway device with two interfaces:* Each of the tunnels from each interface on the Cloud VPN gateway must be connected to its own interface on the peer gateway.
> - 
>
> - *A single on-premises VPN gateway device with a single interface:* Both of the tunnels from each interface on the Cloud VPN gateway must be connected to the same interface on the peer gateway.



In this lab you are simulating an on-premises setup with both VPN gateways in Google Cloud. You ensure that **interface0** of one gateway connects to **interface0** of the other and **interface1** connects to **interface1** of the remote gateway.



* Create the first VPN tunnel in the **vpc-demo** network:

```
gcloud beta compute vpn-tunnels create vpc-demo-tunnel0 \
    --peer-gcp-gateway on-prem-vpn-gw1 \
    --region us-central1 \
    --ike-version 2 \
    --shared-secret [SHARED_SECRET] \
    --router vpc-demo-router1 \
    --vpn-gateway vpc-demo-vpn-gw1 \
    --interface 0
```



The output should look similar to this:

```
Creating VPN tunnel...done.     
NAME: vpc-demo-tunnel0
REGION: us-central1
GATEWAY: vpc-demo-vpn-gw1
VPN_INTERFACE: 0
PEER_ADDRESS: 35.242.106.234
```

* Create the second VPN tunnel in the **vpc-demo** network:

```
gcloud beta compute vpn-tunnels create vpc-demo-tunnel1 \
    --peer-gcp-gateway on-prem-vpn-gw1 \
    --region us-central1 \
    --ike-version 2 \
    --shared-secret [SHARED_SECRET] \
    --router vpc-demo-router1 \
    --vpn-gateway vpc-demo-vpn-gw1 \
    --interface 1
```



* Create the first VPN tunnel in the **on-prem** network:

```
gcloud beta compute vpn-tunnels create on-prem-tunnel0 \
    --peer-gcp-gateway vpc-demo-vpn-gw1 \
    --region us-central1 \
    --ike-version 2 \
    --shared-secret [SHARED_SECRET] \
    --router on-prem-router1 \
    --vpn-gateway on-prem-vpn-gw1 \
    --interface 0
```



* Create the second VPN tunnel in the **on-prem** network:

```
gcloud beta compute vpn-tunnels create on-prem-tunnel1 \
    --peer-gcp-gateway vpc-demo-vpn-gw1 \
    --region us-central1 \
    --ike-version 2 \
    --shared-secret [SHARED_SECRET] \
    --router on-prem-router1 \
    --vpn-gateway on-prem-vpn-gw1 \
    --interface 1
```



# Task 5. Create Border Gateway Protocol (BGP) peering for each tunnel



In this task you configure BGP peering for each VPN tunnel between **vpc-demo** and VPC **on-prem**. HA VPN requires dynamic routing to enable 99.99% availability.

* Create the router interface for **tunnel0** in network **vpc-demo**:

```
gcloud compute routers add-interface vpc-demo-router1 \
    --interface-name if-tunnel0-to-on-prem \
    --ip-address 169.254.0.1 \
    --mask-length 30 \
    --vpn-tunnel vpc-demo-tunnel0 \
    --region us-central1
```



The output should look similar to this:

```
Updated [https://www.googleapis.com/compute/v1/projects/binal-sandbox/regions/us-central1/routers/vpc-demo-router1].
```

* Create the BGP peer for **tunnel0** in network **vpc-demo**:

```
gcloud compute routers add-bgp-peer vpc-demo-router1 \
    --peer-name bgp-on-prem-tunnel0 \
    --interface if-tunnel0-to-on-prem \
    --peer-ip-address 169.254.0.2 \
    --peer-asn 65002 \
    --region us-central1
```

The output should look similar to this:

```
Creating peer [bgp-on-prem-tunnel0] in router [vpc-demo-router1]...done.
```

* Create a router interface for **tunnel1** in network **vpc-demo**:

```
gcloud compute routers add-interface vpc-demo-router1 \
    --interface-name if-tunnel1-to-on-prem \
    --ip-address 169.254.1.1 \
    --mask-length 30 \
    --vpn-tunnel vpc-demo-tunnel1 \
    --region us-central1
```



* Create the BGP peer for **tunnel1** in network **vpc-demo**:

```
gcloud compute routers add-bgp-peer vpc-demo-router1 \
    --peer-name bgp-on-prem-tunnel1 \
    --interface if-tunnel1-to-on-prem \
    --peer-ip-address 169.254.1.2 \
    --peer-asn 65002 \
    --region us-central1
```



* Create a router interface for **tunnel0** in network **on-prem**:

```
gcloud compute routers add-interface on-prem-router1 \
    --interface-name if-tunnel0-to-vpc-demo \
    --ip-address 169.254.0.2 \
    --mask-length 30 \
    --vpn-tunnel on-prem-tunnel0 \
    --region us-central1
```



* Create the BGP peer for **tunnel0** in network **on-prem**:

```
gcloud compute routers add-bgp-peer on-prem-router1 \
    --peer-name bgp-vpc-demo-tunnel0 \
    --interface if-tunnel0-to-vpc-demo \
    --peer-ip-address 169.254.0.1 \
    --peer-asn 65001 \
    --region us-central1
```



* Create a router interface for **tunnel1** in network **on-prem**:

```
gcloud compute routers add-interface  on-prem-router1 \
    --interface-name if-tunnel1-to-vpc-demo \
    --ip-address 169.254.1.2 \
    --mask-length 30 \
    --vpn-tunnel on-prem-tunnel1 \
    --region us-central1
```



* Create the BGPp peer for **tunnel1** in network **on-prem**:

```
gcloud compute routers add-bgp-peer  on-prem-router1 \
    --peer-name bgp-vpc-demo-tunnel1 \
    --interface if-tunnel1-to-vpc-demo \
    --peer-ip-address 169.254.1.1 \
    --peer-asn 65001 \
    --region us-central1
```



# Task 6. Verify router configurations



In this task you verify the router configurations in both VPCs. You configure firewall rules to allow traffic between each VPC and verify the status of the tunnels. You also verify private connectivity over VPN between each VPC and enable global routing mode for the VPC.

* View details of Cloud Router **vpc-demo-router1** to verify its settings:

```
gcloud compute routers describe vpc-demo-router1 \
    --region us-central1
```



The output should look similar to this:

```
bgp:
  advertiseMode: DEFAULT
  asn: 65001
  keepaliveInterval: 20
bgpPeers:
- bfd:
    minReceiveInterval: 1000
    minTransmitInterval: 1000
    multiplier: 5
    sessionInitializationMode: DISABLED
  enable: 'TRUE'
  interfaceName: if-tunnel0-to-on-prem
  ipAddress: 169.254.0.1
  name: bgp-on-prem-tunnel0
  peerAsn: 65002
  peerIpAddress: 169.254.0.2
- bfd:
    minReceiveInterval: 1000
    minTransmitInterval: 1000
    multiplier: 5
    sessionInitializationMode: DISABLED
  enable: 'TRUE'
  interfaceName: if-tunnel1-to-on-prem
  ipAddress: 169.254.1.1
  name: bgp-on-prem-tunnel1
  peerAsn: 65002
  peerIpAddress: 169.254.1.2
creationTimestamp: '2022-01-25T03:06:23.370-08:00'
id: '2408056426544129856'
interfaces:
- ipRange: 169.254.0.1/30
  linkedVpnTunnel: https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-03-cdb29e18d20d/regions/us-central1/vpnTunnels/vpc-demo-tunnel0
  name: if-tunnel0-to-on-prem
- ipRange: 169.254.1.1/30
  linkedVpnTunnel: https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-03-cdb29e18d20d/regions/us-central1/vpnTunnels/vpc-demo-tunnel1
  name: if-tunnel1-to-on-prem
kind: compute#router
name: vpc-demo-router1
network: https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-03-cdb29e18d20d/global/networks/vpc-demo
region: https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-03-cdb29e18d20d/regions/us-central1
selfLink: https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-03-cdb29e18d20d/regions/us-central1/routers/vpc-demo-router1
```

* View details of Cloud Router **on-prem-router1** to verify its settings:

```
gcloud compute routers describe on-prem-router1 \
    --region us-central1
```



The output should look similar to this:

```
bgp:
  advertiseMode: DEFAULT
  asn: 65002
  keepaliveInterval: 20
bgpPeers:
- bfd:
    minReceiveInterval: 1000
    minTransmitInterval: 1000
    multiplier: 5
    sessionInitializationMode: DISABLED
  enable: 'TRUE'
  interfaceName: if-tunnel0-to-vpc-demo
  ipAddress: 169.254.0.2
  name: bgp-vpc-demo-tunnel0
  peerAsn: 65001
  peerIpAddress: 169.254.0.1
- bfd:
    minReceiveInterval: 1000
    minTransmitInterval: 1000
    multiplier: 5
    sessionInitializationMode: DISABLED
  enable: 'TRUE'
  interfaceName: if-tunnel1-to-vpc-demo
  ipAddress: 169.254.1.2
  name: bgp-vpc-demo-tunnel1
  peerAsn: 65001
  peerIpAddress: 169.254.1.1
creationTimestamp: '2022-01-25T03:07:40.360-08:00'
id: '3252882979067946771'
interfaces:
- ipRange: 169.254.0.2/30
  linkedVpnTunnel: https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-03-cdb29e18d20d/regions/us-central1/vpnTunnels/on-prem-tunnel0
  name: if-tunnel0-to-vpc-demo
- ipRange: 169.254.1.2/30
  linkedVpnTunnel: https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-03-cdb29e18d20d/regions/us-central1/vpnTunnels/on-prem-tunnel1
  name: if-tunnel1-to-vpc-demo
kind: compute#router
name: on-prem-router1
network: https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-03-cdb29e18d20d/global/networks/on-prem
region: https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-03-cdb29e18d20d/regions/us-central1
selfLink: https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-03-cdb29e18d20d/regions/us-central1/routers/on-prem-router1
```



## Configure firewall rules to allow traffic from the remote VPC



Configure firewall rules to allow traffic from the private IP ranges of peer VPN.

* Allow traffic from network VPC **on-prem** to **vpc-demo**:

```
gcloud compute firewall-rules create vpc-demo-allow-subnets-from-on-prem \
    --network vpc-demo \
    --allow tcp,udp,icmp \
    --source-ranges 192.168.1.0/24
```



The output should look similar to this:

```
Creating firewall...working..Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-03-cdb29e18d20d/global/firewalls/vpc-demo-allow-subnets-from-on-prem].
Creating firewall...done.
NAME: vpc-demo-allow-subnets-from-on-prem
NETWORK: vpc-demo
DIRECTION: INGRESS
PRIORITY: 1000
ALLOW: tcp,udp,icmp
DENY:
DISABLED: False
```

* Allow traffic from **vpc-demo** to network VPC **on-prem**:

```
gcloud compute firewall-rules create on-prem-allow-subnets-from-vpc-demo \
    --network on-prem \
    --allow tcp,udp,icmp \
    --source-ranges 10.1.1.0/24,10.2.1.0/24
```



## Verify the status of the tunnels

* List the VPN tunnels you just created:

```
gcloud beta compute vpn-tunnels list
```



There should be four VPN tunnels (two tunnels for each VPN gateway). The output should look similar to this:

```
NAME: on-prem-tunnel0
REGION: us-central1
GATEWAY: on-prem-vpn-gw1
PEER_ADDRESS: 35.242.117.95
NAME: on-prem-tunnel1
REGION: us-central1
GATEWAY: on-prem-vpn-gw1
PEER_ADDRESS: 35.220.73.93
NAME: vpc-demo-tunnel0
REGION: us-central1
GATEWAY: vpc-demo-vpn-gw1
PEER_ADDRESS: 35.242.106.234
NAME: vpc-demo-tunnel1
REGION: us-central1
GATEWAY: vpc-demo-vpn-gw1
PEER_ADDRESS: 35.220.88.140
```

* Verify that **vpc-demo-tunnel0** tunnel is up:

```
gcloud beta compute vpn-tunnels describe vpc-demo-tunnel0 \
      --region us-central1
```



The tunnel output should show detailed status as *Tunnel is up and running.*

```
creationTimestamp: '2022-01-25T03:21:05.238-08:00'
description: ''
detailedStatus: Tunnel is up and running.
id: '3268990180169769934'
ikeVersion: 2
kind: compute#vpnTunnel
labelFingerprint: 42WmSpB8rSM=
localTrafficSelector:
- 0.0.0.0/0
name: vpc-demo-tunnel0
peerGcpGateway: https://www.googleapis.com/compute/beta/projects/qwiklabs-gcp-03-cdb29e18d20d/regions/us-central1/vpnGateways/on-prem-vpn-gw1
peerIp: 35.242.106.234
region: https://www.googleapis.com/compute/beta/projects/qwiklabs-gcp-03-cdb29e18d20d/regions/us-central1
remoteTrafficSelector:
- 0.0.0.0/0
router: https://www.googleapis.com/compute/beta/projects/qwiklabs-gcp-03-cdb29e18d20d/regions/us-central1/routers/vpc-demo-router1
selfLink: https://www.googleapis.com/compute/beta/projects/qwiklabs-gcp-03-cdb29e18d20d/regions/us-central1/vpnTunnels/vpc-demo-tunnel0
sharedSecret: '*************'
sharedSecretHash: AOs4oVY4bX91gba6DIeg1DbtzWTj
status: ESTABLISHED
vpnGateway: https://www.googleapis.com/compute/beta/projects/qwiklabs-gcp-03-cdb29e18d20d/regions/us-central1/vpnGateways/vpc-demo-vpn-gw1
vpnGatewayInterface: 0
```

* Verify that **vpc-demo-tunnel1** tunnel is up:

```
gcloud beta compute vpn-tunnels describe vpc-demo-tunnel1 \
      --region us-central1
```



The tunnel output should show detailed status as *Tunnel is up and running.*

* Verify that **on-prem-tunnel0** tunnel is up:

```
gcloud beta compute vpn-tunnels describe on-prem-tunnel0 \
      --region us-central1
```



The tunnel output should show detailed status as *Tunnel is up and running.*

* Verify that **on-prem-tunnel1** tunnel is up:

```
gcloud beta compute vpn-tunnels describe on-prem-tunnel1 \
      --region us-central1
```



The tunnel output should show detailed status as *Tunnel is up and running.*



## Verify private connectivity over VPN



* Open a new Cloud Shell tab and type the following to connect via SSH to the instance **on-prem-instance1**:

```
gcloud compute ssh on-prem-instance1 --zone us-central1-a
```



Type "y" to confirm that you want to continue.

* Press **Enter** twice to skip creating a password.

* From the instance **on-prem-instance1** in network **on-prem**, to reach instances in network **vpc-demo**, ping 10.1.1.2:

```
ping -c 4 10.1.1.2
```



Pings are successful. The output should look similar to this:

```
PING 10.1.1.2 (10.1.1.2) 56(84) bytes of data.
64 bytes from 10.1.1.2: icmp_seq=1 ttl=62 time=9.65 ms
64 bytes from 10.1.1.2: icmp_seq=2 ttl=62 time=2.01 ms
64 bytes from 10.1.1.2: icmp_seq=3 ttl=62 time=1.71 ms
64 bytes from 10.1.1.2: icmp_seq=4 ttl=62 time=1.77 ms
--- 10.1.1.2 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 8ms
rtt min/avg/max/mdev = 1.707/3.783/9.653/3.391 ms
```



## Global routing with VPN

HA VPN is a regional resource and cloud router that by default only sees the routes in the region in which it is deployed. To reach instances in a different region than the cloud router, you need to enable global routing mode for the VPC. This allows the cloud router to see and advertise routes from other regions.

* Open a new Cloud Shell tab and update the **bgp-routing mode** from **vpc-demo** to **GLOBAL**:

```
gcloud compute networks update vpc-demo --bgp-routing-mode GLOBAL
```



* Verify the change:

```
gcloud compute networks describe vpc-demo
```



The output should look similar to this:

```
autoCreateSubnetworks: false
creationTimestamp: '2022-01-25T02:52:58.553-08:00'
id: '4735939730452146277'
kind: compute#network
name: vpc-demo
routingConfig:
  routingMode: GLOBAL
selfLink: https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-03-cdb29e18d20d/global/networks/vpc-demo
subnetworks:
- https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-03-cdb29e18d20d/regions/us-central1/subnetworks/vpc-demo-subnet1
- https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-03-cdb29e18d20d/regions/us-east1/subnetworks/vpc-demo-subnet2
x_gcloud_bgp_routing_mode: GLOBAL
x_gcloud_subnet_mode: CUSTOM
```

* From the Cloud Shell tab that is currently connected to the instance in network **on-prem** via **ssh**, ping the instance **vpc-demo-instance2** in region us-east1:

```
ping -c 2 10.2.1.2
```



Pings are successful. The output should look similar to this:

```
PING 10.2.1.2 (10.2.1.2) 56(84) bytes of data.
64 bytes from 10.2.1.2: icmp_seq=1 ttl=62 time=34.9 ms
64 bytes from 10.2.1.2: icmp_seq=2 ttl=62 time=32.2 ms
--- 10.2.1.2 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 2ms
rtt min/avg/max/mdev = 32.189/33.528/34.867/1.339 ms
```



# Task 7. Verify and test the configuration of HA VPN tunnels

In this task you will test and verify that the high availability configuration of each HA VPN tunnel is successful.

* Open a new Cloud Shell tab.

* Bring **tunnel0** in network **vpc-demo** down:

```
gcloud compute vpn-tunnels delete vpc-demo-tunnel0  --region us-central1
```



Respond "y" when asked to verify the deletion. The respective **tunnel0** in network **on-prem** will go down.

* Verify that the tunnel is down:

```
gcloud compute vpn-tunnels describe on-prem-tunnel0  --region us-central1
```



The detailed status should show as *Handshake_with_peer_broken*.

```
creationTimestamp: '2022-01-25T03:22:33.581-08:00'
description: ''
detailedStatus: Handshake with peer broken for unknown reason. Trying again soon.
id: '4116279561430393750'
ikeVersion: 2
kind: compute#vpnTunnel
localTrafficSelector:
- 0.0.0.0/0
name: on-prem-tunnel0
peerGcpGateway: https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-03-cdb29e18d20d/regions/us-central1/vpnGateways/vpc-demo-vpn-gw1
peerIp: 35.242.117.95
region: https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-03-cdb29e18d20d/regions/us-central1
remoteTrafficSelector:
- 0.0.0.0/0
router: https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-03-cdb29e18d20d/regions/us-central1/routers/on-prem-router1
selfLink: https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-03-cdb29e18d20d/regions/us-central1/vpnTunnels/on-prem-tunnel0
sharedSecret: '*************'
sharedSecretHash: AO3jeFtewmjvTMO7JEM5RuyCtqaa
status: FIRST_HANDSHAKE
vpnGateway: https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-03-cdb29e18d20d/regions/us-central1/vpnGateways/on-prem-vpn-gw1
vpnGatewayInterface: 0
```

* Switch to the previous Cloud Shell tab that has the open **ssh** session running, and verify the pings between the instances in network **vpc-demo** and network **on-prem**:

```
ping -c 3 10.1.1.2
```



The output should look similar to this:

```
PING 10.1.1.2 (10.1.1.2) 56(84) bytes of data.
64 bytes from 10.1.1.2: icmp_seq=1 ttl=62 time=6.31 ms
64 bytes from 10.1.1.2: icmp_seq=2 ttl=62 time=1.13 ms
64 bytes from 10.1.1.2: icmp_seq=3 ttl=62 time=1.20 ms
--- 10.1.1.2 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 5ms
rtt min/avg/max/mdev = 1.132/2.882/6.312/2.425 ms
```

Pings are still successful because the traffic is now sent over the second tunnel. You have successfully configured HA VPN tunnels.



# Task 8. Clean up lab environment



In this task you clean up the resources you have used, to save on costs and reduce resource usage.



## Delete VPN tunnels



* From Cloud Shell, type the following commands to delete the remaining tunnels. Type "y" to confirm each action when asked:

```
gcloud compute vpn-tunnels delete on-prem-tunnel0  --region us-central1
```



```
gcloud compute vpn-tunnels delete vpc-demo-tunnel1  --region us-central1
```



```
gcloud compute vpn-tunnels delete on-prem-tunnel1  --region us-central1
```



## Remove BGP peering



* Type the following commands from each BGP peer to remove peering:

```
gcloud compute routers remove-bgp-peer vpc-demo-router1 --peer-name bgp-on-prem-tunnel0 --region us-central1
```



```
gcloud compute routers remove-bgp-peer vpc-demo-router1 --peer-name bgp-on-prem-tunnel1 --region us-central1
```



```
gcloud compute routers remove-bgp-peer on-prem-router1 --peer-name bgp-vpc-demo-tunnel0 --region us-central1
```



```
gcloud compute routers remove-bgp-peer on-prem-router1 --peer-name bgp-vpc-demo-tunnel1 --region us-central1
```



## Delete cloud routers



* Type each command to delete the routers. Type "y" to confirm each action when asked:

```
gcloud compute  routers delete on-prem-router1 --region us-central1
```



```
gcloud compute  routers delete vpc-demo-router1 --region us-central1
```



## Delete VPN gateways



* Type each command to delete the VPN gateways. Type "y" to confirm each action when asked:

```
gcloud beta compute vpn-gateways delete vpc-demo-vpn-gw1 --region us-central1
```



```
gcloud beta compute vpn-gateways delete on-prem-vpn-gw1 --region us-central1
```



## Delete instances



* Type the following commands to delete each instance. Type "y" to confirm each action when asked:

```
gcloud compute instances delete vpc-demo-instance1 --zone us-central1-b
```



```
gcloud compute instances delete vpc-demo-instance2 --zone us-east1-b
```



```
gcloud compute instances delete on-prem-instance1 --zone us-central1-a
```



## Delete firewall rules



* Type the following to delete the firewall rules. Type "y" to confirm each action when asked:

```
gcloud beta compute firewall-rules delete vpc-demo-allow-internal
```



```
gcloud beta compute firewall-rules delete on-prem-allow-subnets-from-vpc-demo
```



```
gcloud beta compute firewall-rules delete on-prem-allow-ssh-icmp
```



```
gcloud beta compute firewall-rules delete on-prem-allow-internal
```



```
gcloud beta compute firewall-rules delete vpc-demo-allow-subnets-from-on-prem
```



```
gcloud beta compute firewall-rules delete vpc-demo-allow-ssh-icmp
```



## Delete subnets



* Type the following to delete the subnets. Type "y" to confirm each action when asked:

```
gcloud beta compute networks subnets delete vpc-demo-subnet1 --region us-central1
```



```
gcloud beta compute networks subnets delete vpc-demo-subnet2 --region us-east1
```



```
gcloud beta compute networks subnets delete on-prem-subnet1 --region us-central1
```



## Delete VPC



* Type these commands to delete the VPCs. Type "y" to confirm each action when asked:

```
gcloud compute networks delete vpc-demo
```



```
gcloud compute networks delete on-prem
```



# Task 9: Review



In this lab you configured HA VPN gateways. You also configured dynamic routing with VPN tunnels and configured global dynamic routing mode. Finally you verified that HA VPN is configured and functioning correctly.

