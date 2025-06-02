---
#layout: post-argon
title: Azure Network Security Modes
date: 2025-06-01 18:00:00 +0000
#last_modified_at: 2025-06-01 18:00:00 +0000
author: Igor
categories: [Azure, Networking]
tags: [azure, network, security, architecture]
permalink: /blog/azure-network-security-modes
slug: azure-network-security-modes
excerpt_separator: <!--more-->
redirect_from: [/blog/2025/06/01/azure-network-security-modes]
image: https://github.com/user-attachments/assets/6ad6c2fb-d4fa-4544-bde6-02d0eb3e508d
description: >-
  Describing network security modes

#published: false
---

<div style="text-align: center">

![Network Diagram - Modes](https://github.com/user-attachments/assets/6ad6c2fb-d4fa-4544-bde6-02d0eb3e508d)

</div>

This was initially intended to debunk the traditional concept of network segmentation as obsolete in the context of Azure, and briefly offer a better replacement: the concept of Network Security Modes.

While expanding the context, I ended up giving away my principles for network architecture documentation, which I have been refining for the last 10+ years. I keep it abstracted to only one step away from a complete LLD that you can reuse to build a modern Hub-and-Spoke network topology in Azure.

Organic, AI-free writing.

<!--more-->

## Network segmentation rationale

### Ancient times

In the old days, when servers lived in on-premise datacenters, there was a concept of network segmentation.

**Hubs and Switches**

Servers were connected by hubs and switches.

Every server connected to the same network hub was receiving the same [datagrams](## "messages on the network"), including ones not addressed to them. These hubs were also called repeaters.

Network switches are helping to reduce the number of situations where datagrams are repeated on multiple ports when the switch "remembers" who is on the port. It is still not perfect at remembering everyone, especially in big networks.

**Subnets**

And networks were big. You remember subnet classes A/B/C? C means 256 addresses. It was normal to have hundreds and thousands of servers connected via hubs and switches, and they formed subnets or broadcast domains.

To reiterate: any traffic in the subnet is broadcast and can be overheard by any interface on that subnet.

**Routers** are used to interconnect and remove broadcast traffic between subnets, allowing only traffic destined for these subnets.

Limiting broadcast traffic is good, but routers are expensive. That's why subnets were big and had many servers.

Traditional routers were not as smart as today's, so they could not filter ports, protocols, and other attributes that security guys like.

**Firewalls** are used to filter traffic and are even rarer and more expensive than routers.

It was so expensive that it was only placed on critical network boundaries, like dividing internal networks from DMZ and internets.

The cost and rarity of firewalls forced companies to group their servers into network segments, where everyone trusts each other.

**Network segmentation**

At that time, network traffic was rarely encrypted -- everyone in the subnet could potentially hear all network conversations.

Operating systems like Windows 2000 did not have a personal firewall, so all ports were open by default -- making them vulnerable to any bad guy or a virus that is on the network

Authentication protocols and passwords were weak (and no MFA!) -- to the pleasure of hackers.

These aspects led to the necessity of making sure that all neighbours in the networks are trusted.

Here comes the concept of a network segment -- a part of a network that groups servers with a similar level of trust together, protected by a single firewall.

Typical network segments were Prod / Dev / Workplace / DMZ / Internet, and all devices within the same segment were treated equally.

### Modern times

Nowadays, when you hear the term network hub, it does not mean repeater, but a central place connecting multiple virtual networks where routing and traffic filtering happen.

Virtual Networks in Azure run in a shared, multitenant environment of the cloud provider, and

- network virtualisation protocol ensures that traffic is not visible to other customers -- no bad neighbours in the subnet can overhear your conversations
- traffic filtering platform is processing every datagram -- firewall capabilities are available on every connected interface via NSG

More than that,

- modern operating systems have firewalls enabled by default (and it's a shame if your company disable them as a standard)
- modern network protocols keep traffic properly authenticated (MFA/SSO/SAML/OAuth -- all these beauties) and encrypted (TLS/AES/oh-yes) from end-to-end

This makes the concept of network segmentation mostly irrelevant for Azure virtual networks since the issues that segmentation was initially addressing no longer exist.

_Henceforth_, I declare the traditional network segmentation obsolete.

## Azure Network Security Modes

New times are coming, security guys stay the same.

They still want to control and see the traffic on a firewall and don't want to rely on zero-trust principles.

To please them, I want to introduce you to a modern way of organising network filtering, which I call Azure Network Security Modes.

### Mode 1, aka Cloud Native

> _this is when you implement Virtual WAN in default configuration_

- Traffic between VNets is routed and filtered by the Hub Firewall
  - Firewal is managing only core security rules, like filtering connectivity between spokes and outside, and outbound internet, of course
- Traffic between subnets is routed directly by the virtual network and filtered on the NSG
  - Only incoming rules are used in the NSG.
  - Outgoing traffic is not controlled in the NSG
- Traffic inside the subnet is routed directly by the virtual network and can be optionally filtered on the NSG
- FW and (optionally) NSG Flogs are streamed to Log Analytics

### Mode 2, aka Traditional

> _this is when you mimic a traditional datacenter_

- Traffic between VNets is routed and filtered by the Hub Firewall
  - Firewall is managing core security rules here
- Traffic between subnets is routed and filtered by the Hub Firewall, too
  - Firewall is additionally filtering traffic between subnets, becoming an additional point of failure, latency, and costs
- Traffic inside the subnet is routed directly by the virtual network and can be optionally filtered on the NSG

### Mode 3, aka Micro-Segmented

> _this is where you go to please some insane regulations_

- Traffic between VNets is routed and filtered by the Hub Firewall
- Traffic between subnets is routed and filtered by the Hub Firewall
- Traffic inside the subnet is routed and filtered by the Hub Firewall
  - All work and no play makes Firewall a dull boy: a point of failure, latency, and costs for all traffic in the subnet

### Comparison

Here is the comparison table:

<table>
  <thead>
    <tr>
      <th>Criteria / Mode</th>
      <th>Mode 1.
          <br/>Firewall between VNets
          <br/>Inter-VNet FW
          <br/>(Cloud Native)</th>
      <th>Mode 2.
          <br/>Firewall between Subnets
          <br/>Inter-subnet FW
          <br/>(Traditional)</th>
      <th>Mode 3.
          <br/>Firewall between NICs
          <br/>"Intra-Subnet FW"
          <br/>(Micro-Segmented)</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><strong>Traffic Filtering</strong></td>
      <td></td>
      <td></td>
      <td></td>
    </tr>
    <tr>
      <td>VNet &lt;-&gt; VNet</td>
      <td>via FW</td>
      <td>via FW</td>
      <td>via FW</td>
    </tr>
    <tr>
      <td>Subnet &lt;-&gt; Subnet</td>
      <td>via NSG</td>
      <td>via FW</td>
      <td>via FW</td>
    </tr>  
    <tr>
      <td>Inside Subnet</td>
      <td>via NSG</td>
      <td>via NSG</td>
      <td>via FW</td>
    </tr>
    <tr>
      <td><strong>Cost Impact</strong></td>
      <td>Low/Med &ndash; NSGs are free; however, Flow Logs costs should be considered.</td>
      <td>Medium &ndash; Costs to FW traffic and scaling</td>
      <td>High &ndash; More costs to FW traffic and scaling</td>
    </tr>
    <tr>
      <td><strong>Management Efforts</strong></td>
      <td>&bull; High &ndash; if management of traffic filtering (both FW and NSG) is centralised to the Network team.<br/>&bull; Med &ndash; if automated end-to-end via code<br/>&bull; Low &ndash; if NSG management is delegated to Applications team, but requires a level of trust</td>
      <td>&bull; Medium/Low &ndash; centralisation of filtering management to FW reduces management efforts, provided inter-subnet filtering is managed by app teams</td>
      <td>&bull; Med/High &ndash; need to filter and monitor all traffic increases management efforts</td>
    </tr>
    <tr>
      <td><strong>Routing</strong></td>
      <td>Simple, standardised UDR for all subnets</td>
      <td>Unique UDR for each subnet with a few Routes</td>
      <td>Simple, standardised UDR for all subnets</td>
    </tr>
    <tr>
      <td><strong>Throughput and Latency</strong></td>
      <td>&bull; Max throughput, minimum latency</td>
      <td>&bull; As platform-native for traffic inside Subnets <br/>&bull; Limited by FW for traffic between subnets</td>
      <td>&bull; Limited by FW performance and scale<br/>&bull; May not be supported by some high demanding apps</td>
    </tr>
    <tr>
      <td><strong>Log Management Costs and Efforts</strong>strong></td>
      <td>Med &ndash; Fragmented log collection on NSG and FW, however, may stream to the same Log Analytics workspace</td>
      <td>Low &ndash; Centralised log collection on FW, low volumes if NSG logs are not collected</td>
      <td>Low &ndash; Centralised log collection on FW, but high volumes</td>
    </tr>
      <td><strong>Reliability</strong></td>
      <td>High &ndash; FW is only a point of failure for traffic between VNets</td>
      <td>Medium &ndash; FW becomes a point of failure between subnets</td>
      <td>Low &ndash; FW is PoF for any traffic</td>
    </tr>
    <tr>
      <td><strong>Use cases</strong></td>
      <td>&bull; VNets for shared platform components that can protect themselves with NSGs or personal firewalls<br/>&bull; VNets dedicated to single apps or groups of mutually trusted apps (like SAP landscapes)<br/>&bull; VNets with a decentralised management model (where apps are managing their own NSGs)</td>
      <td>&bull; Shared VNets, hosting multiple apps with centralised security management (where a central team is managing communications between apps)</td>
      <td>&bull; Special zones requiring maximum security (should only be used where absolutely required by regulations)</td>
    </tr>
  </tbody>
</table>

## Implementation

Here is some example technical documentation for implementing the security modes described above.

### VNet Topology

Traditional "Regional Hub-and-Spokes", wherein each region has a dedicated Hub VNet with a Firewall used for traffic routing and filtering, and several spoke VNets hosting workloads are attached via peerings.

| Region        | Address Spaces   | Address Range                | Remarks |
|-------------- |----------------|------------------------------| --|
| West Europe   | 10.0.0.0/10    | 10.0.0.0 – 10.63.255.255     | It is always recommended to pre-allocate simpler, reasonable chunks of IP space to specific regions... |
| North Europe  | 10.64.0.0/10   | 10.64.0.0 – 10.127.255.255   | ... This makes routing tables simpler and helps to avoid unnecessary complexity and hitting limits for the number of routes... |
| New region  | 10.128.0.0/10   | 10.128.0.0 – 10.191.255.255   | ...  As you need to provision new spokes, you cut IP addresses from these pre-allocated ranges... |
| Free  | 10.192.0.0/10   |  10.192.0.0 – 10.255.255.255   | ... But always keep free space for potential growth... |

I might write a separate article on efficient IP range allocation. However, special tools for [IPAM](## "IP address management") exist, like Efficient IP and Azure IPAM.

### VNet Structure

A spoke VNet should be used as a security boundary, and Security mode should be assigned to the VNet at its creation.

Because VNets cannot span across Azure subscriptions, the VNets structure should follow the subscription structure of the company.

This means separate VNets could exist for company business units, service lines, and environments.

| VNet Name   | Address Space | Region      | Subscription | Comments                                                                      |
| ----------- | ------------- | ----------- | ------------ | ----------------------------------------------------------------------------- |
| Hub-vnet    | 10.0.0.0/24   | West Europe | Hub-sub      | Regional Hub                                                                  |
| SAP-vnet    | 10.1.0.0/16   | West Europe | SAP-sub      | Mode 1: VNet is dedicated to SAP application landscapes that trust each other |
| Shared-vnet | 10.2.0.0/16   | West Europe | Shared-sub   | Mode 2: VNet is hosting multiple apps managed by different teams              |
| Secret-vnet | 10.3.0.0/16   | West Europe | Secret-sub   | Mode 3: VNet is hosting apps that need to comply with insane regulations      |

Once assigned, security mode sets the default level for the workload provisioned in the spoke. Since we are in the cloud with software-defined networks, the flexibility of changing this is possible for some subnets or even for the whole VNet, but it is not possible to change the VNet name. That's why I don't encourage "tattooing" the initial network security mode in the VNet name.

### Subnets Structure

Separate subnets should be used only when necessary to simplify management or implement security requirements like different route tables and NSGs.

In Native and Traditional security modes,

- Separate subnets can be used to delegate NSG control to specific application owners

In MicroSeg security mode,

- Network security may not rely on subnets; big flat subnets can be used in conjunction with centrally managed NSGs and application-specific ASGs (details below)

| VNet Name   | Subnet Name | Address Space | Comments                                                                     |
| ----------- | ----------- | ------------- | ---------------------------------------------------------------------------- |
| SAP-vnet    |             | 10.1.0.0/16   | Apps in this VNet are managed by the same team and use NSG/ASG               |
|             | SPM         | 10.1.0.0/24   |                                                                              |
|             | GRC         | 10.1.2.0/25   |                                                                              |
|             | CRM         | 10.1.3.0/25   |                                                                              |
| Shared-vnet |             | 10.2.0.0/16   | Apps in this VNet are managed by different teams who do not trust each other |
|             | App1        | 10.2.0.0/24   | App 1                                                                        |
|             | App2        | 10.2.1.0/24   | App 2                                                                        |
|             | App3_FE     | 10.2.2.0/24   | App 3 Front-end                                                              |
|             | App3_BE     | 10.2.3.0/24   | App 3 Back-end                                                               |
|             | App4_FE     | 10.2.4.0/24   | App 4 Front-end                                                              |
|             | App4_BE     | 10.2.5.0/24   | App 4 Back-end                                                               |
| Secret-vnet |             | 10.3.0.0/16   | Apps in this VNet have to comply with strict industry regulations            |
|             | PCIDSS      | 10.3.0.0/24   |                                                                              |
|             | HIPAA       | 10.3.1.0/24   |                                                                              |

### Traffic Routing

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

For enable direct traffic inside the subnet, the route table should also contain:

- Route to the current Subnet address space via Virtual Network

Here is the configuration example:

| UDR Name                    | Assigned to            | Routes                          | Comments                                                                             |
| --------------------------- | ---------------------- | ------------------------------- | ------------------------------------------------------------------------------------ |
| SAP-vnet-udr                | SAP-vnet / *Subnets    | 0.0.0.0/0 → NVA IP (Firewall)   | To override system route pointing to Internet                                        |
| Shared-vnet-Subnet_App1-udr | App1                   | 0.0.0.0/0 → NVA IP (Firewall)   | To override system route pointing to Internet                                        |
|                             | App1                   | 10.2.0.0/16 → NVA IP (Firewall) | To override system route to VNet Address Space pointing to Virtual Network           |
|                             | App1                   | 10.2.0.0/24 → Virtual Network   | To override route above to enable direct communication for interfaces in App1 subnet |
| Shared-vnet-Subnet_App2-udr | App2                   | 0.0.0.0/0 → NVA IP (Firewall)   | To override system route pointing to Internet                                        |
|                             | App2                   | 10.2.0.0/16 → NVA IP (Firewall) | To override system route to VNet Address Space pointing to Virtual Network           |
|                             | App2                   | 10.2.1.0/24 → Virtual Network   | To override route above to enable direct communication for interfaces in App2 subnet |
| Secret-vnet-udr             | Secret-vnet / *Subnets | 0.0.0.0/0 → NVA IP (Firewall)   | To override system route pointing to Internet                                        |
|                             | Secret-vnet / *Subnets | 10.3.0.0/16 → NVA IP (Firewall) | To override system route to VNet Address Space pointing to Virtual Network           |

As you see, to implement Mode 2, you need a separate UDR resource for each subnet in the VNet, which complicates management unless it is fully automated.

To implement modes 1 and 3, a single route table object per VNet is enough.

Route table content can be enforced or checked for compliance with Azure Policies.

### Security Zones

In some complex situations where you need flexibility to bypass firewall filtering between endpoints in different subnets, but keep filtering for any other subnets, you might need to implement a concept of security zones.

A Security Zone consists of subnets with the same level of trust. A security zone can be represented by a single subnet, a set of subnets in the same VNet, or even a set of subnets from different VNets (in rare cases, where it is required to peer VNets directly).

- The traffic within the same security zone can travel directly between VMs, bypassing the firewall; however, it can still be controlled on NSGs
- The traffic between different security zones should pass through the firewall for inspection.

For this to work, route tables should be configured for a combination of the security modes described.

We might call it Mode 1.5, in which some subnets are routed directly, but other subnets are still routed through a firewall.

I don't encourage using this mode, but I am describing it here to demonstrate the beauty of SDWAN flexibility.

In a nutshell, a VNet with a single security zone (all subnets equal) becomes Mode 1, and a VNet with security zones equal to the number of subnets becomes Mode 2.

I recommend using either mode 1 or 2 and keeping the option for Security zones for special occasions that might never happen.

| UDR Name                  | Assigned to         | Routes                                                         | Comments                                                                            |
| ------------------------- | ------------------- | -------------------------------------------------------------- | ----------------------------------------------------------------------------------- |
| Shared-vnet_Zone_App3-udr | App3_FE <br>App3_BE | 0.0.0.0/0 → NVA IP (Firewall)                                  | To override system route pointing to Internet                                       |
|                           | App3_FE <br>App3_BE | 10.2.0.0/16 → NVA IP (Firewall)                                | To override system route to VNet Address Space pointing to Virtual Network          |
|                           | App3_FE <br>App3_BE | 10.2.2.0/16 → Virtual Network<br>10.2.3.0/24 → Virtual Network | To override route above to enable direct communication between  App3_FE and App3_BE |
| Shared-vnet-Zone_App4-udr | App4_FE <br>App4_BE | 0.0.0.0/0 → NVA IP (Firewall)                                  | To override system route pointing to Internet                                       |
|                           | App4_FE <br>App4_BE | 10.2.0.0/16 → NVA IP (Firewall)                                | To override system route to VNet Address Space pointing to Virtual Network          |
|                           | App4_FE <br>App4_BE | 10.2.4.0/16 → Virtual Network<br>10.2.5.0/24 → Virtual Network | To override route above to enable direct communication between  App1_FE and App1_BE |

### Filtering principles

Traffic filtering is applied at the destination (on FW or NSG) when necessary. Traffic is not filtered at the source unless absolutely necessary (like opening flows to internet destinations).

This approach allows a single control point without spreading security between sources and destinations.

### Firewalls

For spokes in Native security mode:

- Any undefined inbound and outbound traffic is allowed for this spoke VNet (but still managed on NSG)
- The traffic between the subnets of this spoke VNet is not passing through the firewall

For spokes in Traditional security mode:

- Any undefined inbound traffic is denied for this spoke VNet
- Outbound traffic from this spoke VNet is allowed by default
- Explicitly defined Inbound traffic is allowed
- The traffic between the subnets of Spoke VNet is passing through the firewall and should be explicitly opened
- Any traffic within the same subnet of this spoke VNet is not passing through the firewall (but still can be managed on NSG)

For spokes in MicroSeg security mode:

- Any undefined inbound and outbound traffic is denied for this spoke VNet
- Any traffic in the subnets of this spoke VNet is passing through the firewall and should be explicitly opened (no point to manage on NSG, unless required)

Here is an example of Firewall policy configuration to enable this model:

| Rule   Collection Group     | From                     | To                       | Settings                                                           |
| --------------------------- | ------------------------ | ------------------------ | ------------------------------------------------------------------ |
| Core   outbound - Local     | {All   connected spokes} | {Local   ranges}         | Permitted   core services, like AD, DSN, NTP, etc                  |
| Core   outbound - Internet  | {All   connected spokes} | {Internet}               | Safe Internet destinations, like Azure Portal, Windows Update, etc |
| Mode 1   - inbound          | *                        | {All   spokes in Mode 1} | Any   traffic -- not filtered on firewall, managed on NSG          |
| Mode 1   - outbound         | {All   spokes in Mode 1} | *                        | Any   traffic -- not filtered on firewall, managed on NSG          |
| Mode   2  - App1 - outbound | App1   Subnet            | {Internet}               | Rules   for outbound traffic from App1                             |
| Mode   2  - App1 - inbound  | *                        | App1   Subnet            | Rules   for inbound traffic from App1                              |
| Mode   2  - App1_to_App2    | App1   Subnet            | App2   Subnet            | Rules   between App1 and App2                                      |
| Mode   3  - NIC1 - inbound  | NIC1 IP                  | {Internet}               | Rules   for outbound traffic from NIC1 in Secret Vnet              |
| Mode   3  - NIC1 - outbound | *                        | NIC1 IP                  | Rules   publishing specific services on NIC1 in VNetVNet           |

The rules above are for example only. You need to carefully consider the rules' order to ensure that rules opening from Mode 1 do not affect traffic in Modes 2 and 3, since the same firewall is used for all networks.

This also shows the benefits of using NSG when you cannot fully trust the firewall configuration.

### Network Security Groups

In Traditional and Native security modes,

- NSG management can be delegated to application owners so they can control their applications' traffic independently
- NSGs are attached at the Subnet level. Attaching NSGs to individual NICs is not encouraged due to overkill management complexity and overlapping functionality with personal firewalls.
- If required, NSGs can be managed centrally via code or 3rd party solutions like AlgoSec
- Use of ASG is encouraged to simplify and organise NSGs, where possible (several services like load balancer IPs are not able to use ASG)

In MicroSeg Security Mode,

- Use of NSGs does not make a lot of sense, since the Firewall already filters all traffic, unless you need additional protection and logging on top of the Firewall

Here is an example of NSGs' configuration to enable this model:

| NSG                         | Source                  | Destination  | Settings            | Remarks                                                              |
| --------------------------- | ----------------------- | ------------ | ------------------- | -------------------------------------------------------------------- |
| SAP-vnet-nsg                | {Local   ranges}        | *            | Allow   In TCP_3389 | Opening   flow to all destination in the VNet                        |
|                             | *                       | 10.1.2.0/24  | Allow   In TCP_443  | Opening   flow to GRC subnet                                         |
|                             | {Some   Trusted Source} | {SAP_DB-asg} | Allow   In TCP_1433 | Opening   flow to specific ASG                                       |
|                             | *                       | *            | Deny In   *         | Deny   all not explicitly open inbound traffic                       |
| Shared-vnet_Subnet_App1-nsg | {Local   ranges}        | 10.2.0.0/24  | Allow   In TCP_3389 | Opening   flow to App1 subnet                                        |
| Shared-vnet_Subnet_App2-nsg | {Local   ranges}        | 10.2.1.0/24  | Allow   In TCP_22   | Opening   flow to App2 subnet                                        |
| Secret-vnet-nsg             |                         |              | {empty}             | No   point managing NSG for Mode 3 as everything is controlled on FW |

### Application Security Groups

In Traditional and Native security modes:

- Use of ASG is encouraged to simplify and organise NSGs, where possible (several services like load balancer IPs are not able to use ASG)
- ASG may also be used to delegate some freedom to application teams, while NSG management is centralised and not in their control

In MicroSeg security mode:

- ASG may enable some flexibility when NSGs are still used in addition to the firewall

Here is an example of ASGs' configuration of this model:

| Network   Interface Config                                                      | Member   of ASG |
| ------------------------------------------------------------------------------- | --------------- |
| SAP-GRC-DB01-nic     SAP-GRC-DB02-nic     SAP-CRM-DB01-nic     SAP-SRM-DB01-nic | SAP_DB-asg      |
| SAP-GRC-FE01-nic     SAP-GRC-FE02-nic     SAP-CRM-FE01-nic     SAP-SRM-FE01-nic | SAP_FE-asg      |

## Diagram

The diagram from the page cover was used for attention and does not completely reflect the architecture described here. Once I update it, I will post the diagram in bigger resolution here.

## Discussion

This page ended up much bigger than I originally expected, and might need polishing.

As usual, I'm happy to discuss this topic with you in LinkedIn comments. If you want to support this post to get more exposure, please use [this link](https://www.linkedin.com/in/iromanovsky) to react, comment, or repost.
