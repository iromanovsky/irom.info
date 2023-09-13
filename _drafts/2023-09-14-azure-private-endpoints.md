---
#layout: post-argon
title:  Public, Service, and Private endpoints in Azure
date:   2023-09-14 09:00:00 +0100
author: Igor
categories: Blog
tags: [azure, devops, caf]
#permalink: /post1/
slug: azure-endpoints
excerpt_separator: <!--more-->
redirect_from: [/post/azure-endpoints]
#published: false
---
A lot of time passed since intruduction of Private Endpoints into Azure platform, Many customers take 

Tools like Azure score, various benchmarks, and CAF documentation may look convincing for many customers to enble private endpoints for every resource possible. However, blindly following these generic recomendations may lead to more fragile solutions with higher maintenance costs.

In this post, i will cover the difference between Public, Service and Private endpoints, and share Pros and Cons for their use in common scenarios. 

# Endpoints comparison

To get started, let's compare how each type of endpoints works.

| | Public Endpoints  | Service Endpoints | Private Endpoints |
| - | - | - | - |
| **Availability** | On all PaaS services | On [limited number of PaaS services](https://learn.microsoft.com/en-us/azure/virtual-network/virtual-network-service-endpoints-overview) | On [on most of PaaS Services](https://learn.microsoft.com/en-us/azure/private-link/private-endpoint-overview#private-link-resource) |
| **How routing works** | Public Endpoints are available on public Internet IP addresses. <br /><br />**From public Internet** anyone can reach them.<br /><br />**From private VNet**, traffic is routed according to your routig tables (via Azure NAT or a Firewall).<br /><br />On some resources, like storage, there is a setting to prefer Microsoft routing, where actual traffic stays in Microsoft backbone. | Direct routes  public IP ranges forare advertised to your subnet and visible in effective routes.  These routes are specific to  [resource types and regions](https://www.microsoft.com/download/details.aspx?id=56519) <br /><br />The client traffic flows to endpoint, bypassing routing table atttached to the subnet, but stays in Microsoft backbone. It appears like NAT is not used, since the resources on service endpoints see the client original private IP from the VNet. | An IP interface is created in the VNet where you deploy your private endpoint. <br /><br />All traffic to endpoints stays in private address space. |
| **Network security measures** | &bull; On destination: Resource ACL<br />&bull; On client: NSG, firewalls | &bull; On destination: Resource ACL which can use public and private IPs, and specific VNet subnets<br />&bull; On client: [Service Endpoints policies](https://learn.microsoft.com/en-us/azure/virtual-network/virtual-network-service-endpoint-policies-overview) (only available for selected services), NSG | &bull; On destination: NSG (when [Private Endpoint Network Policies](https://learn.microsoft.com/en-us/azure/private-link/disable-private-endpoint-network-policy?tabs=network-policy-portal) are enabled)<br />&bull; On client: NSG, firewalls |
| **Management burden** | No | &bull; Need to disable Public Access on Resource ACL<br />&bull; Need to enable Service Endpoints on each subnet where they are used | &bull; Need to disable Public Access on Resource ACL<br />&bull; Need to deploy PE to VNets, for each service (blob, file) on each resource (storage, key vault)<br />&bull; Consuming private IP addresses, especially if you deploy PEs in dedicated subnets (+5 reserved IP addresses wasted)<br />&bull; Requires changes in name resolution (from changing HOSTS file on each client up to implementing sophisitcated DNS integration<br /> &bull; High complexity for name resolution when need to deploy multiple PEs of the same resource |

# General recommendations

Below are scenarios where using each type of endpoints is recommended.

## Public Endpoints

Use Public endpoints for:

- Public resources, like web hosting (app service, storage)
- Resources for cloud-born applications, like SaaS, PaaS, and apps that are not integrated with your private network (web apps, functions, storage)
- Sharead internal resources, which should be available across networks, and clouds (storage, monitoring, backup, site recovery)
- Resources used from other clouds

## Service Endpoints

Use service endpoints for:

- Services requiring highest possible bandwidth and least possible latency (storage, databases)
- Shared services consumed from multiple VNets and regions (central file share, Cloud Witness for clusters)
- Services requiring less administratove effort than managing private endpoints

## Private Endpoints

Use private endpoints for:

- Resources that need to be accessed from on-premises over private network. Requires establishing hybred connectivity and DNS integration.
- Resources only required to be accessed from one VNet and not exposed outside of their system (private data pattern, like internal databases and storage).
- Resources for environments requiring highest protection from data exfiltration. These class of systems should be traeated separately from general class. To effectively protect from data exfiltration, all inbound and outbound traffic should be restricted to tenant-specific destinations. This may come with a cost of losing a number of native platform and vendor capabilities, like Monitoring, Backup, Site Recovery, Patching automation, and others. Most of these services are multi-tenant and do not work without opening wide IP ranges and wildcard URLs to  destinations shared between multiple tenants.
- Complying to specific regulations

The benefits of Private Endpoints are **doubtful** in the below scenarios:

- When blindly following generic recomendations, "best practices", improving benchmarks and "secure scores". These tools usually do not consider your usage scneario, and blndly implementing all recomendations may lead to higher complexity and fragility of your systyem, while increasing maintenance costs. In particular case of private endpoints, enforcing them without a valid reason may lead to losing cloud capabilities while not gaining benefits.
- Relying on network to provide secure access. In zero trust security model, a breach is assumed on every point of the data flow. Network should not be trusted to provide secury on the path of data. Means such as selective traffic routing, private networks, and firewalls do not provide enough protection in hybrid environments, spanning across multiple datacenters, clouds and VPN devices. Instead of relying on network, end-to-end encryption, authentications and endpoint protection should be used.
- Providing access to corporate resources over VPN. Due to nature of VPN, the clients are distributed and VPN becomes a bottleneck when accessing a private endpoint. When accessing the resource over a public endpoin, Microosft backbone is utilised to provide more efficient data path.
- When you need to provide access to a resource across multiple Azure regions in hub-and-spoke model (including the situation when you need to fail-over a resource to another region). While maintaining private endpoints of the same resource in each region and/or even each VNet could be a solution, it introduces high complexity with name resolution.
- When practicing cloud native approach for designing new applications, and using as much of PaaS services as possible, private endpoints could be unnecessary "grounding point" between all these PaaS components, leading to hogher complexity while increasing costs (usually vnet integration options of PaaS services come with a premium price points).

# Endpoint type recommendations per Azure service

Since every company may have unique environment and requeremes, it makes sense to document the guidance on use cases of endpoints for specific scenarios.

| Service | Recommendation |
|--|--|
| Storage Account | <p>**Public Endpoint**  for publicly accessible storage like diagnostic, Terraform state file, exchanging information with external parties etc<p/><p>**Service Endpoint** for storage shared across VNets and/or regions</p><p>**Private Endpoint**: for private data for apps in high security zone; for file shares when migrating data from on-premise |
| Azure KeyVault | <p>_Note: Key Vault security is not improved much by using PE, since KV access is protected with Azure AD authentication. KV is not practical for data exfiltration, and KV client still requires outbound access to AAD_</p><p>**Public + Service Endpoint** for shared Key Vaults used by Automation / DevOps / Terraform / PaaS / Admins </p> <p>**Private Endpoint** for app-specific KVs deployed on-demand for storing private data like enryption keys</p> |
| Azure SQL Database | <p>**Public Endpoints Endpoints** for cloud native PaaS apps not integrated into VNet</p> <p>**Private Endpoints** for private database when used from IaaS servers</p>
| ..and so on..|  |

# Discussion

You are very welcome to discuss this topic in Comments.