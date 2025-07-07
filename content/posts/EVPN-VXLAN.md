---
title: "EVPN VXLAN"
date: 2025-06-30T10:32:05Z
draft: true
---

As a bit of depature from DevOps skills, I've been delving into EVPN-VXLAN; I've done a design and implementation on EVPN-MPLS in my SP career, and thought it was time to look into EVPN and VXLAN.

VXLAN is a L2 overlay technology that encapsulates L2 frames within a L4 UDP datagram. Interestingly, it's not an EoIP tunnelling protocol like GRE, but EoUDP.

Arguably the biggest reason for this is because of the anticipated use case; VXLAN is designed to scale Data Centres, which would likely use multiple links and/or link aggregation. Having a L4 header visible to the underlay network allows a larger entropy for link aggregation hashing and ECMP.

VXLAN uses the concept of Virtual Network Identifiers (VNI) which can scale to around 16 million, compared to 4096 for single-tag VLANs.

EVPN-VXLAN (as with EVPN-MPLS) brings the benefit of control-plane MAC learning, as well as efficient inter-subnet routing using Virtual Gateway Addressing and active-active multihoming, allowing the efficient use of multiple uplinks, again critical in high-traffic high-speed DC requirements.

The best-practice deployment for DC networks is leaf-spine; each leaf is connected to every spine, keeping the "traffic cost" between any two leafs identical. This allows for predictable latency and failover times, as well as a redundant and resilient deployment.

I used Arista switches to build a small lab to learn and demonstrate EVPN-VXLAN. The underlay is OSPF and MP-BGP with a vlan-aware bundle configured.

* A VLAN-aware bundle can map many VLANs to the same EVPN Routing Instance (EVI) - otherwise known as a _Virtual-Switch_
* A VLAN-based service maps exactly one VLAN to one EVI - one EVPN per VLAN 

![Arista EVPN-VXLAN Lab](/EVPN-VXLAN-lab.png)

I've set the fabric up as a Centrally-Routed Bridging (CRB) Fabric; the L3 VXLAN Gatway is on the Spine device, and the leafs act as the L2 VXLAN Gateway. 

This is the preferred setup for North-South traffic patterns; traffic that will likely be egressing the fabric rather than East-West towards/from other devices intra-fabric.

The linux servers send dot1Q-tagged traffic on VLAN10, with the leafs mapping VLAN10 to VNI 12345. 


Output of VLAN/VNI Mapping:
 
    Leaf1#show vxlan vni
    VNI to VLAN Mapping for Vxlan1
    VNI         VLAN       Source       Interface       802.1Q Tag
    ----------- ---------- ------------ --------------- ----------
    12345       10         static       Ethernet2       10        
                                        Vxlan1          10     

</br>

Output of BGP EVPN MAC-IP (Type-2) routes 

    Leaf1#show bgp evpn route-type mac-ip
 

            Network                Next Hop              Metric  LocPref Weight  Path
    * >     RD: 192.168.0.1:12 mac-ip 12345 5003.000a.0000
                                    -                     -       -       0       i
    * >     RD: 192.168.0.3:12 mac-ip 12345 5003.000c.0000
                                    192.168.0.3           -       100     0       i Or-ID: 192.168.0.3 C-LST: 1.1.1.1 


<b>Interestingly</b> with the comparison of CRB EVPN-VXLAN on JUNOS, there is a switch that enables the MAC-IP/ARP bindings to be advertised to <b>L2</b> VXLAN Gateways - this isn't usually the case as ARP bindings are sync'd only with L3 VXLAN Gateways.


