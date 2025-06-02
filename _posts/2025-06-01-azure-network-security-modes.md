---
#layout: post-argon
title: Azure Network Security Modes
date: 2025-06-01 18:00:00 +0000
#last_modified_at: 2025-06-01 18:00:00 +0000
author: Igor
categories: [Azure, Networking]
tags: [azure, security]
#permalink: /blog/azure-network-security-modes
slug: azure-network-security-modes
excerpt_separator: <!--more-->
#redirect_from: [/post/azure-network-security-modes]
image: https://github.com/user-attachments/assets/6ad6c2fb-d4fa-4544-bde6-02d0eb3e508d
description: >-
    Describing network security modes
#published: false
---
<div style="text-align: center">

![Network Diagram - Modes](https://github.com/user-attachments/assets/6ad6c2fb-d4fa-4544-bde6-02d0eb3e508d)

</div>

Organic, AI-free. Work in progress.

<!--more-->

## Network segmentation rationale

### Ancient times

In the old days, when servers were living in on-premise datacenters, there was a concept of network segmentation.

**Hubs and Switches**

Servers were connected by hubs and switches.

Every server connected to the same network hub was receiving the same datagrams, including ones not addressed to them. These hubs were also called repeaters.

Network switches are helping to reduce the number of situations where datagrams are repeated on multiple ports when the switch "remembers" who is on the port. It is still not perfect at remembering everyone, especially in big networks.

**Subnets**

And networks were big. You remember subnet classes A/B/C? C means 256 addresses. It was normal to have hundreds and thousands of servers connected via hubs and switches, and they formed subnets or broadcast domains.

To reiterate: any traffic in the subnet is broadcast and can be overheard by any interface on that subnet.

**Routers** are used to interconnect and remove broadcast traffic between subnets, allowing only traffic destined for these subnets.

Limiting broadcast traffic is good, but routers are expensive. That's why subnets were big and had many servers.

Traditional routers were not smart, so they were not able to filter ports, protocols, and other attributes that security guys like.

**Firewalls** are used to filter traffic, and were even rarer and expensive than routers.

It was so expensive that it was only placed on critical network boundaries, like dividing internal networks from DMZ and internets.

The cost and rarity of firewalls forced companies to group their servers into network segments, where everyone trusts each other.

**Network segmentation**

At that time, network traffic was rarely encrypted -- everyone in the subnet could potentially hear all network conversations.

Operating systems like Windows 2000 did not have a personal firewall, so all ports were open by default -- making them vulnerable to any bad guy or a virus that is on the network

Authentication protocols and passwords were weak (and no MFA!) -- to the pleasure of hackers.

These aspects led to the necessity of making sure that all neighbours in the networks are trusted.

Here comes a concept of network segment -- a part of a network that groups servers with a similar level of trust together, protected by a single firewall.

Typical network segments were Prod /  Dev / Workplace / DMZ / Internet, and all devices within the same segment were treated the same.

### Modern times

Nowadays, when you hear the term network hub, it does not mean repeater, but it means a central place connecting multiple virtual networks, where routing and traffic filtering happen.

Virtual Networks in Azure run in a shared, multitenant environment of the cloud provider, and 
- network virtualisation protocol ensures that traffic is not visible to other customers -- no bad neighbours in the subnet can overhear your conversations
- traffic filtering platform is processing every datagram - firewall capabilities are available on every connected interface via NSG

More than that, 
- modern operating systems have firewalls enabled by default (and it's a shame if your company disable them as a standard)
- modern network protocols keep traffic properly authenticated (MFA/SSO/SAML/OAuth -- all these beauties) and encrypted (TLS/AES/oh-yes) from end-to-end

This all makes the concept of network segmentation mostly irrelevant for Azure virtual networks, since issues that segmentation was initially addressing no longer exist.

_Henceforth_, I declare the traditional network segmentation obsolete.


## Azure Network Security Modes

New times are coming, security guys stay the same.

They still want to control and see the traffic on a firewall, and don't want to rely on zero trust principles.

To please them, I want to introduce you to a modern way of organising network filtering, which I call Azure Network Security Modes.

### Mode 1, aka Cloud Native 
> _this is when you implement Virtual WAN in default configuration_
- Traffic between VNets is routed and filtered by the Hub Firewall
  - Firewal is managing only core security rules, like filtering connectivity between spokes and outside, and outbound internet, of course
- Traffic between subnets is routed directly by the virtual network and filtered on the NSG
  -	Only incoming rules are used in the NSG.
  -	Outgoing traffic is not controlled in the NSG
- Traffic inside the subnet is routed directly by the virtual network and can be optionally filtered on the NSG
- FW and (optionally) NSG Flogs are streamed to Log Analytics

### Mode 2, aka Traditional
> _this is when you mimic network segmentatin described above_
- Traffic between VNets is routed and filtered by the Hub Firewall
  - Firewal is managing core security rules here
- Traffic between subnets is routed and filtered by the Hub Firewall too
  - Firewal is additinally filtering traffic between sunets, becomes an additinal point of failure, latency, and costs
- Traffic inside the subnet is routed directly by the virtual network and can be optionally filtered on the NSG

### Mode 3, aka Micro-Segmented
> _this is when you go insane to please some regulations_
- Traffic between VNets is routed and filtered by the Hub Firewall
- Traffic between subnets is routed and filtered by the Hub Firewall
- Traffic inside the subnet is routed and filtered by the Hub Firewall
  - All work and no play makes Firewall a dull boy: a point of failure, latency, and costs for all traffic in the subnet

### Comparison

Here is the comparison table:

|Criteria / Mode | Mode 1. <br/>Firewall between VNets <br/>Inter-VNet FW <br/>(Cloud Native) | Mode 2. <br/>Firewall between Subnets <br/>Inter-subnet FW <br/>(Traditional) | Mode 3. <br/>Firewall between NICs <br/>"Intra-Subnet FW"<br/>(Micro-Segmented) |
|--|--|--|--|
|Traffic Filtering|	VNET <->VNET via FW <br/>Subnet <-> Subnet via NSG <br/>Inside Subnet via NSG/ASG |VNET <->VNET via FW <br/>Subnet <-> Subnet via FW <br/>Inside Subnet via NSG	| <br/>VNET <->VNET -> FW <br/>Subnet <-> Subnet via FW <br/>Inside Subnet -> FW|
| Cost Impact | Low/Med – NSGs are free; however, Flow Logs costs should be considered. |	Medium – Costs to FW traffic and scaling | High – More costs to FW traffic and scaling |
| Management Efforts | 	High – if management of traffic filtering (both FW and NSG) is centralised to the Network team. <br/>Med - if automated end-to-end via code <br/>Low – if NSG management is delegated to Applications team, but requires level of trust |Medium/Low - centralisation of filtering management to FW reduces management efforts, provided inter-subnet filtering is managed by app teams |Med/High - need to filter and monitor all traffic increases management efforts |
| Routing |Simple, standardized UDR for all subnets	| Unique UDR for each subnet with a few Routes | Simple, standardized UDR for all subnets |
| Performance and Latency | 	Max performance, minimum latency | As platform-native for traffic inside Subnets <br/>Limited by FW for traffic between subnets | Limited by FW performance and scale <br/>May not be supported by some high demanding apps |
| Log Management Costs and Efforts | Med - Fragmented log collection on NSG and FW, however, may stream to the same Log Analytics workspace | 	Low - Centralised log collection on FW, low volumes if NSG logs are not collected	| 	Low - Centralised log collection, but high volumes |
| Visibility coverege |	100 %| Filtering between different application tiers is not applied	| 100 % |
| Reliability | High -- FW is only a point of failure for traffic between VNets | Medium -- FW becomes a point of failure between subnets | Low -- FW is PoF for any traffic |
| Use cases	| VNets for shared platform components that can protect themself with NSGs or personal firewalls <br/>VNets dedicated to single apps or groups of mutually trusted apps (like SAP landscapes) <br/>VNets with a decentralised management model (where apps are managing their own NSGs) | Shared VNets, hosting multiple apps with centralised security management (where a central team is managing communications between apps)	| Special zones requiring maximum security (should only be used where absolutely required by regulations) |


# Implementation

To implement the security modes described above, here is some high-level technical documentation.

## VNet Topology

Traditional "Regional Hub-and-Spokes", where in each region there is a dedicated Hub Vnet with a Firewall used for traffic routing and filtering, and several spoke VNets are created for workloads.

## VNet Structure

A spoke VNet should be used as a security boundary, and Security mode should be assigned for the VNet at the time of creation.

Because VNets cannot span across Azure subscriptions, the VNets structure should follow the subscription structure of the company.

This means there could be separate VNets for company business units, service lines, and environments.

## Subnets Structure

Separate Subnets should be used only when necessary, to simplify management or to implement security requirements like different route tables and NSGs. 

In Traditional and Native security modes,
- Separate subnets can be used to delegate NSG control to specific application owners

In MicroSeg security mode,
- Network security may not rely on subnets; big flat subnets can be used in conjunction with centrally managed NSGs and application-specific ASGs (details below)


## Filtering principles

When necessary, traffic filtering is applied at the destination (on FW or NSG). Traffic is not filtered at source unless absolutely necessary (like opening flows to internet destinations).

This approach allows a single point of control without spreading security between sources and destinations.

## Traffic Routing

All traffic routing to and from any Spoke Vnet is traversing the regional Hub FW:
1. Between Spokes in the same region connected via peerings
2. Between Spoke in the region and Virtual Network Gateways (Express Route, VPN, Other Regions connected to the same ER circuits)
3. Between Spoke and the Internet
4. Between Spoke and Hubs in other regions connected via global peerings

This requires each spoke VNet to have an assignment of Route tables (UDR) containing:
- BGP route propagation = disabled
- Single route to 0.0.0.0/0 via Hub NVA

For filtering traffic on FW between subnets, the route table should also contain:
- Route to current VNet address space via Hub NVA

For filtering traffic inside the subnet, the route table should also contain:
- Route to the current Subnet address space via Hub NVA

Route table content can be enforced or checked for compliance with Azure Policies.

## Security Zones

In some complex situations where you need flexibility to bypass firewall filtering between endpoints in different subnets, but keep filtering for any other subnets, you might need to implement a concept of security zones.

Security Zone consists of subnets which have the same level of trust. Security zone can be represented by a single subnet, a set of subnets in the same VNet, or even a set of subnets from different VNets (in rare cases, where it is required to peer VNets directly).

- The traffic within the same security zone can travel directly between VMs, bypassing the firewall; however, it can still be controlled on NSGs
- The traffic between different security zones should pass firewall for inspection.

For this to work, route tables should be configured for a combination of the security modes described.

## Firewalls

In MicroSeg security mode:
- Any undefined inbound and outbound traffic is denied for Spoke Vnets
- Undefined traffic between different security zones in the same VNet is denied as well
- Only explicitly defined traffic is allowed
- The traffic between the subnets of the same security zone is not passing through the firewall

In Traditional security mode:
- Any undefined inbound traffic is denied for Spoke Vnets
- Outbound traffic from Spoke Vnets is allowed by default
- Only explicitly defined incoming traffic is allowed to Spoke Vnets
- The traffic between the subnets Spoke VNet is not passing through the firewall

In Native security mode:
- Any undefined inbound and outbound traffic is allowed for Spoke Vnets
- The traffic between the subnets Spoke VNet is not passing through the firewall

Central Management: if both Regional Hub NVA

## Network Security Groups

In MicroSeg Security Mode, 
- A single NSG created for each spoke VNet.  
- The same NSG is applied to all subnets in the VNet
- NSG is managed by the central team
- 3rd party solutions can be used to centrally manage multiple NSGs
 - The last rule in NSG is to block any incoming traffic.
- Outbound traffic is not filtered on the NSG level. It can be filtered on the Hub Firewall level, if necessary.
- Any rules allowing communications between applications or for incoming traffic should be done with ASG, where possible (and load balancer IPs where not possible to use ASG)

In Traditional and Native security modes,
- NSG management is delegated to application owners, so they can control the traffic for their applications on their own. These NSGs are attached at the NIC level to the respective virtual machines.

## Application Security Groups

In MicroSeg security mode
- Application owners have to create ASGs themselves and request the central teams to create NSG rules with these ASGs
- Communications between any VMs are possible only after NSG rules are created by the central team and ASGs are assigned to VMs by application owners

In Traditional and Native security modes:
- Application owners are encouraged to use ASGs to manage their NSGs. This practice is required to be able to define in the Dev/Test environment the working rules for MicroSeg security mode.

<!--
## Discussion

As usual, I'm happy to discuss this topic with you in LinkedIn comments. If you want to support this post to get more exposure, please use [this link](https://www.linkedin.com/in/iromanovsky) to react, comment, or repost.
-->
